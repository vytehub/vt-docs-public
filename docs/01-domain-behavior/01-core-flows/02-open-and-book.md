---
title: Flow 02 — Browse Listing and Book Slot
description: >
  Un usuario autenticado navega al detalle de un Listing publicado,
  selecciona un Slot disponible, completa el intake form y confirma la reserva.
  El sistema crea un Booking (Pending o Confirmed según confirmationPolicy)
  y crea el Event correspondiente en el Timeline del proveedor.
status: draft
version: 1
last-reviewed: 2026-03-12
---

# Flow 02 — Browse Listing and Book Slot

## 1. Summary

| | |
|---|---|
| **Goal** | Permitir que un usuario autenticado reserve un slot en un Listing publicado, resultando en un Booking activo y un Event linked a los Timelines del proveedor y del cliente. |
| **Actors** | Guest/User (cliente), Sistema VyteMerge |
| **Out of scope** | Gestión del lado del proveedor (Flow 10), pagos (v1 out of scope), reagendamiento y cancelación (Flow 11), listings `FollowersOnly/Restricted/PartnerOnly` (NEEDS-CLARIFICATION) |
| **Surfaces** | `vt-catalog-mfe`: Listing Detail, Slot Picker, Booking Form, Booking Confirmation |

---

## 2. Domain Context

### Bounded Contexts involucrados

| BC | Módulo backend | Contenido |
|---|---|---|
| **Catalog BC** | `Catalog` | `Service` — definición operativa (duración, precio base, rules). Fuente interna. |
| **Offer BC** | `Listing` | `Listing` — cómo se vende (copy, canales, scheduling, reglas comerciales). Lo que el cliente ve. |
| **Booking BC** | `Booking` | `Booking` — reserva del cliente. Guarda snapshots al confirmar. |
| **Supply BC** | `Timelines` | `Timeline` — configuración de agenda (privacy, conflict rules, members). |
| **Events BC** | `Events` | `Event` — entidad independiente que ocupa tiempo. Creado al confirmar booking, linked a timelines de proveedor y cliente (ADR-0006). |

> ⚠️ El módulo `Offering` (legacy) debe descomponerse en `Catalog` + `Listing` + `Booking`.
> Ver **ADR Follow-up** al final de este documento.

### Slot vs Event

- **Slot**: proyección calculada de disponibilidad (scheduling + eventos ocupados). Efímero.
- **Event**: entidad independiente (ADR-0006) que ocupa tiempo. Creado cuando `Booking → Confirmed`, linked a timelines de proveedor y cliente via `EventTimelineLink`.
- Los Slots son referencia; los Events son la fuente de verdad de disponibilidad bloqueada.

### confirmationPolicy

| `confirmationPolicy` | Booking inicial | Evento emitido | Event (ADR-0006) |
|---|---|---|---|
| `AutoConfirm` | `Confirmed` (inmediato) | `BookingCreated` | Event creado y linked a timelines de proveedor + cliente |
| `ManualConfirm` | `Pending` (espera proveedor) | `BookingRequested` | Event creado en Flow 10 al aprobar y linked a timelines |

---

## 3. Preconditions

1. `Listing.status = Published`
2. `Listing.visibility ∈ {Public, Private}` (`Private` accesible solo por link directo)
3. `Access.VisibilityMode(actor, listing) ≠ None` (actor no está bloqueado)
4. Para reservar: `Access.VisibilityMode(actor, listing) = BusyOnlyDetails`
5. Existen Slots proyectados disponibles en el rango solicitado
6. Usuario autenticado (sesión Keycloak válida)

---

## 4. Trigger

El usuario navega a Listing Detail:
- Desde feed / discovery (listing `Public`)
- Desde link directo (listing `Private` o `Public`)
- Desde deep link a un Slot específico

---

## 5. Main Flow

### Paso 1 — Cargar Listing Detail

1. Frontend hace `GET /listings/{listingId}`.
2. Backend (`Listing` module) evalúa:
   - `Listing.status ≠ Published` → `404`
   - `Access.VisibilityMode = None` → `403`
   - `Access.VisibilityMode = BusyOnly` → retorna preview degradado (sin slots ni precio exacto; CTA de reserva oculto)
   - `Access.VisibilityMode = BusyOnlyDetails` → retorna detalle completo
3. Frontend renderiza: título, descripción, precio, media, timezone del Place (default).

### Paso 2 — Seleccionar Slot

