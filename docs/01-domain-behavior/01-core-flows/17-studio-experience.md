---
title: Flow 17 — Studio Experience (Create, Manage, Analyze)
description: >
  El usuario accede al tab Studio y gestiona sus offerings (Service+Listing unificados),
  ve solicitudes pendientes de booking con acciones rápidas, y consulta estadísticas
  básicas. Un wizard de 5 pasos oculta la complejidad interna Service/Listing.
  Progressive disclosure adapta la experiencia: usuario sin offerings ve un empty state
  invitador; provider activo ve dashboard operativo.
status: draft
version: 1
last-reviewed: 2026-03-20
---

# Flow 17 — Studio Experience (Create, Manage, Analyze)

## 1. Summary

| | |
|---|---|
| **Goal** | Que el usuario cree, edite y gestione sus offerings (Service+Listing) en un espacio unificado, sin ver la complejidad interna del modelo de dominio. Gestionar solicitudes de booking con acciones rápidas. Consultar estadísticas básicas. |
| **Actors** | Usuario autenticado (todo usuario potencial provider); Delegado con ManageListings (via Agreement) |
| **Out of scope** | Booking flow completo del consumidor (Flow 02), configuración de Timeline/ConflictRules (Flow 08), configuración de Agreements (Flow 09), moderación (Flow 14), pagos/checkout (NC-05) |
| **Surfaces** | `vt-provider-mfe`: `/studio` (home), `/studio/create` (wizard), `/studio/offering/:id` (edit), `/studio/requests` (booking requests), `/studio/stats` (analytics) |
| **Decision ref** | ADR-0005 (Navigation Architecture — Studio es tab 3); ADR-0003 (Service/Listing boundary) |

---

## 2. Domain Context

### "Offering" es lo que el usuario ve

El usuario nunca ve "Service" ni "Listing" como conceptos separados. Ve un **offering** — la unidad que crea, edita y publica. Internamente:

| Lo que el usuario dice | Lo que el sistema crea |
|---|---|
| "Mi servicio" / "Mi oferta" | `Service` (reglas operativas) + `Listing` (estrategia comercial) |
| "Nombre, duración, precio base" | → `Service.name`, `Service.durationMin`, `Service.basePrice` |
| "Fotos, descripción, tags" | → `Listing.media[]`, `Listing.description`, `Listing.tags[]` |
| "Disponibilidad, ubicación" | → `Listing.placeId`, `Listing.slotConfig` + `Service.preBufferMin/postBufferMin` |
| "Publicar / Pausar / Archivar" | → `Service.status` + `Listing.status` (synced) |

> **Regla de UX:** 1 Service = 1 Listing = 1 "offering" para el 95% de usuarios.
> El caso 1:N (múltiples Listings por Service) es feature de power user (futuro).

### Módulos involucrados

- **Catalog** (Service creation, lifecycle Draft → Active → Archived)
- **Listing** (commercial wrapper, lifecycle Draft → Published → Unpublished → Archived)
- **Booking** (incoming requests, confirmation policy)
- **Timeline** (availability — link a "Configurar mi disponibilidad")
- **Place** (location assignment for service delivery)
- **Social** (LISTING_SHARE posts when publishing)

---

## 3. Preconditions

| # | Condición |
|---|---|
| PRE-01 | Usuario autenticado |
| PRE-02 | Profile completado (nombre, avatar como mínimo) |
| PRE-03 | Al menos un Timeline existente (creado automáticamente en onboarding) |
| PRE-04 | Para publicar: al menos un Place configurado (o "Virtual" seleccionado) |

---

## 4. Trigger

El usuario toca el tab **Studio** en la barra de navegación inferior (3er tab, ADR-0005).

---

## 5. Main Flow

### Capability A — Studio Home (Dashboard)

El Studio Home es un **dashboard operativo** con tres secciones verticales. No es un admin panel — es un espacio de creación que crece con el usuario.

#### Layout

