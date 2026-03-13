---
title: Flow 11 — Cancelar, Reagendar y Policies
description: >
  El proveedor configura las políticas de cancelación, reagendado y no-show en su Listing.
  El cliente y el proveedor pueden cancelar o reagendar Bookings; el sistema evalúa las
  políticas vigentes al momento de la confirmación (policySnapshot) para determinar
  reembolsos, penalidades y condiciones de reagendado.
status: draft
version: 1
---

# Flow 11 — Cancelar, Reagendar y Policies

## 1. Resumen
- **Goal:** habilitar la gestión completa del ciclo de vida post-confirmación de un Booking:
  configurar políticas por Listing, cancelar con reembolso calculado por Policy, y reagendar
  con aprobación del otro actor.
- **Actores:**
  - **Primary:** Proveedor (Owner o Delegado con `ManageBookings`) — configura políticas y gestiona reagendados.
  - **Secondary:** Cliente (attendee) — cancela o solicita reagendar sus propios Bookings.
  - **Sistema VyteMerge** — evalúa Policies, procesa reembolsos/penalidades vía Commerce, notifica.
- **Surfaces:**
  - Proveedor: `vt-listings-mfe` (`/listings/:id/edit#policies`) y `vt-timeline-mfe` (`/timeline/bookings/:id`).
  - Cliente: `vt-timeline-mfe` (`/bookings` y `/bookings/:id`, `role=attendee`).

---

## 2. Domain Context

### Relación con Flow 10
Flow 10 documentó acciones del proveedor con una versión simplificada de cancelación (100% reembolso fijo)
y NoShow (penalidad fija desde config del Listing). Flow 11 implementa el módulo `Policies` (BC 07)
que reemplaza esa lógica simplificada.

### Módulo Policies (BC 07)
Policies **no gestiona estado** ni ejecuta pagos. Responde a la pregunta:
*"Dado este Booking y esta acción en este momento, ¿qué reglas aplican y cuáles son las consecuencias?"*

| Policy | Descripción |
|--------|-------------|
| `CancellationPolicy` | Ventanas de tiempo con porcentaje de reembolso y penalidades por actor |
| `ReschedulePolicy` | Quién puede reagendar, ventana mínima, `samePriceRequired`, requiere aprobación |
| `NoShowPolicy` | Penalidad al cliente; acción de plataforma si el proveedor no se presenta |

### PolicySnapshot en el Booking
Al confirmarse un Booking, se guarda un snapshot de las Policies vigentes del Listing.
Las acciones posteriores (cancelar, reagendar) se evalúan contra el **snapshot**, no contra
las Policies actuales del Listing. Si el proveedor cambia sus políticas, solo afecta a Bookings
confirmados después del cambio.

### Default de plataforma
Si un Listing no tiene CancellationPolicy configurada, el sistema aplica el default:

| Ventana | Reembolso |
|---------|-----------|
| > 24h antes del slot | 100% |
| ≤ 24h antes del slot | 0% |

### Reschedule: slot original permanece bloqueado
Mientras un reagendado está en estado `Pending` (esperando confirmación del otro actor),
el slot original permanece bloqueado. El nuevo slot queda en soft-hold. Al confirmarse
el reagendado, el slot original se libera y el nuevo slot queda bloqueado.

---

## 3. Preconditions
- Existe un Booking en estado `Confirmed` (para cancelar o reagendar).
- Para cancelar un `Pending`: solo el cliente o el proveedor pueden hacerlo (ya cubierto en Flow 10 para el proveedor).
- Para configurar Policies: el actor es Owner o Delegado con `ManageBookings`; el Listing existe.

---

## 4. Trigger

**Capacidad A (configurar Policies):**
- Proveedor edita un Listing en `vt-listings-mfe` y navega a la sección Policies.
- Proveedor accede a `/timeline/bookings/:id` y pulsa "Configurar políticas del Listing".

**Capacidad B (cancelar):**
- Cliente navega a `/bookings/:id` y pulsa "Cancelar reserva".
- Proveedor navega a `/timeline/bookings/:id` y pulsa "Cancelar".

