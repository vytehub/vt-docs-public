# API Contracts — Timelines & Availability

Base path: `/api/v1`

---

## Timelines

### `GET /timelines` 🚧
Returns Timelines owned by the authenticated Profile.

**Response:**
```json
{
  "items": [
    {
      "timelineId": "uuid",
      "profileId": "uuid",
      "name": "string",
      "timezoneId": "string",
      "type": "personal | resource | team"
    }
  ]
}
```

---

### `GET /timelines/:timelineId` 🚧
Get a single Timeline. Visibility enforced by Access module.

---

### `GET /timelines/:timelineId/events` 🚧
Get Events on a Timeline for a date range.

**Query params:** `?from=ISO8601&to=ISO8601`

**Response:**
```json
{
  "items": [
    {
      "eventId": "uuid",
      "timelineId": "uuid",
      "title": "string",
      "startAt": "ISO8601",
      "endAt": "ISO8601",
      "type": "manual | booking | block",
      "bookingId": "uuid | null",
      "placeId": "uuid | null",
      "visibility": "public | busyOnly | private"
    }
  ]
}
```

---

### `POST /timelines/:timelineId/events` 🚧
Create a manual Event (block, personal appointment, etc.).

**Request:**
```json
{
  "title": "string",
  "startAt": "ISO8601",
  "endAt": "ISO8601",
  "type": "manual | block",
  "placeId": "uuid | null",
  "visibility": "public | busyOnly | private"
}
```

**Command dispatched:** `UpsertEvents`
**Event emitted:** `EventCreated`

---

### `PATCH /timelines/:timelineId/events/:eventId` 🚧
Update a manual Event.

**Command dispatched:** `UpsertEvents`
**Event emitted:** `EventUpdated`

---

### `DELETE /timelines/:timelineId/events/:eventId` 🚧
Cancel/delete a manual Event.

**Command dispatched:** `CancelEvent`
**Event emitted:** `EventCancelled`

---

## Availability (Slots)

### `GET /availability` 🚧
Query available slots for a Listing within a date range.

This is the primary endpoint for booking UIs and search results.

**Query params:**
```
?listingId=uuid
&from=ISO8601
&to=ISO8601
&timezoneId=string   (optional; defaults to Listing's Place timezone)
```

**Command dispatched:** `QueryAvailability` (read-only)

**Response:**
```json
{
  "listingId": "uuid",
  "timezoneId": "string",
  "slots": [
    {
      "slotId": "uuid",
      "startAt": "ISO8601",
      "endAt": "ISO8601",
      "available": true,
      "remainingCapacity": 1
    }
  ]
}
```

---

### `POST /timelines/:timelineId/project-slots` 🚧
Manually trigger slot re-projection for a Listing on a Timeline.

Normally triggered automatically by `ListingPublished` / `ListingUpdated` events. Use this for manual refresh or debugging.

**Request:**
```json
{
  "listingId": "uuid",
  "fromDate": "ISO8601",
  "toDate": "ISO8601"
}
```

**Command dispatched:** `ProjectSlots`
**Event emitted:** `SlotsProjected`

---

## Conflict Rules

### `GET /timelines/:timelineId/conflict-rules` 🚧
Get the Conflict Rules configured for a Timeline.

---

### `PUT /timelines/:timelineId/conflict-rules` 🚧
Replace the Conflict Rules for a Timeline.

**Request:**
```json
{
  "rules": [
    {
      "eventTypes": ["string"],
      "action": "block | busyOnly | requireConfirmation",
      "priority": 1
    }
  ]
}
```

**Command dispatched:** `ConfigureConflictRules`
