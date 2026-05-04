# API Contracts — Listing

Base path: `/api/v1`
Module: `Vt.Modules.Listing`
Schema: `listing`

> Un Listing expone un Service comercialmente: precio efectivo, slotConfig, visibility y add-ons.
> **1 Service → N Listings.** Discovery usa Listings. Booking usa el Service (snapshot).
> Channels (distribution layer) fueron diferidos post-v1 — ver [ADR-0007](../../../private/decisions/ADR-0007-channels-deferred-from-v1.md).
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

Lista Listings del profile autenticado (owner view). Para el viewer público ver `GET /profiles/:profileId/listings` más abajo. Para search global ver `search.md` (futuro).

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

### `GET /profiles/:profileId/listings`

Public viewer surface. Devuelve los Listings del profile filtrados a `Status=Published` y `Visibility=Public` server-side (nunca expone Draft/Archived/Private). Pensado para que un visitante navegue el catálogo de un centro médico, profesional o estudio.

**Query params:**
- `tag` *(opcional)* — filtra por tag exacta (ej: `cardiologia`).
- `page`, `pageSize` *(opcionales)* — defaults `1` / `20`, max `100`.

**Auth:** requiere usuario autenticado (mismo permiso que `GET /listings`). El filtro a Published+Public se aplica en el handler — el endpoint **no puede** devolver listings privados aunque el caller manipule la query.

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "profileId": "uuid",
      "title": "string",
      "description": "string",
      "currency": "ARS",
      "tags": ["cardiologia", "consultorio"],
      "options": [
        {
          "id": "uuid",
          "name": "Online",
          "method": "GoogleCall | OnOwnerLocation | OnAttendeeLocation | PhoneCall | None",
          "price": 5000.0,
          "isDefault": true
        }
      ],
      "coverMediaUrl": "https://cdn.example.com/cover.jpg"
    }
  ],
  "totalCount": 1,
  "page": 1,
  "pageSize": 20
}
```

**Slots NO se incluyen.** El cliente pide la disponibilidad por listing+option vía `GET /events/availability` cuando el visitante elige un listing concreto. Esto evita N range queries por carga del catálogo.

**Filtro por profesional:** no soportado en v1. Hoy todos los listings de un profile usan el default timeline del profile. El filtrado real por profesional llega cuando se implemente `Listing.PerformerTimelineId` (PR2 del plan de availability federada).

---

### `GET /profiles/:profileId/listings/:listingId`

Public viewer detail. Devuelve **el mismo `PublicListingCard`** que el endpoint de listado, pero para un listing puntual identificado por `listingId` y restringido a un profile específico.

**Auth:** requiere usuario autenticado (mismo permiso `offerings:read` que el listado público).

**Path params:**
- `profileId` — el dueño esperado del listing.
- `listingId` — el listing concreto a consultar.

**Filtros server-side (hardcodeados):**
- `status = Published`
- `visibility = Public`
- `profile_id = profileId` (de la URL)

**Response:** `200 OK` con el shape `PublicListingCard` documentado arriba.

**Errors:**
- `404 NotFound` si el listing es Draft, Private, pertenece a otro profile o no existe. Las cuatro situaciones devuelven el **mismo error** — el endpoint nunca filtra entre "no existe" y "no es público" para no leak la existencia de listings no públicos.

**Por qué un endpoint dedicado y no `GET /listings/:id`:** el endpoint genérico devuelve shape admin (incluye `Status`, `IntakeForm`, `Capacity`, etc.) y no filtra Published+Public, por lo que expone Draft/Private a cualquiera con el id. Este endpoint es el contrato seguro para viewers públicos.

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