```
┌─────────────────────────────────────────┐
│  Studio                     [+ Crear]   │
├─────────────────────────────────────────┤
│                                         │
│  ┌─ Solicitudes pendientes (N) ───────┐ │
│  │ [Avatar] Juan quiere Masaje        │ │
│  │ Mañana 15:00  [✓ Aprobar] [✗]     │ │
│  │ [Avatar] María pidió Consulta      │ │
│  │ Vie 10:00     [✓ Aprobar] [✗]     │ │
│  │              [Ver todas →]         │ │
│  └────────────────────────────────────┘ │
│                                         │
│  ┌─ Mis Offerings ────────────────────┐ │
│  │ [img] Consulta General    ● Live   │ │
│  │       $5000 · 30 min · 12 bookings │ │
│  │ [img] Masaje Relajante    ● Live   │ │
│  │       $8000 · 60 min · 8 bookings  │ │
│  │ [img] Consulta Express    ○ Draft  │ │
│  │       $3000 · 15 min              │ │
│  └────────────────────────────────────┘ │
│                                         │
│  ┌─ Esta semana ──────────────────────┐ │
│  │  15 bookings  │  $120k  │  342     │ │
│  │  confirmados   │ ingresos│  vistas  │ │
│  └────────────────────────────────────┘ │
│                                         │
└─────────────────────────────────────────┘
```

#### Secciones

**1. Solicitudes pendientes (top priority)**
- Card compacto con avatar del solicitante, nombre del offering, fecha/hora propuesta.
- Acciones inline: botón Aprobar (✓) y Rechazar (✗).
- Solo visible si hay solicitudes con `confirmationPolicy: ManualConfirm`.
- Si `AutoConfirm`, las solicitudes se confirman solas — esta sección no muestra nada.
- Link "Ver todas →" navega a `/studio/requests`.

**2. Mis Offerings (lista principal)**
- Cards verticales con: thumbnail, nombre, precio, duración, status badge, booking count (últimos 30 días).
- Status badges: `● Live` (Published), `○ Draft`, `⏸ Pausado` (Unpublished), `📦 Archivado`.
- Tap en card → navega a `/studio/offering/:id` (edición).
- Floating Action Button "+" o header button "Crear" → navega a `/studio/create`.

**3. Esta semana (stats snapshot)**
- Tres métricas en una fila: bookings confirmados, ingresos estimados, vistas de listings.
- Tap → navega a `/studio/stats` para detalle.
- Solo visible si tiene al menos 1 booking histórico (activity-gating, patrón YouTube Studio).

#### Responsive

| Breakpoint | Layout |
|---|---|
| Mobile (< 768px) | Single column, sections stacked |
| Tablet (768-1024px) | Solicitudes + Stats side by side, offerings debajo |
| Desktop (> 1024px) | Three-column: Solicitudes (left), Offerings (center), Stats (right) |

---

### Capability B — Empty State (First Visit)

Cuando el usuario abre Studio por primera vez sin offerings:

```
┌─────────────────────────────────────────┐
│  Studio                                 │
├─────────────────────────────────────────┤
│                                         │
│           [Ilustración invitadora]      │
│                                         │
│     Tu espacio de creación te espera    │
│                                         │
│   Creá tu primer servicio y empezá a    │
│   recibir reservas. Solo necesitás:     │
│                                         │
│   ✦ Un nombre                           │
│   ✦ Cuánto dura                         │
│   ✦ Cuánto cuesta                       │
│                                         │
│        [ Crear mi primer offering ]     │
│                                         │
│   "Podés publicar en menos de 2 min"    │
│                                         │
└─────────────────────────────────────────┘
```

**Principios del empty state** (basado en research: Booksy, Gumroad, Square):
- **Single CTA** — un solo botón de acción, no múltiples opciones.
- **Show structure, not blankness** — la ilustración comunica el potencial.
- **Quantify the effort** — "solo necesitás 3 cosas" reduce la ansiedad de empezar.
- **Social proof (futuro)** — "1,200 providers ya crearon su primer offering esta semana".

El empty state **desaparece para siempre** después de crear el primer offering (incluso en Draft).

---

### Capability C — Crear Offering (Wizard de 5 Pasos)

