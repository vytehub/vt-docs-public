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

The command used depends on `Listing.bookingRules.confirmationRequired`:
- `false` → dispatches `CreateBooking` → Booking starts as **Confirmed** → event `BookingCreated`
- `true` → dispatches `RequestBooking` → Booking starts as **Requested** → event `BookingRequested`

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
  "status": "Requested",
  "listingId": "uuid",
  "slotId": "uuid",
  "startAt": "ISO8601",
  "endAt": "ISO8601"
}
```

---

## Get a Booking

### `GET /bookings/:bookingId` 🚧

Returns a Booking. Visible to the attendee and the provider.

**Response:**
```json
{
  "bookingId": "uuid",
  "status": "Requested | Confirmed | Cancelled | Completed | NoShow",
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
