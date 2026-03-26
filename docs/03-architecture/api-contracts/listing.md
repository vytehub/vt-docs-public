# API Contracts — Listing

Base path: `/api/v1`
Module: `Vt.Modules.Listing`
Schema: `listing`

> Un Listing expone un Service comercialmente: precio efectivo, canales de distribución, slotConfig, visibility y add-ons.
> **1 Service → N Listings.** Discovery usa Listings. Booking usa el Service (snapshot).
> Ver flow `06-crear-listing.md` y spec pack `DRAFT-create-listing` para reglas completas.

---

## Create

### `POST /listings` 🚧

Crea un Listing en estado `Draft` vinculado a un Service `Active`.

**Auth:** `listing:create` permission

**Request:**
```json
{
  "profileId": "uuid",
  "serviceId": "uuid",
  "title": "string | null",
  "description": "string | null",
  "placeId": "uuid | null",
  "price": { "amount": 0, "currency": "ARS" },
  "confirmationPolicy": "AutoConfirm | ManualConfirm | RequestOnly",
  "visibility": "Public | Private",
  "channelIds": ["uuid"],
  "slotConfig": {
    "durationMinutes": 60,
    "preBufferMinutes": 0,
    "postBufferMinutes": 0,
    "bookingWindow": {
      "minNoticeHours": 1,
      "maxAdvanceDays": 30
    }
  },
  "capacity": 1,
  "intakeForm": [
    { "label": "string", "type": "text | select | bool", "required": true, "options": ["string"] }
  ],
  "addOns": [],
  "tags": ["string"],
  "media": []
}
```

**Command dispatched:** `CreateListingCommand`
**Event emitted:** `ListingCreated`

**Response:** `201 Created`
```json
{ "id": "uuid", "status": "Draft" }
```

---

## Read

### `GET /listings/:listingId` 🚧

Retorna un Listing por ID. Público si `status=Published` y `visibility=Public`.

**Response:**
```json
{
  "id": "uuid",
  "profileId": "uuid",
  "serviceId": "uuid",
  "type": "ServiceListing",
  "status": "Draft | Published | Unpublished | Archived",
  "title": "string",
  "description": "string",
  "placeId": "uuid | null",
  "price": { "amount": 0, "currency": "ARS" },
  "confirmationPolicy": "AutoConfirm | ManualConfirm | RequestOnly",
  "visibility": "Public | Private",
  "channelIds": ["uuid"],
  "slotConfig": {
    "durationMinutes": 60,
    "preBufferMinutes": 0,
    "postBufferMinutes": 0,
    "bookingWindow": {
      "minNoticeHours": 1,
      "maxAdvanceDays": 30
    }
  },
  "capacity": 1,
  "intakeForm": [
    { "label": "string", "type": "text | select | bool", "required": true, "options": ["string"] }
  ],
  "addOns": [],
  "tags": ["string"],
  "media": [
    { "url": "string", "type": "image | video", "order": 0 }
  ],
  "shareableUrl": "string | null",
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

---

### `GET /listings` 🚧

Lista Listings del profile autenticado (owner view). Para discovery pública ver `search.md` (futuro).

**Query params:** `?profileId=uuid&status=Draft|Published|Unpublished|Archived&page=1&pageSize=20`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "serviceId": "uuid",
      "title": "string",
      "status": "Draft | Published | Unpublished | Archived",
      "price": { "amount": 0, "currency": "ARS" },
      "visibility": "Public | Private",
      "capacity": 1,
      "createdAt": "ISO8601"
    }
  ],
  "total": 0,
  "page": 1,
  "pageSize": 20
}
```

---

## Update

### `PATCH /listings/:listingId` 🚧

Actualiza campos de un Listing (Draft, Published o Unpublished). No permitido si Archived.
Cambios en Published no modifican Bookings existentes (usan snapshot).
Si cambia `slotConfig`: Supply reproyecta Slots futuros.

**Auth:** `listing:create` permission

**Request (partial — solo campos a modificar):**
```json
{
  "title": "string",
  "description": "string",
  "placeId": "uuid",
  "price": { "amount": 0, "currency": "ARS" },
  "confirmationPolicy": "AutoConfirm | ManualConfirm | RequestOnly",
  "visibility": "Public | Private",
  "channelIds": ["uuid"],
  "slotConfig": { "...": "..." },
  "capacity": 1,
  "intakeForm": [],
  "addOns": [],
  "tags": [],
  "media": []
}
```