El wizard sigue el patrón Airbnb: **un tema por pantalla**, con progress bar arriba y "Guardar y salir" siempre visible. Internamente crea un `Service (Draft)` + `Listing (Draft)` atómicamente.

#### Progress Bar

```
[Lo básico] → [Precio y duración] → [Lugar y horarios] → [Fotos y detalles] → [Revisar y publicar]
    ●               ○                     ○                      ○                    ○
```

#### Paso 1 — Lo básico

| Campo | Destino interno | Requerido | UI |
|---|---|---|---|
| Nombre del offering | `Service.name` + `Listing.title` | ✅ | Text input, max 100 chars |
| Descripción corta | `Service.description` | No | Textarea, max 300 chars |
| Categoría | `Listing.tags[]` (curated taxonomy) | ✅ | Search-select con categorías predefinidas |
| Tags adicionales | `Listing.tags[]` (custom) | No | Tag input, max 5 |

**Validación:** Solo nombre es requerido para avanzar. El usuario puede guardar Draft con solo el nombre.

**Auto-save:** Cada paso guarda automáticamente (patrón Airbnb). Si el usuario sale y vuelve, retoma exactamente donde estaba.

#### Paso 2 — Precio y duración

| Campo | Destino interno | Requerido | UI |
|---|---|---|---|
| Duración | `Service.durationMin` | ✅ | Preset chips (15, 30, 45, 60, 90, 120 min) + custom input |
| Precio base | `Service.basePrice` | ✅ | Currency input con selector de moneda |
| Confirmación | `Listing.confirmationPolicy` | ✅ | Toggle: "Confirmar automáticamente" (AutoConfirm) / "Revisar cada solicitud" (ManualConfirm) |

**Defaults inteligentes:**
- Duración: 60 min pre-seleccionado (Calendly pattern — most common duration).
- Confirmación: AutoConfirm pre-seleccionado (reduce fricción para el provider novato).

**Sección expandible "Opciones avanzadas"** (patrón Calendly "More options"):
- Buffer pre/post servicio: `Service.preBufferMin`, `Service.postBufferMin` (default: 0).
- Ventana de anticipación mínima (default: 2 horas).
- Ventana máxima de reserva futura (default: 30 días).

#### Paso 3 — Lugar y disponibilidad

| Campo | Destino interno | Requerido | UI |
|---|---|---|---|
| Lugar | `Listing.placeId` | ✅ | Radio: "En mi local/consultorio" (select Place) / "Virtual" / "A domicilio" |
| Disponibilidad | Link a Timeline config | Info | Card informativa: "Tu disponibilidad se configura en Timeline → [Ir a Timeline]" |

**Nota sobre disponibilidad:** El wizard NO replica la configuración de Timeline/ConflictRules. Muestra un resumen visual de la disponibilidad actual y un CTA para ajustarla. Esta decisión evita duplicar lógica compleja (Flow 08).

**Si no tiene Places configurados:**
- Mostrar mini-wizard inline: "¿Dónde atendés?" → Nombre del lugar + Dirección + Timezone → Se crea el Place on-the-fly.
- O "Virtual" que no requiere Place físico.

#### Paso 4 — Fotos y detalles

| Campo | Destino interno | Requerido | UI |
|---|---|---|---|
| Fotos/Video | `Listing.media[]` | No (recomendado) | Drag-and-drop upload, hasta 10 fotos + 1 video |
| Descripción completa | `Listing.description` | No | Rich text editor (bold, lists, links) |
| Política de cancelación | `Listing.cancellationPolicy` | No | Select: Flexible / Moderada / Estricta (NEEDS-CLARIFICATION: exact policies) |

**Nudging sin bloquear:**
- Si no sube fotos: badge "Incompleto" en el listing card (patrón Fiverr quality score), pero permite publicar.
- "Los offerings con fotos reciben 3x más reservas" (social proof micro-copy).

**Integración cámara (mobile):** Botón de cámara directo para tomar fotos desde el wizard (patrón Depop).

#### Paso 5 — Revisar y publicar