1. Frontend hace `GET /listings/{listingId}/slots?from=YYYY-MM-DD&to=YYYY-MM-DD`.
2. Backend retorna Slots disponibles (proyección sobre scheduling + `Confirmed + Pending` bookings activos).
3. Usuario selecciona un Slot en el Slot Picker.
4. Frontend muestra el slot con **timezone del Place/Listing** por defecto.
   - Si el usuario tiene otro timezone configurado en su perfil: se muestra en ambos.
   - Si el usuario cambia el timezone en la UI: se muestra tooltip → *"Estás viendo este horario en [tz-elegida]. El evento ocurrirá en [tz-del-place]."*

### Paso 3 — Completar Intake Form + Soft-hold

1. Al seleccionar slot, frontend hace `POST /bookings/hold` con `{ listingId, slotId }`.
   - Backend crea Booking en `Holding` con TTL (default: 5 min).
   - Frontend muestra countdown timer.
   - Si TTL expira: backend cancela el `Holding`, slot liberado, UI notifica.
2. Frontend renderiza Booking Form:
   - **Campos base** (autofill si autenticado): `name`, `lastName`, `email`
   - **Campos custom** (si `Listing.formId` definido): renderizados dinámicamente
   - Si `Listing.formRequired = true`: bloquear submit hasta completar

### Paso 4 — Confirmar Booking

1. Usuario hace submit del form.
2. Frontend hace `POST /bookings` con `{ listingId, slotId, holdId, intake, formSubmissionId? }`.
3. Backend (`Booking` module):
   a. Revalida hold activo y pertenece al actor.
   b. Revalida disponibilidad del slot (race condition check).
   c. Evalúa capacidad (Pool fijo: `PoolAvailable ≥ 1`; Pool dinámico: matching de staff).
   d. **Auto-asignación de staff** (Pool dinámico): sistema asigna automáticamente; transparente al cliente.
   e. Guarda **snapshots inmutables**:
      - `ServiceSnapshot`: duración, precio base, policy
      - `ListingSnapshot`: precio efectivo, terms, copy
      - `FormSnapshot`: `formId`, `formVersion`, `submissionId`
   f. Ejecuta comando según `confirmationPolicy`:
      - `AutoConfirm` → `CreateBooking` → Booking a `Confirmed`
      - `ManualConfirm` → `RequestBooking` → Booking a `Pending`
4. Si `Confirmed`: publica `BookingCreated`; módulo `Events` crea Event (type=booking) y lo linkea a los timelines del proveedor y del cliente via `EventTimelineLink`.
5. Si `Pending`: publica `BookingRequested`; Event se crea en Flow 10 cuando el proveedor apruebe.

### Paso 5 — Confirmación UI

1. Frontend navega a `/booking/{id}/confirmation`.
2. Muestra:
   - Detalle del booking (servicio, fecha/hora, lugar)
   - Estado: `Confirmed` o `Pending awaiting provider approval`
   - Timezone del event con nota si difiere del browser
3. Notificaciones emitidas:
   - **Al cliente**: in-app + email + push
   - **Al proveedor**: in-app + email + push

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|---|---|
| Slot tomado antes de confirmar (race) | Backend → `409`; UI: "Este slot ya no está disponible. Elige otro." |
| Soft-hold TTL expirado | Backend cancela `Holding`, slot liberado; UI: "Tu reserva temporal venció. Vuelve a elegir un slot." |
| Listing unpublished mientras usuario está en pantalla | `POST /bookings` → `422`; UI: "Este listing ya no está disponible." |
| Form required no completado | Frontend bloquea submit; backend retorna `422` con detalle de campos si llega igual |
| Pool dinámico sin staff disponible | Backend → `409`; UI: "No hay disponibilidad para este servicio en el horario seleccionado." |
| Hold de otro actor / inválido | Backend → `403`; frontend redirige a slot picker |
| Usuario no autenticado intenta reservar | Redirect a login; post-login regresa al Listing Detail |
| `VisibilityMode = BusyOnly` | Listing visible en preview; CTA de reserva oculto |

---

## 7. Data Model (v1 minimal)

### Booking (schema: `booking`)

```
Booking {
  id:                  UUID (PK)
  listingId:           UUID        -- FK lógica → Listing
  slotId:              UUID        -- referencia al slot proyectado
  clientProfileId:     UUID        -- FK lógica → Users
  status:              Holding | Pending | Confirmed | Cancelled | Completed | NoShow
  confirmationPolicy:  AutoConfirm | ManualConfirm   -- snapshot del Listing
  holdExpiresAt:       DateTime?   -- solo cuando status = Holding

  -- Intake base
  clientName:          string
  clientLastName:      string
  clientEmail:         string

  -- Snapshots inmutables
  serviceSnapshot:     JSONB       -- duración, precio base, policy
  listingSnapshot:     JSONB       -- precio efectivo, terms, copy
  formSnapshot:        JSONB?      -- formId, formVersion, submissionId

  -- Staff (transparente al cliente en v1)
  assignedStaffId:     UUID?

  -- Timeline link
  timelineEventId:     UUID?       -- FK lógica → Event; null hasta Confirmed

  createdAt:           DateTime
  updatedAt:           DateTime
}
```