**Capacidad C (reagendar):**
- Cliente navega a `/bookings/:id` y pulsa "Reagendar".
- Proveedor navega a `/timeline/bookings/:id` y pulsa "Reagendar".
- Cliente o proveedor recibe notificación "Solicitud de reagendado pendiente" y accede al detalle.

---

## 5. Main Flow

### Capacidad A — Configurar Policies en el Listing

1. Proveedor abre `/listings/:id/edit#policies` (o accede desde el detalle de un Booking).
2. Sistema muestra el panel de Policies con tres secciones: Cancelación / Reagendado / No-Show.
3. Proveedor configura **CancellationPolicy**:
   - `cancellableBy`: attendee | provider | both.
   - Ventanas (`CancellationWindow[]`) ordenadas por `hoursBeforeStart`.
   - Por cada ventana: `refundPercent` (0–100), `creditOnly`, `penaltyAmount?`, `appliesTo` (ambos o solo un actor).
4. Proveedor configura **ReschedulePolicy**:
   - `reschedulableBy`: attendee | provider | both.
   - `windowHours`: anticipación mínima para poder reagendar.
   - `samePriceRequired`: si el nuevo slot debe tener el mismo precio.
   - `requiresApproval`: siempre `true` en v1.
5. Proveedor configura **NoShowPolicy**:
   - `attendeeNoShow.penaltyPercent` y `creditImpact`.
   - `providerNoShow.refundPercent` y `platformAction`.
6. Proveedor guarda. Sistema emite `PolicyAttached` o `PolicyUpdated`.
7. Los Bookings confirmados **después** de este cambio incluirán el nuevo `policySnapshot`.

### Capacidad B — Cancelar Booking (cliente o proveedor)

8.  Actor abre el detalle del Booking (`/bookings/:id` o `/timeline/bookings/:id`).
9.  Actor pulsa **"Cancelar reserva"**.
10. Sistema llama a `EvaluateCancellation(bookingId, cancelledBy, cancelledAt)` usando el `policySnapshot` del Booking.
    - Si no hay Policy configurada: aplica default de plataforma.
    - Si la Policy no permite cancelar al actor que intenta → respuesta 422 con motivo claro.
11. Sistema muestra modal con consecuencias calculadas:
    - Reembolso al cliente: `refundAmount` (importe y tipo: cash o crédito).
    - Penalidad si aplica: `penaltyAmount`.
12. Actor confirma la cancelación.
13. Sistema:
    - Transiciona el Booking a `Cancelled`.
    - Commerce emite reembolso (`IssueRefund(paymentId, refundAmount, refundType)`).
    - Timeline cancela el Event del proveedor.
    - Notificación in-app + email a ambas partes con el detalle del reembolso.

### Capacidad C — Reagendar Booking

#### Modo 1: Cliente propone nuevo slot

14. Cliente abre `/bookings/:id` y pulsa **"Reagendar"**.
15. Sistema evalúa `EvaluateReschedule(bookingId, attendee)` con el `policySnapshot`.
    - Si `reschedulableBy` no incluye `attendee` → 422.
    - Si falta menos de `windowHours` para el slot → 422 con motivo.
16. Sistema muestra el calendario del proveedor con slots disponibles.
    - Si `samePriceRequired=true`: solo muestra slots con precio igual al Booking original.
17. Cliente selecciona nuevo slot y confirma la solicitud.
18. Sistema crea `RescheduleRecord` (status=Pending), aplica soft-hold al nuevo slot.
19. Proveedor recibe notificación "Solicitud de reagendado — [cliente] quiere mover su reserva al [fecha]".
20. Proveedor abre `/timeline/bookings/:id` y ve la solicitud pendiente.
21. Proveedor **confirma** o **rechaza**.
22. **Si confirma:**
    - `RescheduleRecord.status → Confirmed`.
    - Booking se mueve al nuevo slot; slot original liberado; Timeline actualizado.
    - Notificación in-app + email al cliente: "Tu reagendado fue confirmado".
