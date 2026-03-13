# API Contracts — Places

Base path: `/api/v1`
Module: `Vt.Modules.Places`
Schema: `places`

> Cubre Places propios, Community Places, jerarquías y búsqueda Google Maps.
> Un Place provee el contexto "dónde" a Listings y Bookings.
> Ver flow `09-gestionar-place.md` y spec pack `DRAFT-manage-place` para reglas completas.
>
> **Tipos:** `Physical | Online | Community`
> **Roles:** `Venue` (contexto de ubicación) | `Resource` (recurso escaso; auto-crea Timeline)
> **Jerarquía:** hasta 3 niveles (Place → SubPlace → SubSubPlace)

---

## Create

### `POST /places` 🚧

Crea un Place propio. Si `role=Resource`, el sistema auto-crea un resource Timeline.

**Auth:** usuario autenticado con Profile activo.

**Request:**
```json
{
  "type": "Physical | Online | Community",
  "role": "Venue | Resource",
  "name": "string",
  "description": "string | null",
  "visibility": "Public | Approximate | Hidden",
  "parentPlaceId": "uuid | null",
  "address": {
    "street": "string",
    "number": "string | null",
    "city": "string",
    "state": "string | null",
    "country": "string",
    "postalCode": "string | null",
    "lat": 0.0,
    "lng": 0.0
  },
  "onlineConfig": {
    "platform": "Zoom | GoogleMeet | MicrosoftTeams | CustomUrl"
  },
  "timezone": "America/Buenos_Aires"
}
```

> `address` requerido si `type=Physical`. `onlineConfig` requerido si `type=Online`. `timezone` requerido.

**Command dispatched:** `CreatePlaceCommand`
**Events emitted:** `PlaceCreated` → si `role=Resource`: Timeline module crea resource Timeline

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "type": "Physical | Online | Community",
  "role": "Venue | Resource",
  "status": "Active",
  "resourceTimelineId": "uuid | null"
}
```

---

### `POST /places/import` 🚧

Importa un Place desde Google Maps usando `googlePlaceId`.
Si el `googlePlaceId` ya existe en VyteMerge, **redirige al existente** (no crea duplicado).

**Request:**
```json
{
  "googlePlaceId": "string",
  "role": "Venue | Resource",
  "visibility": "Public | Approximate | Hidden"
}
```

**Command dispatched:** `ImportPlaceFromGoogleMapsCommand`
**Events emitted:** `PlaceCreated` (si nuevo) | redirige al existente (si duplicado)

**Response (nuevo):** `201 Created` con el Place creado.

**Response (duplicado):** `303 See Other`
```json
{ "existingPlaceId": "uuid" }
```

---

### `POST /places/:placeId/sub-places` 🚧

Crea un sub-place hijo del Place especificado. La jerarquía no puede superar 3 niveles.

**Request:** mismo shape que `POST /places` (sin `parentPlaceId` — se toma del path).

**Command dispatched:** `AddSubPlaceCommand`

**Errors:**
- `422` si la jerarquía resultante supera 3 niveles.

---

## Read

### `GET /places` 🚧

Lista los Places del usuario autenticado (propios + referenciados en Listings).

**Query params:** `?type=Physical|Online|Community&role=Venue|Resource&status=Active|Archived&page=1&pageSize=20`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "name": "string",
      "type": "Physical | Online | Community",
      "role": "Venue | Resource",
      "visibility": "Public | Approximate | Hidden",
      "status": "Active | Archived",
      "timezone": "string",
      "address": { "city": "string", "country": "string" },
      "parentPlaceId": "uuid | null",
      "resourceTimelineId": "uuid | null",
      "createdAt": "ISO8601"
    }
  ],
  "total": 0,
  "page": 1,
  "pageSize": 20
}
```

---

### `GET /places/:placeId` 🚧

Retorna el detalle completo de un Place. Público si `visibility=Public`.

**Response:**
```json
{
  "id": "uuid",
  "ownerProfileId": "uuid",
  "creatorProfileId": "uuid",
  "parentPlaceId": "uuid | null",
  "type": "Physical | Online | Community",
  "role": "Venue | Resource",
  "name": "string",
  "description": "string | null",
  "visibility": "Public | Approximate | Hidden",
  "status": "Active | Archived",
  "timezone": "string",
  "googlePlaceId": "string | null",
  "resourceTimelineId": "uuid | null",
  "address": {
    "street": "string",
    "number": "string | null",
    "city": "string",
    "state": "string | null",
    "country": "string",
    "postalCode": "string | null",
    "lat": 0.0,
    "lng": 0.0
  },
  "onlineConfig": {
    "platform": "Zoom | GoogleMeet | MicrosoftTeams | CustomUrl"
  },
  "subPlaces": [
    { "id": "uuid", "name": "string", "role": "Venue | Resource", "visibility": "string" }
  ],
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

> `address` solo presente si `type=Physical`. `onlineConfig` solo si `type=Online`.
> Coordenadas solo si `visibility != Hidden` (para Approximate: sin coordenadas exactas).

---

### `GET /places/search` 🚧

Busca Places públicos y Community Places por nombre o ubicación.
Usado desde el formulario de Listing para referenciar un Place.

**Query params:** `?q=string&type=Physical|Community&lat=0.0&lng=0.0&radiusKm=10&page=1&pageSize=20`

**Response:** mismo shape que `GET /places` con campo adicional `distanceKm: number | null`.

---

### `GET /places/google` 🚧

Proxy hacia Google Places API para autocompletado de direcciones.
Devuelve sugerencias de Google sin crear ningún registro.

**Query params:** `?q=string`

**Response:**
```json
{
  "suggestions": [
    {
      "googlePlaceId": "string",
      "name": "string",
      "address": "string",
      "city": "string",
      "country": "string"
    }
  ]
}
```

---

## Update

### `PATCH /places/:placeId` 🚧

Actualiza campos de un Place activo.

**Auth:** Owner del Place o Delegado con permiso `ManagePlaces`.

**Request (partial):**
```json
{
  "name": "string",
  "description": "string | null",
  "visibility": "Public | Approximate | Hidden",
  "address": { "...": "..." },
  "onlineConfig": { "platform": "Zoom | GoogleMeet | MicrosoftTeams | CustomUrl" }
}
```

**Command dispatched:** `UpdatePlaceCommand`
**Event emitted:** `PlaceUpdated`

**Errors:**
- `403` si el caller no es el Owner ni tiene `ManagePlaces`.
- `422` si el Place está `Archived`.
- `422` si es Community Place y el caller no es el `creatorProfileId`.

---

## Lifecycle

### `POST /places/:placeId/archive` 🚧

Archiva un Place. Irreversible.

**Auth:** Owner o Delegado con `ManagePlaces`.

**Validations (pre-archive):**
- Si `role=Resource`: no puede tener Bookings futuros en su resource Timeline.
- Si tiene Listings Published: requiere reasignación previa de esos Listings.

**Command dispatched:** `ArchivePlaceCommand`
**Event emitted:** `PlaceArchived` → Listing module marca Listings afectados como degradados.

**Errors:**
- `422` con `{ "blockingBookings": ["uuid"], "affectedListings": ["uuid"] }` si hay bloqueantes.

---
