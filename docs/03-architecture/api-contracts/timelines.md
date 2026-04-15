# API Contracts — Timelines, Events & Availability

Base path: `/api/v1`

> **ADR-0006:** Event es entidad independiente linked a N Timelines. Timeline es configuración pura.
> Los endpoints se organizan por módulo backend: **Timelines** (config) y **Events** (tiempo + availability).

---

## Timelines (módulo Timelines — configuración de agenda)

### `GET /timelines`
Returns Timelines owned by the authenticated Profile.

**Response:**
```json
{
  "items": [
    {
      "timelineId": "uuid",
      "profileId": "uuid",
      "name": "string",
      "type": "personal | resource | team",
      "privacy": "public | followersOnly | private"
    }
  ]
}
```

---

### `GET /timelines/:timelineId`
Get a single Timeline configuration. Visibility enforced by Access module.

---

### `POST /timelines`
Create a new Timeline.

**Request:**
```json
{
  "name": "string",
  "type": "personal | resource | team",
  "privacy": "public | followersOnly | private",
  "isDefault": false
}
```

---

### `PATCH /timelines/:timelineId/privacy`
Update Timeline privacy level.

**Request:**
```json
{
  "privacy": "public | followersOnly | private"
}
```

---

## Conflict Rules (módulo Timelines)

### `GET /timelines/:timelineId/conflict-rules`
Get the Conflict Rules configured for a Timeline.

**Response:**
```json
{
  "items": [
    {
      "ruleId": "uuid",
      "sourceTimelineId": "uuid",
      "action": "notifyOnly | hideSlots",
      "createdAt": "ISO8601"
    }
  ]
}
```

---

### `POST /timelines/:timelineId/conflict-rules`
Add a Conflict Rule (observe another timeline for overlaps).

**Request:**
```json
{
  "sourceTimelineId": "uuid"
}
```

**Command dispatched:** `AddConflictRule`

---

### `DELETE /timelines/conflict-rules/:ruleId`
Remove a Conflict Rule and its associated detections.

**Command dispatched:** `RemoveConflictRule`

---

## Events (módulo Events — lo que ocupa tiempo)

### `GET /timelines/:timelineId/events`
Get Events linked to a Timeline for a date range.

> Nota: aunque la ruta es `/timelines/:timelineId/events`, este endpoint lo sirve el módulo **Events** (query por EventTimelineLink).

**Query params:** `?from=ISO8601&to=ISO8601`

**Response:**
```json
{
  "items": [
    {
      "eventId": "uuid",
      "title": "string",
      "startAt": "ISO8601",
      "endAt": "ISO8601",
      "type": "manual | booking | block | external",
      "sourceId": "uuid | null",
      "placeId": "uuid | null",
      "visibility": "public | busyOnly | private",
      "status": "active | cancelled",
      "linkedTimelines": [
        {
          "timelineId": "uuid",
          "role": "primary | viewer | readonly"
        }
      ]
    }
  ]
}
```

---

### `POST /timelines/:timelineId/events`
Create a manual Event (block, personal appointment, etc.) linked to the specified Timeline.

**Request:**
```json
{
  "title": "string",
  "startAt": "ISO8601",
  "endAt": "ISO8601",
  "type": "manual | block",
  "placeId": "uuid | null",
  "visibility": "public | busyOnly | private",
  "notes": "string | null"
}
```

**Command dispatched:** `AddEvent`
**Event emitted:** `EventCreated`

---

### `PATCH /timelines/:timelineId/events/:eventId`
Update a manual Event (only events with type=manual|block can be updated).

**Command dispatched:** `UpdateEvent`
**Event emitted:** `EventUpdated`

---

### `DELETE /timelines/:timelineId/events/:eventId`
Cancel/delete a manual Event.

**Command dispatched:** `RemoveEvent`
**Event emitted:** `EventCancelled`

---

## Conflict Detections (módulo Events)

### `GET /timelines/:timelineId/conflicts`
Get pending conflict detections for a Timeline.

**Response:**
```json
{
  "items": [
    {
      "conflictDetectionId": "uuid",
      "conflictRuleId": "uuid",
      "sourceEventId": "uuid",
      "targetEventId": "uuid",
      "status": "pending | resolved | dismissed",
      "detectedAt": "ISO8601"
    }
  ]
}
```

---

### `POST /timelines/conflicts/:conflictDetectionId/resolve`
Resolve a detected conflict.

**Request:**
```json
{
  "resolution": "keepAll | cancelTarget | rescheduleTarget"
}
```

**Command dispatched:** `ResolveConflict`

---

## Availability / Slots (módulo Events — query de proyección)

### `GET /availability`
Query available slots for a Listing within a date range.

This is the primary endpoint for booking UIs and search results. Slot Projection lives in the Events module because it has the occupied events locally.

**Query params:**
```
?listingId=uuid
&from=ISO8601
&to=ISO8601
&timezoneId=string   (optional; defaults to Listing's Place timezone)
```

**Projection algorithm:**
1. Read Listing scheduling info (recurrence + slotConfig) via `Listing.PublicApi`
2. Generate candidate slots from recurrence rules
3. Subtract occupied Events (local query)
4. Apply ConflictRules (read from `Timelines.PublicApi`)
5. Apply capacity constraints (per-listing + SharedCapacityGroup)
6. Return available slots

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
