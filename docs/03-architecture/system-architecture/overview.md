# VyteMerge — Arquitectura (simple y concisa)

## Objetivo
Mostrar **cómo fluye el sistema** para "vender" un **Listing** (publicación) a través de **configurar y generar Slots** (disponibilidad) y convertirlos en **Bookings** (reservas).

## Stack (actual)
- **Frontend**: Angular + **Micro Frontends (Module Federation)** (Shell + MFEs)
- **Identity**: **Keycloak** (service) para AuthN/AuthZ
- **Backend**: .NET Core **Modular Monolith**
- **DB**: **PostgreSQL**
- **Infra dev**: Docker / Docker Compose

---

## 1) Diagrama de sistema (alto nivel)

```mermaid
flowchart LR
  U[Usuario / Cliente] --> FE[Angular Shell + MFEs]
  P[Proveedor / Negocio] --> FE

  FE -->|OIDC/OAuth2| KC[Keycloak]

  FE -->|REST/JSON| API[.NET Modular Monolith]
  API --> PG[(PostgreSQL)]

  subgraph Docker[Docker Compose]
    KC
    API
    PG
  end
```

---

## 2) Módulos (flujo Listing → Slots → Booking)

| Módulo (backend) | Responsabilidad | Entidades clave |
|---|---|---|
| Identity (Keycloak) | Login, tokens, roles/claims | Realm, Client, User, Role |
| Profiles | Identidad de negocio (proveedor/cliente) | Profile |
| Catalog | Definir qué se ofrece | Service |
| Listing | Definir cómo se vende (scheduling, pricing, media) | Listing, SlotConfig, Recurrence |
| Timelines | Configuración de agendas y reglas de conflicto | Timeline, ConflictRule |
| Events | Lo que ocupa tiempo + slot projection | Event, EventTimelineLink, ConflictDetection |
| Booking | Transacción de reserva | Booking (hold → confirm → complete) |
| Communication (opcional) | Notificaciones (confirmaciones) | Notification |

> **ADR-0006:** Event es entidad independiente linked a N Timelines (no owned por Timeline). Slot Projection vive en Events.

---

## 3) Modelo mínimo (relaciones clave)

```mermaid
erDiagram
  PROFILE ||--o{ LISTING : offers
  LISTING ||--|| SERVICE : describes
  LISTING ||--o{ SLOT_CONFIG : defines
  PROFILE ||--o{ TIMELINE : owns
  TIMELINE ||--o{ CONFLICT_RULE : configures
  EVENT }o--o{ TIMELINE : "linked via EventTimelineLink"
  SLOT_CONFIG ||--o{ SLOT : generates
  SLOT ||--o{ BOOKING : converts
  BOOKING ||--o| EVENT : "creates on confirm"
```

**Lectura**
- Un **Listing** representa "lo que vendés" (Service + reglas).
- Un **SlotConfig** define "cómo genero slots" (duración, días, buffers, reglas).
- El **Timeline** es la configuración de la agenda (privacy, conflict rules, members).
- Los **Events** son entidades independientes linked a N Timelines — representan "lo que ocupa tiempo".
- Los **Slots** se proyectan (query-time) desde Listing rules - Events ocupados - ConflictRules.
- Los **Bookings** son transacciones que al confirmarse crean un Event linked a los timelines relevantes.

---

## 4) Flujo principal (end-to-end) — Proveedor publica, cliente reserva

### 4.1 Proveedor: crear Service + Listing + SlotConfig

```mermaid
sequenceDiagram
  participant P as Proveedor
  participant API as .NET API
  participant DB as PostgreSQL

  P->>API: CreateService
  API->>DB: persist Service

  P->>API: CreateListing (refs Service, defines SlotConfig)
  API->>DB: persist Listing + SlotConfig

  P->>API: PublishListing
  API->>DB: Listing.status = Published
```

### 4.2 Cliente: ver disponibilidad y crear Booking

```mermaid
sequenceDiagram
  participant C as Cliente
  participant API as .NET API
  participant DB as PostgreSQL

  C->>API: GET /availability (listingId + dateRange)
  API->>DB: read Events (ocupados) + Listing rules + ConflictRules
  API-->>API: project Slots (Listing rules - Events - Conflicts)
  API-->>C: available Slots

  C->>API: POST /bookings/hold (slotId, listingId)
  API->>DB: persist Booking (status = Holding, TTL)

  C->>API: POST /bookings (confirm)
  API->>DB: persist Booking (status = Confirmed)
  API-->>API: create Event + link to provider & client timelines
  API->>DB: persist Event + EventTimelineLinks
  API-->>C: BookingCreated
```

---

## 5) Estados mínimos (para no complicar)
```mermaid
stateDiagram-v2
  [*] --> Holding
  Holding --> Pending
  Holding --> Confirmed
  Pending --> Confirmed
  Pending --> Cancelled
  Confirmed --> Completed
  Confirmed --> Cancelled
  Confirmed --> NoShow
  Cancelled --> [*]
  Completed --> [*]
  NoShow --> [*]
```

---

## 6) Deployment (dev) — Docker Compose (conceptual)
```mermaid
flowchart TB
  subgraph DockerCompose[Docker Compose]
    KC[Keycloak]
    API[.NET Modular Monolith API]
    PG[(PostgreSQL)]
  end

  FE[Angular Shell + MFEs] -->|OIDC| KC
  FE -->|REST| API
  API --> PG
```

---

## Checklist (para alinear código ↔ doc)
- [x] Confirmar dónde vive **SlotConfig** (Offer vs Supply) — decidido en `ADR-0001`: SlotConfig vive en **Offer** (Listing lo define), Events proyecta Slots.
- [x] Event como entidad independiente con links a Timeline — decidido en `ADR-0006`.
- [ ] Definir endpoint "**GET /availability**" con inputs claros (listingId + dateRange) — ver `03-architecture/api-contracts/timelines.md`.
- [x] Definir transición de estados de Booking — ver `01-domain-behavior/02-lifecycles/booking-lifecycle.md`.
- [ ] Limpiar código de template (Event/Category/TicketType) de Catalog y Booking modules.
- [ ] Implementar EventTimelineLink (N:M) en Events module.
