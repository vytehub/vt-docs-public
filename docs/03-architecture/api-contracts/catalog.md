# API Contracts — Catalog (Services & Products)

Base path: `/api/v1`
Module: `Vt.Modules.Catalog`
Schema: `catalog`

> **Scope v1:** Services (reservables con Slots). Products independientes son v2.
> Un Service define la operativa (duración, precio base, buffers).
> Un Listing (ver `listing.md`) expone el Service comercialmente con precio efectivo, canales, slotConfig,
> confirmationPolicy, intakeForm, media y visibility.
>
> **Decisiones v1 (ver Flow 05):** `location`, `confirmationPolicy`, `intakeForm` y `media` pertenecen
> al Listing, no al Service. Service es puramente operativo.

---

## Services

### `GET /services` 🚧

Retorna todos los Services de un Profile.

**Query params:** `?profileId=uuid&status=active|draft|archived&page=1&pageSize=20`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "profileId": "uuid",
      "name": "string",
      "description": "string",
      "status": "Draft | Active | Archived",
      "durationMinutes": 60,
      "basePrice": { "amount": 0, "currency": "ARS" },
      "preBufferMinutes": 0,
      "postBufferMinutes": 0,
      "listingCount": 0,
      "createdAt": "ISO8601"
    }
  ],
  "total": 0,
  "page": 1,
  "pageSize": 20
}
```

---

### `GET /services/:serviceId` 🚧

Retorna el detalle de un Service.

**Response:**
```json
{
  "id": "uuid",
  "profileId": "uuid",
  "name": "string",
  "description": "string",
  "status": "Draft | Active | Archived",
  "durationMinutes": 60,
  "basePrice": { "amount": 0, "currency": "ARS" },
  "preBufferMinutes": 0,
  "postBufferMinutes": 0,
  "listingCount": 0,
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

---

### `POST /services` 🚧

Crea un Service en estado `Draft`.

**Request:**
```json
{
  "profileId": "uuid",
  "name": "string",
  "description": "string | null",
  "durationMinutes": 60,
  "basePrice": { "amount": 0, "currency": "ARS" },
  "preBufferMinutes": 0,
  "postBufferMinutes": 0
}
```

**Command dispatched:** `CreateServiceCommand`
**Event emitted:** `ServiceCreated`

**Response:** `201 Created` con el Service creado (mismo shape que GET /services/:serviceId)

---

### `PATCH /services/:serviceId` 🚧

Actualiza un Service en estado `Draft` o `Active`.
No permitido si status = `Archived`.

**Request (partial — solo campos a modificar):**
```json
{
  "name": "string",
  "description": "string",
  "durationMinutes": 60,
  "basePrice": { "amount": 0, "currency": "ARS" },
  "preBufferMinutes": 0,
  "postBufferMinutes": 0
}
```

**Command dispatched:** `UpdateServiceCommand`
**Event emitted:** `ServiceUpdated`

---

### `POST /services/:serviceId/publish` 🚧

Transiciona el Service de `Draft` a `Active`.
Valida campos requeridos (name, durationMinutes, basePrice ≥ 0).

**Command dispatched:** `PublishServiceCommand`
**Event emitted:** `ServicePublished`

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Active" }
```

---

### `POST /services/:serviceId/archive` 🚧

Archiva un Service. Bloquea la creación de nuevos Listings sobre él.
Irreversible — Listings existentes en Draft quedan bloqueados para publicar.

**Command dispatched:** `ArchiveServiceCommand`
**Event emitted:** `ServiceArchived`

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Archived" }
```

---

## Products

> **Nota v1:** Products son definiciones de items vendibles (no reservables con Slots). CRUD mínimo.
> Checkout y payments son out of scope v1.

### `GET /products` 🚧

**Query params:** `?profileId=uuid&status=active|draft|archived&page=1&pageSize=20`

**Response:** mismo patrón que `GET /services` con campos propios del Product.

---

### `GET /products/:productId` 🚧

**Response:**
```json
{
  "id": "uuid",
  "profileId": "uuid",
  "name": "string",
  "description": "string",
  "status": "Draft | Active | Archived",
  "category": "string | null",
  "tags": ["string"],
  "basePrice": { "amount": 0, "currency": "ARS" },
  "sku": "string | null",
  "media": [],
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

---

### `POST /products` 🚧

**Request:**
```json
{
  "profileId": "uuid",
  "name": "string",
  "description": "string | null",
  "category": "string | null",
  "tags": ["string"],
  "basePrice": { "amount": 0, "currency": "ARS" },
  "sku": "string | null"
}
```

**Command dispatched:** `CreateProductCommand`
**Event emitted:** `ProductCreated`

---

### `PATCH /products/:productId` 🚧
### `POST /products/:productId/publish` 🚧
### `POST /products/:productId/archive` 🚧

Mismo patrón que Services.
