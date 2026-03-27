# API Contracts — Booking

Base path: `/api/v1`

---

## Overview

The Booking flow spans multiple bounded contexts:
1. **Offering** provides the Listing (rules, SlotConfig)
2. **Timelines** provides available Slots
3. **Booking** manages the Booking lifecycle
4. **Communication** sends notifications

See `01-domain-behavior/01-core-flows/02-open-and-book.md` for the end-to-end flow.

---

## Create a Booking

### `POST /bookings` 🚧

Creates a Booking for a given Slot and Listing.

The command used depends on `Listing.confirmationPolicy`:
- `AutoConfirm` → dispatches `CreateBooking` → Booking starts as **Confirmed** → event `BookingCreated`
- `ManualConfirm | RequestOnly` → dispatches `RequestBooking` → Booking starts as **Pending** → event `BookingRequested`

> **Updated 2026-03-26:** Field name corrected from `bookingRules.confirmationRequired` (boolean) to `confirmationPolicy` (enum).
> Status corrected from `Requested` to `Pending`. See `current-product-truth.md` CONFLICT-01 and CONFLICT-02.

**Request:**
```json
{
  "listingId": "uuid",
  "slotId": "uuid",
  "profileId": "uuid",
  "attendeeCount": 1,
  "notes": "string | null"
}
```

**Response (auto-confirmed):**
```json
{
  "bookingId": "uuid",
  "status": "Confirmed",
  "listingId": "uuid",
  "slotId": "uuid",
  "startAt": "ISO8601",
  "endAt": "ISO8601",
  "placeId": "uuid | null",
  "eventId": "uuid"
}
```

**Response (awaiting confirmation):**
```json
{
  "bookingId": "uuid",
  "status": "Pending",
  "listingId": "uuid",
  "slotId": "uuid",
  "startAt": "ISO8601",
  "endAt": "ISO8601"
}
```

---

## Soft-hold a Slot

### `POST /bookings/hold` 🚧

Creates a soft-hold on a Slot with TTL (default: 5 min). Used when the user selects a slot before completing the booking form.

> **Added 2026-03-26:** Previously defined in Flow 02 but missing from this contract. See `current-product-truth.md` CONFLICT-04.

**Request:**
```json
{
  "listingId": "uuid",
  "slotId": "uuid"
}
```

**Response:** `201 Created`
```json
{
  "holdId": "uuid",
  "slotId": "uuid",
  "expiresAt": "ISO8601"
}
```

**Behavior:**
- Backend creates a Booking in `Holding` state with TTL.
- If TTL expires: backend auto-cancels the hold, slot becomes available again.
- Frontend should display countdown timer.

**Command dispatched:** `HoldSlotCommand`
**Event emitted:** `SlotHeld`

---

### `DELETE /bookings/hold/:holdId` 🚧

Releases a soft-hold before TTL expiry (e.g., user navigates away).

**Command dispatched:** `ReleaseHoldCommand`
**Event emitted:** `SlotHoldReleased`

---

## Get a Booking

### `GET /bookings/:bookingId` 🚧

Returns a Booking. Visible to the attendee and the provider.

**Response:**
```json
{
  "bookingId": "uuid",
  "status": "Pending | Confirmed | Cancelled | Completed | NoShow",
  "listingId": "uuid",
  "listingTitle": "string",
  "profileId": "uuid",
  "providerId": "uuid",
  "startAt": "ISO8601",
  "endAt": "ISO8601",
  "placeId": "uuid | null",
  "attendeeCount": 1,
  "notes": "string | null",
  "createdAt": "ISO8601"
}
```

---

## List Bookings

### `GET /bookings` 🚧

Returns Bookings for the authenticated user (as attendee or provider).

**Query params:** `?role=attendee|provider&status=Confirmed&page=1&pageSize=20`

---

## Provider: confirm a Booking

### `POST /bookings/:bookingId/confirm` 🚧

Provider confirms a `Requested` Booking.

**Command dispatched:** `ConfirmBooking`
**Event emitted:** `BookingConfirmed`

---

## Cancel a Booking

### `POST /bookings/:bookingId/cancel` 🚧

Cancels a Booking. Either the attendee or the provider can cancel.

**Request:**
```json
{
  "reason": "string | null"
}
```

**Command dispatched:** `CancelBooking`
**Event emitted:** `BookingCancelled`

> **Note:** Cancellation policies (penalties, refunds, windows) are applied per `Listing.bookingRules`. See `07-policies/` *(P0 — pending)*.

---

## Provider: mark no-show

### `POST /bookings/:bookingId/no-show` 🚧

Marks a `Confirmed` Booking as NoShow. Only the provider can call this.

**Command dispatched:** `MarkNoShow`
**Event emitted:** `BookingNoShow`

---

## Provider: complete a Booking

### `POST /bookings/:bookingId/complete` 🚧

Marks a `Confirmed` Booking as Completed. Only the provider can call this.

**Command dispatched:** `CompleteBooking`
**Event emitted:** `BookingCompleted`