23. **Si rechaza:**
    - `RescheduleRecord.status → Rejected`.
    - Booking permanece en slot original; soft-hold del nuevo slot liberado.
    - Notificación in-app + email al cliente: "Tu solicitud de reagendado fue rechazada".

#### Modo 2: Proveedor propone nuevo slot

24. Proveedor abre `/timeline/bookings/:id` y pulsa **"Reagendar"**.
25. Sistema evalúa `EvaluateReschedule(bookingId, provider)`.
26. Proveedor selecciona nuevo slot del propio calendario y propone el cambio.
27. Sistema crea `RescheduleRecord` (status=Pending), notifica al cliente.
28. Cliente abre `/bookings/:id` y ve la propuesta: "Tu proveedor propone mover tu reserva al [fecha]".
29. Cliente **acepta** o **rechaza**.
30. Efectos idénticos a pasos 22–23 (roles invertidos).

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| CancellationPolicy dice `cancellableBy=attendee` y el proveedor intenta cancelar | 422: "Esta política solo permite cancelación por el cliente" |
| Cliente cancela dentro de la ventana de 0% reembolso | Modal muestra "No aplica reembolso"; cliente puede confirmar igualmente |
| Proveedor cancela `Confirmed` sin Policy configurada | Aplica default de plataforma (100% si >24h, 0% si ≤24h) |
| Reschedule solicitado y el proveedor no responde antes del slot | NEEDS-CLARIFICATION: ¿el sistema auto-rechaza o el Booking se mantiene en slot original? |
| Cliente propone reagendar pero el nuevo slot se llena antes de que el proveedor confirme | Sistema libera el soft-hold; informa al cliente que el slot ya no está disponible |
| `samePriceRequired=true` y el nuevo slot tiene precio diferente | Sistema bloquea la selección del slot; muestra advertencia |
| Booking sin policySnapshot (creado antes de que existiera el módulo Policies) | Sistema aplica default de plataforma; no falla |
| Proveedor actualiza Policy y hay Bookings `Confirmed` existentes | Bookings existentes usan su `policySnapshot`; no se ven afectados |
| Cliente intenta cancelar un Booking `Completed` o `NoShow` | 422: estado terminal, operación rechazada |
| `providerNoShow` con `platformAction=suspension` | NEEDS-CLARIFICATION: proceso de suspensión es manual (Trust & Safety) o automático en v1 |
| Reagendado rechazado, cliente intenta reagendar nuevamente | Permitido si la Policy lo permite; se crea nuevo `RescheduleRecord` |
| Booking con múltiples attendees (Open booking) — uno cancela | NEEDS-CLARIFICATION: ¿cancela solo su participación o el Booking entero? (fuera de scope v1) |

---

## 7. Data Model (v1)

```
-- Módulo: Policies (nuevo, schema: policies)

CancellationPolicy {
  id               UUID
  listingId        UUID
  cancellableBy    attendee | provider | both
  windows          CancellationWindow[]
  appliesTo        all | first_booking | repeat_bookings
  createdAt        DateTime
  updatedAt        DateTime
}

CancellationWindow {
  id               UUID
  policyId         UUID
  hoursBeforeStart int         -- más de N horas antes del slot
  refundPercent    int         -- 0–100
  creditOnly       bool        -- si true: crédito de plataforma, no cash
  penaltyAmount    decimal?    -- cargo fijo al que cancela
  appliesTo        attendee | provider | both
}

ReschedulePolicy {
  id                UUID
  listingId         UUID
  reschedulableBy   attendee | provider | both
  windowHours       int         -- anticipación mínima para reagendar
  maxReschedules    int         -- 0 = ilimitado (v1 siempre 0)
  samePriceRequired bool
  requiresApproval  bool        -- v1: siempre true
  createdAt         DateTime
  updatedAt         DateTime
}

NoShowPolicy {
  id              UUID
  listingId       UUID
  attendeeNoShow  {
    penaltyPercent  int         -- 0–100; % del pago que se retiene
    creditImpact    bool        -- afecta crédito de plataforma del cliente
  }
  providerNoShow  {
    refundPercent   int         -- reembolso al cliente (típicamente 100)
    platformAction  none | warning | suspension
  }
  createdAt       DateTime
  updatedAt       DateTime
}

PolicySnapshot {
  cancellationPolicy  CancellationPolicy?
  reschedulePolicy    ReschedulePolicy?
  noShowPolicy        NoShowPolicy?
  snapshotAt          DateTime
}

-- Extensión del aggregate Booking (módulo Bookings)

Booking {
  ...                          -- campos existentes (Flow 02 + Flow 10)
  policySnapshot        PolicySnapshot?
  rescheduleCount       int    -- veces reagendado (default 0)
  rescheduleHistory     RescheduleRecord[]
}

RescheduleRecord {
  id             UUID
  bookingId      UUID
  requestedBy    attendee | provider
  requestedAt    DateTime
  fromSlot       Slot           -- snapshot del slot original
  proposedSlot   Slot?          -- propuesto; null si solo se solicitó sin propuesta
  status         Pending | Confirmed | Rejected
  confirmedAt    DateTime?
  rejectedAt     DateTime?
  rejectedBy     attendee | provider?
}
```

