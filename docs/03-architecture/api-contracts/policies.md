# API Contracts — Policies

Base path: `/api/v1`
Module: `Vt.Modules.Policies` *(o extendido desde Listing module — a definir)*
Schema: `policies`

> Cubre políticas de cancelación, reagendado y no-show asociadas a Listings.
> Las políticas se evalúan en el momento de la cancelación para calcular el reembolso.
> Ver flow `11-cancelar-reagendar-policies.md` y spec pack `DRAFT-cancel-reschedule-policies`.

---

## Cancellation Policies

### Policy Types

| Tipo | Ventana de reembolso completo | Ventana de reembolso parcial |
|------|------------------------------|------------------------------|
| `Flexible` | Hasta 24h antes | — |
| `Moderate` | Hasta 5 días antes | 24h–5 días: 50% |
| `Strict` | Hasta 7 días antes | 24h–7 días: 0% |
| `Custom` | Definido en `customRules` | Definido en `customRules` |

### `GET /listings/:listingId/cancellation-policy` 🚧

Retorna la política de cancelación efectiva de un Listing. Pública.

**Response:**
```json
{
  "listingId": "uuid",
  "type": "Flexible | Moderate | Strict | Custom",
  "fullRefundHours": 24,
  "partialRefundPercent": 0,
  "partialRefundHours": null,
  "noShowPolicy": {
    "enabled": false,
    "feePercent": 0
  },
  "customRules": "string | null"
}
```

---

### `POST /listings/:listingId/cancellation-policy` 🚧

Define o reemplaza la política de cancelación de un Listing.

**Auth:** Owner del Listing o Delegado con `ManageListings`.

**Request:**
```json
{
  "type": "Flexible | Moderate | Strict | Custom",
  "noShowPolicy": {
    "enabled": false,
    "feePercent": 0
  },
  "customRules": "string | null"
}
```

> `customRules` requerido si `type=Custom`. Si `type` es predefinido, se ignora `customRules`.

**Command dispatched:** `SetCancellationPolicyCommand`
**Event emitted:** `CancellationPolicyUpdated`

**Response:** `200 OK` con la política actualizada.

---

### `POST /bookings/:bookingId/evaluate-cancellation` 🚧

Evalúa cuánto se reembolsaría si se cancela el Booking ahora mismo.
Útil para mostrar el impacto antes de confirmar la cancelación.

**Auth:** Attendee o Provider del Booking.

**Response:**
```json
{
  "bookingId": "uuid",
  "cancellationPolicy": "Flexible | Moderate | Strict | Custom",
  "hoursUntilBooking": 48,
  "refundPercent": 100,
  "refundAmount": { "amount": 5000, "currency": "ARS" },
  "eligibleToCancel": true
}
```

---

## Reschedule Policies

### `GET /listings/:listingId/reschedule-policy` 🚧

Retorna la política de reagendado del Listing. Pública.

**Response:**
```json
{
  "listingId": "uuid",
  "allowed": true,
  "minNoticeHours": 24,
  "maxReschedulesPerBooking": 1,
  "providerMustApprove": false
}
```

---

### `POST /listings/:listingId/reschedule-policy` 🚧

Define o reemplaza la política de reagendado.

**Auth:** Owner del Listing o Delegado con `ManageListings`.

**Request:**
```json
{
  "allowed": true,
  "minNoticeHours": 24,
  "maxReschedulesPerBooking": 1,
  "providerMustApprove": false
}
```

**Command dispatched:** `SetReschedulePolicyCommand`
**Event emitted:** `ReschedulePolicyUpdated`

---

## Reschedule Requests

### `POST /bookings/:bookingId/reschedule-requests` 🚧

Attendee propone reagendar el Booking a un nuevo slot.

**Auth:** Attendee del Booking.

**Request:**
```json
{
  "newSlotId": "uuid",
  "reason": "string | null"
}
```

**Command dispatched:** `RequestRescheduleCommand`
**Events emitted:**
- Si `providerMustApprove=false` → `RescheduleConfirmed` (automático)
- Si `providerMustApprove=true` → `RescheduleRequested`

**Errors:**
- `422` si no se alcanza la ventana mínima (`minNoticeHours`).
- `422` si ya se alcanzó `maxReschedulesPerBooking`.
- `422` si `reschedule.allowed=false` en el Listing.

---

### `POST /bookings/:bookingId/reschedule-requests/:requestId/confirm` 🚧

Provider confirma un reagendado pendiente.

**Auth:** Provider del Booking.

**Command dispatched:** `ConfirmRescheduleCommand`
**Event emitted:** `RescheduleConfirmed` → Booking se actualiza al nuevo slot; notificaciones.

---

### `POST /bookings/:bookingId/reschedule-requests/:requestId/reject` 🚧

Provider rechaza un reagendado pendiente.

**Auth:** Provider del Booking.

**Request:**
```json
{ "reason": "string | null" }
```

**Command dispatched:** `RejectRescheduleCommand`
**Event emitted:** `RescheduleRejected` → Booking permanece en el slot original; notificación al Attendee.

---

## NoShow Policy

Configurado como parte de la `CancellationPolicy` via `noShowPolicy.feePercent`.

Cuando el Provider ejecuta `POST /bookings/:id/no-show`:
- Si `noShowPolicy.enabled=true`: se cobra `feePercent` del total al Attendee.
- Si `false`: no hay cargo adicional.

Ver también: `booking.md` → `POST /bookings/:id/no-show`.
