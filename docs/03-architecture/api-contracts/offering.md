# API Contracts — Offering (Catalog & Listings)

Base path: `/api/v1`

---

## Catalog — Services & Products

### `GET /catalog` 🚧
Returns the authenticated provider's Catalog (all Services and Products).

**Query params:** `?profileId=uuid`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "type": "service | product",
      "name": "string",
      "description": "string",
      "category": "string",
      "tags": ["string"],
      "durationMinutes": 0
    }
  ],
  "total": 0
}
```

---

### `POST /catalog/services` 🚧
Create a Service definition.

**Request:**
```json
{
  "profileId": "uuid",
  "name": "string",
  "description": "string",
  "category": "string",
  "tags": ["string"],
  "durationMinutes": 30
}
```

**Command dispatched:** `CreateServiceOrProduct`
**Event emitted:** `ServiceCreated`

---

### `POST /catalog/products` 🚧
Create a Product definition.

**Request:**
```json
{
  "profileId": "uuid",
  "name": "string",
  "description": "string",
  "category": "string",
  "tags": ["string"],
  "basePrice": { "amount": 0, "currency": "ARS" }
}
```

**Command dispatched:** `CreateServiceOrProduct`
**Event emitted:** `ProductCreated`

---

## Listings

### `GET /listings` 🚧
List published Listings. Paginated. Used for discovery.

**Query params:** `?profileId=uuid&tags=string&page=1&pageSize=20`

---

### `GET /listings/:listingId` ✅
Get a single Listing by ID.

**Response:**
```json
{
  "listingId": "uuid",
  "profileId": "uuid",
  "serviceId": "uuid | null",
  "productId": "uuid | null",
  "title": "string",
  "description": "string",
  "price": { "amount": 0, "currency": "ARS" },
  "status": "draft | published | unpublished | archived",
  "placeId": "uuid | null",
  "slotConfig": {
    "durationMinutes": 30,
    "bufferBeforeMinutes": 0,
    "bufferAfterMinutes": 0,
    "bookingWindowDays": 30
  },
  "bookingRules": {
    "confirmationRequired": false,
    "minToConfirm": 1,
    "maxCapacity": 1,
    "cancellationWindowHours": 24
  },
  "visibility": "public | private | partner",
  "createdAt": "ISO8601"
}
```

---

### `POST /listings` 🚧
Create a new Listing.

**Request:**
```json
{
  "profileId": "uuid",
  "serviceId": "uuid",
  "title": "string",
  "description": "string",
  "price": { "amount": 0, "currency": "ARS" },
  "placeId": "uuid | null",
  "slotConfig": {
    "durationMinutes": 30,
    "bufferBeforeMinutes": 0,
    "bufferAfterMinutes": 0,
    "bookingWindowDays": 30
  },
  "bookingRules": {
    "confirmationRequired": false,
    "minToConfirm": 1,
    "maxCapacity": 1,
    "cancellationWindowHours": 24
  },
  "visibility": "public"
}
```

**Command dispatched:** `CreateListing`
**Event emitted:** `ListingCreated`

---

### `POST /listings/:listingId/publish` 🚧
Publish a Listing.

**Command dispatched:** `PublishListing`
**Event emitted:** `ListingPublished`

---

### `POST /listings/:listingId/unpublish` 🚧
Unpublish a Listing.

**Command dispatched:** `UnpublishListing`
**Event emitted:** `ListingUnpublished`

---

### `POST /listings/:listingId/archive` 🚧
Archive a Listing.

**Command dispatched:** `ArchiveListing`
**Event emitted:** `ListingArchived`

---

### `PATCH /listings/:listingId/rules` 🚧
Update scheduling or booking rules (triggers slot re-projection).

**Request:**
```json
{
  "slotConfig": { ... },
  "bookingRules": { ... }
}
```

**Command dispatched:** `UpdateListingRules`
**Event emitted:** `ListingUpdated`

---

### `PATCH /listings/:listingId/visibility` 🚧
Update Listing visibility/channel settings.

**Command dispatched:** `UpdateListingVisibility`
**Event emitted:** `ListingUpdated`