---

## 8. Commands

### Módulo Policies

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `AttachCancellationPolicy` | `CancellationPolicy` | Listing existe; caller = Owner o Delegado `ManageBookings` |
| `UpdateCancellationPolicy` | `CancellationPolicy` | Policy existe para el Listing |
| `RemoveCancellationPolicy` | `CancellationPolicy` | Policy existe |
| `AttachReschedulePolicy` | `ReschedulePolicy` | Listing existe; caller con permisos |
| `UpdateReschedulePolicy` | `ReschedulePolicy` | Policy existe |
| `AttachNoShowPolicy` | `NoShowPolicy` | Listing existe; caller con permisos |
| `UpdateNoShowPolicy` | `NoShowPolicy` | Policy existe |
| `EvaluateCancellation` (query) | — | Booking existe; tiene policySnapshot o usa default |
| `EvaluateReschedule` (query) | — | Booking Confirmed; policySnapshot o default |

### Módulo Bookings (extensión)

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `CancelBooking` (extendido) | `Booking` | Status = Pending \| Confirmed; Policy permite al actor |
| `RequestReschedule` | `Booking` | Status = Confirmed; Policy permite al actor; dentro de windowHours |
| `ProposeRescheduleSlot` | `Booking` | RescheduleRecord en Pending; slot disponible |
| `ConfirmReschedule` | `Booking` | RescheduleRecord en Pending; confirmedBy = otro actor |
| `RejectReschedule` | `Booking` | RescheduleRecord en Pending |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `PolicyAttached` | `AttachCancellationPolicy / ReschedulePolicy / NoShowPolicy` | — |
| `PolicyUpdated` | `UpdateCancellationPolicy / ReschedulePolicy / NoShowPolicy` | — |
| `BookingCancelled` (con refundAmount) | `CancelBooking` | Commerce: IssueRefund; Timeline: cancela Event; Communication: in-app + email |
| `RescheduleRequested` | `RequestReschedule` | Communication: notifica al otro actor |
| `RescheduleProposed` | `ProposeRescheduleSlot` | Communication: notifica al que solicitó |
| `RescheduleConfirmed` | `ConfirmReschedule` | Timeline: actualiza Event al nuevo slot; Communication: in-app + email |
| `RescheduleRejected` | `RejectReschedule` | Communication: in-app + email al solicitante |
| `BookingNoShow` (extendido) | `MarkNoShow` | Commerce: aplica `NoShowPolicy.attendeeNoShow`; Communication |

---

## 10. Invariants