```
┌──────────────────────────────────────────────┐
│  Así se va a ver tu offering                 │
├──────────────────────────────────────────────┤
│                                              │
│  ┌─ Preview Card (como aparece en feed) ──┐  │
│  │ [Foto]                                 │  │
│  │ Consulta General                       │  │
│  │ [Avatar] Dr. García  ★ 4.8 (12)       │  │
│  │ 📍 Consultorio Centro  ⏱ 30 min       │  │
│  │             [Desde $5000 · Reservar]   │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Resumen:                                    │
│  • Nombre: Consulta General                  │
│  • Precio: $5,000                            │
│  • Duración: 30 minutos                      │
│  • Lugar: Consultorio Centro                 │
│  • Confirmación: Automática                  │
│  • Fotos: 3 subidas                          │
│  • [⚠ Falta descripción completa]           │
│                                              │
│  [Guardar como borrador]  [✦ Publicar]       │
│                                              │
└──────────────────────────────────────────────┘
```

**Preview:** Renderiza el offering card exactamente como aparecerá en el feed (patrón Calendly "Preview"). Incluye foto, nombre, avatar del provider, rating (o "Nuevo" si no tiene reviews), ubicación, duración, precio, y botón "Reservar".

**Acciones:**
- **Publicar** → `Service.activate()` + `Listing.publish()`. Emite `ListingPublished`. Opcionalmente genera un post automático (LISTING_SHARE) en el feed del provider.
- **Guardar como borrador** → Ambos quedan en Draft. El offering aparece en "Mis Offerings" con badge `○ Draft`.

**Validación al publicar:**
Requiere todos los campos marcados como ✅ en los pasos anteriores. Si falta algo, scroll automático al paso incompleto con indicador visual.

---

### Capability D — Editar Offering

Después de crear, el offering se edita en una **vista de página completa con secciones colapsables** (patrón Shopify card-based). No un wizard — acceso directo a cualquier sección.

```
┌──────────────────────────────────────────────┐
│  ← Volver    Consulta General    [⋮ Más]     │
├──────────────────────────────────────────────┤
│                                              │
│  Status: ● Published  [Pausar] [Archivar]    │
│                                              │
│  ▼ Lo básico                                 │
│    Nombre: Consulta General        [Editar]  │
│    Categoría: Salud > Médico       [Editar]  │
│                                              │
│  ▶ Precio y duración  ── $5000 · 30 min      │
│                                              │
│  ▶ Lugar y disponibilidad  ── Consultorio     │
│                                              │
│  ▼ Fotos y detalles                          │
│    [foto1] [foto2] [foto3] [+ Agregar]       │
│    Descripción: "Consulta médica general..." │
│                                              │
│  ▶ Estadísticas  ── 12 bookings, 342 vistas  │
│                                              │
│  ▶ Opciones avanzadas  ── 2 configuradas      │
│                                              │
└──────────────────────────────────────────────┘
```

**Secciones colapsables** con badge resumen cuando están cerradas (ej: "2 configuradas" para opciones avanzadas). Cada sección se expande inline para edición — sin navegar a otra pantalla.

**Menú "Más" (overflow):**
- Duplicar offering (crea nuevo Draft con todos los campos copiados — patrón Calendly/Acuity).
- Ver como visitante (preview público).
- Compartir link directo al listing.
- Archivar (con confirmación — bookings existentes se preservan).

**Status transitions desde la vista de edición:**
- Published → Unpublished (botón "Pausar"): confirmación suave "Los usuarios no podrán reservar mientras esté pausado".
- Published → Archived (botón "Archivar"): confirmación modal: "Esta acción no se puede deshacer. Los bookings existentes no se afectan."
- Unpublished → Published (botón "Reactivar"): inmediato, sin confirmación.
- Draft → Published (botón "Publicar"): validación de campos requeridos.

---

### Capability E — Gestión de Solicitudes

La vista de solicitudes (`/studio/requests`) muestra booking requests que requieren acción del provider (cuando `confirmationPolicy = ManualConfirm`).