### Slot (proyección — no persistida como entidad primaria)

> **DEPRECATED (2026-03-26):** This conceptual model uses a `status` enum. The canonical API response shape
> uses `available: boolean` + `remainingCapacity: int`. See `api-contracts/timelines.md` (Availability section)
> and `work/vt-work/current-product-truth.md` CONFLICT-03.

```
Slot {
  id:          UUID
  listingId:   UUID
  start:       DateTime (UTC)
  end:         DateTime (UTC)
  timezoneId:  string     -- del Place/Listing
  status:      Available | Holding | Booked | Blocked
}
```

---

## 8. Commands

| Comando | Módulo | Descripción |
|---|---|---|
| `HoldSlot` | `Booking` | Crea Booking en `Holding` con TTL |
| `ReleaseHold` | `Booking` | Libera hold manualmente (usuario cancela) |
| `RequestBooking` | `Booking` | Crea Booking en `Pending` (ManualConfirm). Requiere hold activo. |
| `CreateBooking` | `Booking` | Crea Booking en `Confirmed` (AutoConfirm). Requiere hold activo. |
| `ExpireHold` | `Booking` | Sistema cancela Booking `Holding` expirado (scheduled job) |

---

## 9. Events

| Evento | Emisor | Consumidores |
|---|---|---|
| `SlotHeld` | `Booking` | `Listing` (marca slot como Holding) |
| `HoldExpired` | `Booking` | `Listing` (libera slot) |
| `BookingRequested` | `Booking` | `Notifications`, Flow 10 |
| `BookingCreated` | `Booking` | `Events` (crea Event + links a timelines), `Notifications` |

---

## 10. Invariants

1. Booking `Confirmed` debe tener `timelineEventId` asociado.
2. No pueden coexistir dos Bookings en `Confirmed + Pending` para el mismo slot salvo que el Pool lo permita.
3. `HoldSlot` falla si el slot ya tiene Booking en `Holding`, `Pending` o `Confirmed`.
4. `CreateBooking` y `RequestBooking` requieren hold activo del mismo actor.
5. Snapshots de Service, Listing y Form son inmutables post-confirmación.
6. `ExpireHold` solo ejecutable si `status = Holding AND holdExpiresAt ≤ now()`.
7. Un cliente no puede tener más de un `Holding` activo para el mismo Listing simultáneamente.

---

## 11. Outputs

| Output | Cuándo |
|---|---|
| Booking en `Confirmed` | AutoConfirm exitoso |
| Booking en `Pending` | ManualConfirm (esperando proveedor) |
| Event linked a timelines (proveedor + cliente) | Solo cuando `Confirmed` |
| Notificación al cliente | Siempre |
| Notificación al proveedor | Siempre |

---

## 12. Technical Mapping (Draft)

### Backend

**Módulos involucrados:**

| Módulo | Rol en este flow |
|---|---|
| `Listing` | Queries: listing detail, slots disponibles |
| `Booking` | Comandos: hold, request, create, expire |
| `Events` | Crea Event + EventTimelineLinks via integration event `BookingCreated` |
| `Access` | Evalúa `VisibilityMode` por actor |
| `Users` | Autofill intake desde perfil autenticado |

**Endpoints:**

| Método | Ruta | Handler | Módulo |
|---|---|---|---|
| `GET` | `/listings/{id}` | `GetListingQuery` | `Listing` |
| `GET` | `/listings/{id}/slots` | `GetAvailableSlotsQuery` | `Listing` |
| `POST` | `/bookings/hold` | `HoldSlotCommand` | `Booking` |
| `DELETE` | `/bookings/hold/{holdId}` | `ReleaseHoldCommand` | `Booking` |
| `POST` | `/bookings` | `CreateBookingCommand` / `RequestBookingCommand` | `Booking` |
| `GET` | `/bookings/{id}` | `GetBookingQuery` | `Booking` |

**Notas:**
- `HoldSlot` requiere idempotency key (anti double-submit).
- Validation behaviors auto-registrados; no agregar por handler.
- Integration event `BookingCreated` → handler en `Events` crea Event + links a timelines de proveedor y cliente (ADR-0006).