1. Solo el Owner o Delegado con `ManageBookings` puede configurar Policies en un Listing.
2. `CancellationPolicy.windows` debe cubrir el rango completo (o el default de plataforma completa los huecos).
3. Un Booking confirmado lleva `policySnapshot` al momento de la confirmación; no se actualiza si el Listing cambia sus Policies.
4. Si la Policy dice `cancellableBy=attendee`, el proveedor no puede cancelar (y viceversa).
5. Un reagendado en estado `Pending` bloquea el soft-hold del nuevo slot y mantiene el slot original bloqueado.
6. Solo puede haber un `RescheduleRecord` con status `Pending` por Booking a la vez.
7. `ConfirmReschedule` solo puede ejecutarlo el actor **opuesto** al que solicitó el reagendado.
8. Si `samePriceRequired=true` y el nuevo slot tiene precio diferente, `RequestReschedule` / `ProposeRescheduleSlot` son rechazados.
9. Los estados terminales (`Completed`, `Cancelled`, `NoShow`) no pueden cancelarse ni reagendarse.
10. `BookingCancelled` emitido por este flow incluye `{ refundAmount, penaltyAmount, refundType }` calculados por Policies.

---

## 11. Outputs

- Listing con Policies configuradas (CancellationPolicy, ReschedulePolicy, NoShowPolicy).
- Booking en estado `Cancelled` con reembolso procesado según Policy.
- Booking movido a nuevo slot (reagendado) con `RescheduleRecord.status=Confirmed`.
- Booking en estado original con `RescheduleRecord.status=Rejected` si se rechazó el reagendado.
- Notificaciones in-app + email a ambas partes para cada acción.

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo nuevo:** `Policies` (BC 07)

```
src/Modules/Policies/
├── Policies.Application/
│   └── Commands/
│       ├── AttachCancellationPolicy/
│       ├── UpdateCancellationPolicy/
│       ├── RemoveCancellationPolicy/
│       ├── AttachReschedulePolicy/
│       ├── UpdateReschedulePolicy/
│       ├── AttachNoShowPolicy/
│       └── UpdateNoShowPolicy/
│   └── Queries/
│       ├── EvaluateCancellation/
│       └── EvaluateReschedule/
├── Policies.Domain/
├── Policies.Infrastructure/    -- PoliciesDbContext (schema: policies)
├── Policies.IntegrationEvents/
└── Policies.Presentation/
```

**Extensión módulo `Bookings`:**
```
src/Modules/Bookings/
└── Bookings.Application/
    └── Commands/
        ├── CancelBooking/         -- extendido: llama a EvaluateCancellation
        ├── RequestReschedule/
        ├── ProposeRescheduleSlot/
        ├── ConfirmReschedule/
        └── RejectReschedule/
```

**Endpoints:**

```
-- Policies
GET    /listings/:id/policies
PUT    /listings/:id/policies/cancellation   → AttachCancellationPolicy
PATCH  /listings/:id/policies/cancellation   → UpdateCancellationPolicy
DELETE /listings/:id/policies/cancellation   → RemoveCancellationPolicy
PUT    /listings/:id/policies/reschedule     → AttachReschedulePolicy
PATCH  /listings/:id/policies/reschedule     → UpdateReschedulePolicy
PUT    /listings/:id/policies/no-show        → AttachNoShowPolicy
PATCH  /listings/:id/policies/no-show        → UpdateNoShowPolicy

-- Cancel (extensión)
GET    /bookings/:id/cancel/preview          → EvaluateCancellation (dry-run)
POST   /bookings/:id/cancel                  → CancelBooking (ahora usa Policies)

-- Reschedule
POST   /bookings/:id/reschedule/request      → RequestReschedule
POST   /bookings/:id/reschedule/propose      → ProposeRescheduleSlot
POST   /bookings/:id/reschedule/confirm      → ConfirmReschedule
POST   /bookings/:id/reschedule/reject       → RejectReschedule
GET    /bookings/:id/reschedule/history      → historial de RescheduleRecords
```

### Frontend

**MFE:** `vt-timeline-mfe` (rutas nuevas para cliente) y `vt-listings-mfe` (policies config)

```
-- Cliente (attendee) — vt-timeline-mfe
/bookings                       → lista de Bookings del usuario como attendee
/bookings/:id                   → detalle + CTAs cancel/reschedule

-- Proveedor (extensión de Flow 10) — vt-timeline-mfe
/timeline/bookings/:id          → extendido: CTAs reagendar + cancel con preview Policy

-- Policies — vt-listings-mfe
/listings/:id/edit#policies     → panel de configuración de Policies
```

**UI States:**