```
┌──────────────────────────────────────────────┐
│  ← Studio    Solicitudes                     │
├──────────────────────────────────────────────┤
│  [Pendientes (3)]  [Aprobadas]  [Rechazadas] │
├──────────────────────────────────────────────┤
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ [Avatar] Juan Pérez                   │  │
│  │ Masaje Relajante · 60 min             │  │
│  │ 📅 Mar 24 Mar · ⏰ 15:00 - 16:00     │  │
│  │ 📍 Mi Studio                          │  │
│  │ "Tengo dolor en la espalda baja"      │  │
│  │                                       │  │
│  │ [Mensaje]  [✗ Rechazar]  [✓ Aprobar]  │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ [Avatar] María López                  │  │
│  │ Consulta General · 30 min             │  │
│  │ 📅 Vie 26 Mar · ⏰ 10:00 - 10:30     │  │
│  │ 📍 Consultorio Centro                 │  │
│  │                                       │  │
│  │ [Mensaje]  [✗ Rechazar]  [✓ Aprobar]  │  │
│  └────────────────────────────────────────┘  │
│                                              │
└──────────────────────────────────────────────┘
```

**Acciones rápidas inline (patrón Slack message actions):**
- **Aprobar** → `ConfirmBookingCommand`. Toast: "Booking confirmado. Juan recibirá notificación."
- **Rechazar** → Bottom sheet con razón opcional + `RejectBookingCommand`. Toast: "Solicitud rechazada."
- **Mensaje** → Abre chat/mensaje directo con el solicitante (NEEDS-CLARIFICATION: messaging module).

**Tabs de filtrado:**
- **Pendientes** (default): solicitudes que esperan respuesta.
- **Aprobadas**: booking confirmados recientes (últimos 7 días).
- **Rechazadas**: rechazados recientes (últimos 7 días).

**Push notifications:** Cada nueva solicitud genera push notification con acción rápida en la notificación misma (aprobar/rechazar desde la notificación, sin abrir la app — patrón Booksy).

**Expiración:** Las solicitudes tienen TTL configurable. Si el provider no responde en X horas, el sistema puede auto-cancelar con mensaje al solicitante (NEEDS-CLARIFICATION: TTL default y behavior).

---

### Capability F — Estadísticas Básicas

Vista simple de analytics (`/studio/stats`) para providers pequeños. No es un BI dashboard — es un resumen motivacional.

```
┌──────────────────────────────────────────────┐
│  ← Studio    Estadísticas                    │
├──────────────────────────────────────────────┤
│  [Esta semana ▾]                             │
├──────────────────────────────────────────────┤
│                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │    15    │ │  $120k   │ │   342    │     │
│  │ Bookings │ │ Ingresos │ │  Vistas  │     │
│  │  ↑ 20%  │ │  ↑ 15%   │ │  ↑ 8%   │     │
│  └──────────┘ └──────────┘ └──────────┘     │
│                                              │
│  ┌─ Bookings por día ─────────────────────┐  │
│  │  [Mini bar chart - últimos 7 días]     │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌─ Top offerings ────────────────────────┐  │
│  │  1. Consulta General    8 bookings     │  │
│  │  2. Masaje Relajante    5 bookings     │  │
│  │  3. Consulta Express    2 bookings     │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌─ Tasa de confirmación ─────────────────┐  │
│  │  92% de solicitudes aprobadas          │  │
│  │  Tiempo promedio de respuesta: 14 min  │  │
│  └────────────────────────────────────────┘  │
│                                              │
└──────────────────────────────────────────────┘
```

**Métricas key (patrón Instagram Insights — solo lo esencial):**
- Bookings confirmados en el período.
- Ingresos estimados (sum of `Booking.priceSnapshot`).
- Vistas de listings (impressions en feed/marketplace).
- Comparación con período anterior (↑/↓ %).

**Filtros temporales:**
- Esta semana (default).
- Este mes.
- Últimos 30 días.
- Custom range (futuro).

**Activity-gating:** Esta sección solo aparece cuando el provider tiene al menos 1 booking completado. Antes de eso, el espacio muestra tips o guías (patrón YouTube Studio progressive cards).

---

## 6. Alternate & Edge Cases