### Frontend (vt-catalog-mfe)

**Rutas:**

| Ruta | Componente | Estado UI |
|---|---|---|
| `/listing/:id` | `ListingDetailPage` | Detalle + Slot Picker |
| `/listing/:id/book` | `BookingFormPage` | Intake form + countdown |
| `/booking/:id/confirmation` | `BookingConfirmationPage` | Confirmed / Pending |

**Componentes nuevos a construir en vt-toolkit:**

| Componente | Descripción |
|---|---|
| `VtSlotPicker` | Grid/calendario de slots con estados (available, holding, booked, blocked) |
| `VtBookingForm` | Form dinámico: campos base + campos custom del Form template |
| `VtTimezoneSelector` | Selector de timezone con tooltip explicativo |
| `VtBookingConfirmation` | Card de confirmación con estado Confirmed / Pending |
| `VtCountdownTimer` | Timer de cuenta regresiva para soft-hold |

**UI States:** `loading` → `preview-only` → `slot-selecting` → `form-active` → `submitting` → `confirmed | pending | error`

---

## 13. Acceptance Criteria

```gherkin
Scenario: AutoConfirm booking exitoso
  Given Listing.confirmationPolicy = AutoConfirm y slot disponible
  And usuario autenticado
  When selecciona slot, completa intake y confirma
  Then Booking.status = Confirmed
  And Event creado y linked a timelines de proveedor y cliente
  And notificaciones enviadas a cliente y proveedor (in-app + email + push)

Scenario: ManualConfirm booking pendiente
  Given Listing.confirmationPolicy = ManualConfirm
  When usuario confirma
  Then Booking.status = Pending
  And NO se crea Event (se crea al confirmar en Flow 10)
  And UI muestra "awaiting provider approval"
  And proveedor recibe notificación

Scenario: Race condition — slot tomado
  Given dos usuarios intentan el mismo slot simultáneamente
  When el segundo confirma
  Then backend retorna 409
  And UI: "Este slot ya no está disponible"

Scenario: Soft-hold expirado
  Given usuario tiene hold activo
  When TTL expira antes de confirmar
  Then Booking.Holding cancelado, slot liberado
  And UI notifica al usuario

Scenario: Form required incompleto
  Given Listing.formRequired = true
  When usuario intenta confirmar sin completar
  Then frontend bloquea submit y muestra errores por campo

Scenario: Usuario no autenticado
  Given usuario sin sesión
  When intenta reservar
  Then redirect a login
  And post-login regresa a Listing Detail

Scenario: Pool dinámico sin staff
  Given no hay staff disponible para el slot
  When usuario intenta confirmar
  Then backend retorna 409
  And UI: "No hay disponibilidad para este servicio en el horario seleccionado"
```

---

## 14. NEEDS-CLARIFICATION

| ID | Pregunta | Impacto |
|---|---|---|
| NC-01 | `visibility = FollowersOnly / Restricted / PartnerOnly`: ¿cómo evaluar eligibility? | Bloquea soporte de listings no-public |
| NC-02 | Módulo `Attendance` existente: ¿se fusiona en `Booking` o es separado? | Define estructura del módulo Booking |
| NC-03 | Open Booking (partido público con MinToConfirm): ¿en scope de este flow o flow separado? | Open bookings requieren lógica adicional |
| NC-04 | ¿Se persiste el timezone elegido por el usuario en el Booking? | Afecta display histórico |
| NC-05 | Pagos: ¿cuándo se define el checkout flow? | Bloquea listings con pago obligatorio |

---

## ADR Follow-up — Descomposición del módulo `Offering`

> **Acción requerida antes de implementar este flow.**

El módulo `Offering` actual debe descomponerse en tres módulos independientes:

| Módulo nuevo | BC | Schema | Contenido |
|---|---|---|---|
| `Catalog` | Catalog BC | `catalog` | `Service`, `Product` |
| `Listing` | Offer BC | `listing` | `Listing`, scheduling config, Slot (proyección) |
| `Booking` | Booking BC | `booking` | `Booking`, `RescheduleRecord`, soft-hold |

**Steps:**
```bash
./scripts/new-module.sh Catalog
./scripts/new-module.sh Listing   # per ADR-0003
./scripts/new-module.sh Booking
# Migrar entidades de Offering → módulos correspondientes
# Eliminar módulo Offering
# Actualizar workspace.manifest.json y Program.cs
```

**Referencias:** ADR-0003 (`private/decisions/ADR-0003-listing-module-boundary.md`)