**`/listings/:id/edit#policies`:**
- Sección Cancelación: form de ventanas con `hoursBeforeStart`, `refundPercent`, `appliesTo`.
- Sección Reagendado: toggles `reschedulableBy`, `windowHours`, `samePriceRequired`.
- Sección No-Show: sliders `penaltyPercent`, toggle `creditImpact`, `platformAction`.
- Preview en tiempo real de la Policy configurada.

**`/bookings/:id` y `/timeline/bookings/:id` — modal cancelar:**
- Preview de consecuencias calculadas: reembolso + penalidad.
- Confirmación con botón explícito.

**`/bookings/:id` — reagendar (cliente):**
- Vista de calendario del proveedor con slots disponibles (filtrados por `samePriceRequired`).
- Estado "Reagendado pendiente de confirmación" mientras espera respuesta.

**`/timeline/bookings/:id` — reagendar (proveedor):**
- CTA "Reagendar" abre selector de slot del propio calendario.
- Badge de solicitudes de reagendado pendientes.

---

## 13. Acceptance Criteria

- [ ] Proveedor puede configurar CancellationPolicy con múltiples ventanas por actor.
- [ ] Proveedor puede configurar ReschedulePolicy (quién puede, ventana, samePriceRequired).
- [ ] Proveedor puede configurar NoShowPolicy (penalidad al cliente, acción si proveedor falta).
- [ ] Al confirmar un Booking, se guarda `policySnapshot` con las Policies vigentes del Listing.
- [ ] Al cancelar, el sistema muestra un preview del reembolso calculado antes de confirmar.
- [ ] El reembolso calculado usa el `policySnapshot` del Booking, no las Policies actuales del Listing.
- [ ] Si no hay Policy configurada, se aplica el default de plataforma (>24h=100%, ≤24h=0%).
- [ ] Si la Policy no permite cancelar al actor que intenta, el sistema devuelve 422 con motivo.
- [ ] Cliente puede solicitar reagendado seleccionando un nuevo slot disponible.
- [ ] Proveedor puede proponer un nuevo slot al cliente.
- [ ] La otra parte (proveedor o cliente) puede confirmar o rechazar el reagendado.
- [ ] Al confirmar reagendado: Booking se mueve al nuevo slot; slot original liberado; Timeline actualizado.
- [ ] Al rechazar reagendado: Booking permanece en slot original; ambas partes notificadas.
- [ ] Solo puede haber un RescheduleRecord Pending por Booking a la vez.
- [ ] Si `samePriceRequired=true`, el sistema bloquea la selección de slots con precio diferente.
- [ ] Cambiar Policies en un Listing no afecta Bookings ya confirmados (usan su snapshot).
- [ ] Delegado con `ManageBookings` puede configurar Policies y gestionar reagendados.
- [ ] Un usuario sin `ManageBookings` recibe 403 al intentar configurar Policies del Owner.

---

## 14. NEEDS-CLARIFICATION

- **Reschedule sin respuesta antes del slot:** Si hay un `RescheduleRecord` Pending y llega la fecha del slot original, ¿el sistema auto-rechaza el reagendado y confirma el slot original, o hace otra cosa?
- **`providerNoShow.platformAction=suspension`:** ¿La suspensión es un proceso manual gestionado por Trust & Safety, o el sistema la aplica automáticamente? En v1 recomiendo `none` como único valor implementado.
- **Open Bookings y cancelación parcial:** Si un Booking tiene múltiples attendees (Open model), ¿cancelar significa que un attendee individual cancela su participación, o que se cancela el Booking completo? Impacta el modelo de datos y está fuera de scope v1 (1:1 services).
- **`penaltyAmount` a quién se cobra:** Si la CancellationPolicy define `penaltyAmount`, ¿se cobra al que cancela (débito de balance/crédito de plataforma) o es una retención del pago ya realizado?
- **Precio distinto en reagendado (`samePriceRequired=false`):** Si el nuevo slot tiene precio diferente, ¿Commerce procesa el ajuste automáticamente (cobro o reembolso de la diferencia), o el proveedor maneja la diferencia fuera de la plataforma?