| # | Caso | Comportamiento |
|---|---|---|
| ALT-01 | Usuario sin offerings toca Stats | No navega — sección no visible. Solo CTA de crear |
| ALT-02 | Wizard interrumpido (app cerrada) | Auto-save preserva progreso. Al volver, bottom sheet "Tenés un offering en progreso — ¿Continuar?" |
| ALT-03 | Publicar sin Place | Bloqueado. Paso 3 resalta error: "Elegí dónde ofrecés este servicio" |
| ALT-04 | Publicar sin fotos | Permitido con nudge: badge "Incompleto" + tooltip "Agregá fotos para más reservas" |
| ALT-05 | Delegado (Agreement) crea offering | El offering se crea bajo el Profile del owner, no del delegado. UI muestra "Creando para [owner name]" |
| ALT-06 | Offering con bookings activos → Archivar | Confirmación explícita: "Hay N bookings pendientes. Archivar no los cancela — se mantendrán activos." |
| ALT-07 | Editar precio de offering Published | Cambio inmediato para nuevas reservas. Bookings existentes mantienen `priceSnapshot` original |
| ALT-08 | Solicitud de booking expirada | Card en gris con label "Expirada". No se puede aprobar. |
| ALT-09 | Provider con 50+ offerings | Barra de búsqueda en "Mis Offerings" + filtros por status (Live, Draft, Pausado) |
| ALT-10 | Network error durante wizard | Auto-save offline. Al reconectar, sync automático. Toast: "Se guardó tu progreso" |

---

## 7. Data Model (Resumen — ver domain docs para detalle completo)

### Service (Catalog module)

```
Service {
  id: UUID
  profileId: UUID                    // owner
  status: Draft | Active | Archived
  name: string                       // ← Paso 1
  description: string?               // ← Paso 1
  durationMin: int                   // ← Paso 2
  basePrice: decimal                 // ← Paso 2
  basePriceCurrency: string          // ← Paso 2
  preBufferMin: int?                 // ← Paso 2 (advanced)
  postBufferMin: int?                // ← Paso 2 (advanced)
  // -- confirmationPolicy: vive en Listing (decisión v1)
  // -- intakeForm: vive en Listing (decisión v1)
  // -- media: vive en Listing (decisión v1)
  // -- location: vive en Listing como placeId (decisión v1)
}
```

### Listing (Listing module)

```
Listing {
  id: UUID
  serviceId: UUID?                   // link al Service (opcional en draft)
  profileId: UUID                    // owner
  status: Draft | Published | Unpublished | Archived
  title: string                      // ← synced from Service.name initially
  description: string?               // ← Paso 4 (rich text)
  media: MediaItem[]                 // ← Paso 4 (max 5)
  placeId: UUID?                     // ← Paso 3
  price: decimal                     // override de Service.basePrice
  currency: string                   // "ARS", "USD"
  confirmationPolicy: AutoConfirm | ManualConfirm | RequestOnly  // ← Paso 2
  visibility: Public | Private
  capacity: int                      // default: 1
  tags: Tag[]                        // ← Paso 1
  slotConfig: SlotConfiguration      // durationMin, buffers, bookingWindow
  intakeForm: FieldDefinition[]      // preguntas al reservar
  addOns: AddOn[]                    // extras opcionales
  recurrence: ListingRecurrence      // weekly/nweeks/specificDates
  // Note: Channels (audiencias comunidad-style) deferidos post-v1 — ver ADR-0007
}
```

### Unified Lifecycle (Offering = Service + Listing)

> **NOTA (actualización v1):** No hay endpoint compuesto `CreateOfferingCommand`.
> El frontend orquesta `POST /catalog/services` + `POST /listings` por separado.
> Esto permite múltiples paths de creación (service-first, listing-first, unified wizard).

```
[Wizard Start]
    │
    │  POST /catalog/services → Service(Draft)
    │  POST /listings → Listing(Draft) con serviceId
    │
    ▼
[Draft] ── editSteps() → [Draft] (auto-save via PATCH per section)
    │
    │  POST /catalog/services/:id/publish → Service(Active)
    │  POST /listings/:id/publish → Listing(Published)
    │
    ▼
[Published] ←��─ unpublish() ──→ [Unpublished]
    │                                   │
    │  archive()                        │  POST /listings/:id/publish
    │                                   │
    ▼                                   │
[Archived] (terminal)    ←──────────────┘
```