**Command dispatched:** `UpdateListingCommand`
**Event emitted:** `ListingUpdated`

---

## Recurrence

### `PUT /listings/:listingId/weekly-recurrence` 🚧

Configura recurrencia semanal (lunes-domingo con time ranges por día).

**Auth:** `listing:create` permission

**Request:**
```json
{
  "days": [
    {
      "day": "Monday",
      "timeRanges": [
        { "start": "09:00", "end": "13:00" },
        { "start": "16:00", "end": "20:00" }
      ]
    }
  ]
}
```

**Command dispatched:** `UpdateWeeklyRecurrenceCommand`

---

### `PUT /listings/:listingId/weekly-nweeks-recurrence` 🚧

Configura recurrencia cada N semanas.

**Request:**
```json
{
  "days": [ "..." ],
  "intervalInWeeks": 2
}
```

**Command dispatched:** `UpdateWeeklyByNWeeksRecurrenceCommand`

---

### `PUT /listings/:listingId/weekly-recurrence-exceptions` 🚧

Configura fechas de excepción (feriados, vacaciones) para recurrencia semanal.

**Request:**
```json
{
  "exceptions": [
    { "date": "2026-04-01", "reason": "Feriado" }
  ]
}
```

**Command dispatched:** `UpdateWeeklyRecurrenceExceptionsCommand`

---

### `PUT /listings/:listingId/specific-recurrence` 🚧

Configura fechas específicas en lugar de recurrencia semanal.

**Request:**
```json
{
  "dates": [
    { "date": "2026-04-15", "timeRanges": [ { "start": "10:00", "end": "18:00" } ] }
  ]
}
```

**Command dispatched:** `UpdateSpecificDatesRecurrenceCommand`

---

## Media

### `POST /listings/:listingId/media` 🚧 PENDING

Upload de imagen/video al Listing. Máximo 5 media items.

**Auth:** `listing:create` permission
**Content-Type:** `multipart/form-data`

**Request:** Form field `file` (image/video)

**Command dispatched:** `AddListingMediaCommand`

---

### `DELETE /listings/:listingId/media/:mediaId` 🚧 PENDING

Elimina un media item del Listing.

**Command dispatched:** `RemoveListingMediaCommand`

---

### `PUT /listings/:listingId/media/:mediaId/cover` 🚧 PENDING

Establece un media item como cover image.

**Command dispatched:** `SetListingCoverMediaCommand`

---

### `PUT /listings/:listingId/media/reorder` 🚧 PENDING

Reordena los media items.

**Request:**
```json
{
  "order": [
    { "mediaId": "uuid", "order": 1 },
    { "mediaId": "uuid", "order": 2 }
  ]
}
```

**Command dispatched:** `ReorderListingMediaCommand`

---

## Lifecycle

### `POST /listings/:listingId/publish` 🚧

Transiciona `Draft | Unpublished` → `Published`.
Valida todos los campos requeridos antes de publicar.

**Auth:** `listing:publish` permission

**Errors:**
- `422` con lista de campos faltantes/inválidos si validación falla
- `422` si el Service referenciado no está `Active`

**Command dispatched:** `PublishListingCommand`
**Events emitted:** `ListingPublished` → Supply proyecta Slots, Discovery indexa

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Published", "shareableUrl": "string" }
```

---

### `POST /listings/:listingId/unpublish` 🚧

Transiciona `Published` → `Unpublished`.
Supply invalida Slots futuros. Discovery deindexea.

**Auth:** `listing:publish` permission

**Command dispatched:** `UnpublishListingCommand`
**Event emitted:** `ListingUnpublished`

---

### `POST /listings/:listingId/archive` 🚧

Archiva un Listing. **Irreversible.**
Supply elimina Slots futuros. Bookings existentes se preservan.

**Auth:** `listing:publish` permission

**Command dispatched:** `ArchiveListingCommand`
**Event emitted:** `ListingArchived`

**Errors:**
- `422` si el Listing ya está `Archived`