---

## 8. Queries

| Query | Params | Returns | Used in |
|---|---|---|---|
| `GetMyOfferings` | `profileId`, `status?`, `page` | List of Offering (Service + Listing joined) | Studio Home |
| `GetOfferingDetail` | `listingId` | Full Service + Listing + stats | Edit view |
| `GetPendingBookingRequests` | `profileId`, `status`, `page` | List of BookingRequest with requester info | Solicitudes |
| `GetStudioStats` | `profileId`, `period` | Aggregated metrics (bookings, revenue, views) | Stats view |
| `GetOfferingDraft` | `listingId` | Draft state for wizard resume | Wizard resume |
| `GetMyPlaces` | `profileId` | List of configured Places | Wizard Paso 3 |

---

## 9. Commands

> **NOTA (actualización v1):** No hay commands compuestos tipo `CreateOffering`.
> El frontend orquesta los endpoints individuales de Catalog y Listing modules.

| Command | Endpoint | Effect | Events emitted |
|---|---|---|---|
| `CreateService` | `POST /catalog/services` | Creates Service(Draft) | `ServiceCreated` |
| `CreateListing` | `POST /listings` | Creates Listing(Draft) con serviceId opcional | `ListingCreated` |
| `UpdateService` | `PATCH /catalog/services/:id` | Updates Service fields | `ServiceUpdated` |
| `UpdateListing` | `PATCH /listings/:id` | Updates Listing fields | `ListingUpdated` |
| `PublishService` | `POST /catalog/services/:id/publish` | Service(Draft) → Active | `ServicePublished` |
| `PublishListing` | `POST /listings/:id/publish` | Listing(Draft) → Published | `ListingPublished` |
| `UnpublishListing` | `POST /listings/:id/unpublish` | Published → Unpublished | `ListingUnpublished` |
| `ArchiveService` | `POST /catalog/services/:id/archive` | Service → Archived | `ServiceArchived` |
| `ArchiveListing` | `POST /listings/:id/archive` | Listing → Archived | `ListingArchived` |
| `ConfirmBookingRequest` | bookingId | BookingRequest.confirm() | `BookingConfirmed` |
| `RejectBookingRequest` | bookingId, reason? | BookingRequest.reject() | `BookingRejected` |

---

## 10. Events (Domain Events)

| Event | Raised by | Consumed by |
|---|---|---|
| `OfferingDraftCreated` | Catalog + Listing | — |
| `ServiceActivated` | Catalog | Listing (triggers slot generation), Timeline |
| `ListingPublished` | Listing | Feed (auto-post LISTING_SHARE), Search (index), Discovery |
| `ListingUnpublished` | Listing | Feed (remove from feed), Search (de-index) |
| `ServiceArchived` / `ListingArchived` | Catalog / Listing | Search (de-index), Discovery (remove) |
| `BookingConfirmed` | Booking | Notifications (to requester), Timeline (block slot) |
| `BookingRejected` | Booking | Notifications (to requester), Timeline (release slot) |

---

## 11. Invariants

1. Un offering (Service + Listing) pertenece a exactamente un Profile.
2. Para publicar: `Service.name` no vacío, `durationMin > 0`, `basePrice ≥ 0`, `confirmationPolicy` definido, `placeId` asignado (o Virtual).
3. Solo `Service.Active` admite `Listing.Published`.
4. `Archived` es terminal — no se puede reactivar (duplicar sí).
5. Archivar no cancela bookings existentes.
6. Cambios en precio/duración no afectan bookings existentes (snapshot protege al consumidor).
7. Auto-save no emite domain events — solo la acción explícita de publicar/pausar/archivar.
8. Un Service sin Listings no es visible en discovery (pero puede existir como Draft interno).
9. Delegados con `ManageListings` pueden crear offerings bajo el Profile del owner — la autoría se audita.

---

## 12. Technical Mapping

### MFE

| MFE | Routes | Purpose |
|---|---|---|
| `vt-provider-mfe` | `/studio` | Studio Home dashboard |
| `vt-provider-mfe` | `/studio/create` | Wizard de creación (5 pasos) |
| `vt-provider-mfe` | `/studio/offering/:id` | Vista de edición |
| `vt-provider-mfe` | `/studio/requests` | Gestión de solicitudes |
| `vt-provider-mfe` | `/studio/stats` | Estadísticas básicas |

### Toolkit Components Needed

| Component | Description | Priority |
|---|---|---|
| `vt-offering-card` | Card de offering con thumbnail, stats badge, status indicator | High |
| `vt-request-card` | Card de booking request con acciones inline | High |
| `vt-wizard-stepper` | Progress indicator para wizard multi-paso | High |
| `vt-stat-card` | Métrica con valor, label, y trend (↑/↓) | Medium |
| `vt-media-uploader` | Drag-and-drop upload con preview y reorder | Medium |
| `vt-status-badge` | Badge de estado (Live, Draft, Pausado, Archivado) | Low |

### Backend Queries (new)

| Query | Module | Description |
|---|---|---|
| `GetMyOfferings` | Catalog + Listing (joined) | Paginated list of Service+Listing for a profile |
| `GetPendingBookingRequests` | Booking | Pending requests for a provider's listings |
| `GetStudioStats` | Booking + Listing (read model) | Aggregated metrics for a period |

---

## 13. Acceptance Criteria

| # | Criterio | Verifiable by |
|---|---|---|
| AC-01 | Usuario sin offerings ve empty state con CTA único | Visual inspection |
| AC-02 | Wizard de 5 pasos crea Service(Draft) + Listing(Draft) atómicamente | Integration test |
| AC-03 | Auto-save persiste progreso entre sesiones | E2E test |
| AC-04 | Publicar valida campos requeridos y transiciona ambos estados | Unit test + E2E |
| AC-05 | Preview en paso 5 renderiza card idéntica al feed | Visual regression |
| AC-06 | Studio Home muestra solicitudes pendientes con acciones inline | E2E test |
| AC-07 | Aprobar solicitud emite `BookingConfirmed` y notifica al requester | Integration test |
| AC-08 | Rechazar solicitud requiere confirmación y emite `BookingRejected` | Integration test |
| AC-09 | Stats muestra métricas correctas para el período seleccionado | Integration test |
| AC-10 | Stats no aparece para providers sin bookings históricos | E2E test |
| AC-11 | Duplicar offering crea nuevo Draft con todos los campos copiados | Integration test |
| AC-12 | Archivar offering no cancela bookings existentes | Integration test |
| AC-13 | Editar precio no afecta bookings existentes (snapshot) | Unit test |
| AC-14 | Mobile: wizard es un tema por pantalla, swipeable | Visual inspection |
| AC-15 | Delegado puede crear offerings bajo el Profile del owner | Integration test |

---

## 14. NEEDS-CLARIFICATION

| # | Pregunta | Impacto |
|---|---|---|
| NC-01 | ¿Cuál es el TTL default para solicitudes de booking no respondidas? | Capability E — expiración de requests |
| NC-02 | ¿Estructura exacta del `intakeForm`? ¿Builder visual o template predefinido? | Paso 2 advanced, Capability D |
| NC-03 | ¿Módulo de messaging existe? ¿Cómo se integra "Mensaje" en solicitudes? | Capability E — botón Mensaje |
| NC-04 | ¿Políticas de cancelación predefinidas? ¿Flexible/Moderada/Estricta? ¿Qué implica cada una? | Paso 4, Listing domain |
| NC-05 | ¿El auto-post LISTING_SHARE al publicar es opt-in o automático? | Capability C — paso 5 |
| NC-06 | ¿Stats endpoint es read model dedicado o queries ad-hoc sobre Booking module? | Capability F, backend |
| NC-07 | ¿Moneda default? ¿Multi-currency support en v1? | Paso 2 — pricing |
