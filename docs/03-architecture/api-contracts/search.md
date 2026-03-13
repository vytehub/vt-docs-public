# API Contracts — Search & Discovery

Base path: `/api/v1`
Module: `Vt.Modules.Search` *(transversal — índice propio; no cross-module EF)*
Schema: `search`

> Cubre búsqueda de Listings, autocompletado y discovery (carruseles home).
> El índice se construye a partir de eventos: `ListingPublished`, `ListingUnpublished`, `ListingArchived`, `ProfileUpdated`.
> Ver flow `04-search-discovery-to-book.md` y spec pack `DRAFT-search-discovery`.

---

## Search

### `GET /search` 🚧

Búsqueda de Listings publicados. Full-text sobre título, descripción, tags y nombre del Provider.

**Query params:**
```
?q=string
&category=string
&lat=0.0
&lng=0.0
&radiusKm=10
&priceMin=0
&priceMax=0
&currency=ARS
&durationMin=30
&durationMax=120
&availability=YYYY-MM-DD
&page=1
&pageSize=20
```

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "listingId": "uuid",
      "title": "string",
      "description": "string",
      "profileId": "uuid",
      "profileName": "string",
      "profileAvatarUrl": "string | null",
      "price": { "amount": 0, "currency": "ARS" },
      "durationMinutes": 60,
      "category": "string | null",
      "tags": ["string"],
      "placeId": "uuid | null",
      "placeName": "string | null",
      "placeCity": "string | null",
      "placeType": "Physical | Online | Community | null",
      "distanceKm": 0.0,
      "media": [{ "url": "string", "type": "image | video" }],
      "shareableUrl": "string",
      "score": 0.0
    }
  ],
  "total": 0,
  "page": 1,
  "pageSize": 20,
  "appliedFilters": {}
}
```

> `distanceKm` solo presente si se proveyó `lat`/`lng`. `score` es el relevance score interno.

---

### `GET /search/autocomplete` 🚧

Retorna sugerencias rápidas (top 5-10) para el input de búsqueda.
Incluye Listings, categorías y nombres de Provider.

**Query params:** `?q=string&limit=10`

**Response:**
```json
{
  "suggestions": [
    {
      "type": "listing | category | provider",
      "id": "uuid | null",
      "label": "string",
      "sublabel": "string | null",
      "avatarUrl": "string | null"
    }
  ]
}
```

---

## Discovery

### `GET /discover` 🚧

Retorna secciones de Listings curados para la Home. No requiere autenticación.
Contenido: trending, recientes, por categoría, cercanos (si se pasa lat/lng).

**Query params:** `?lat=0.0&lng=0.0`

**Response:**
```json
{
  "sections": [
    {
      "id": "string",
      "title": "string",
      "type": "trending | recent | category | nearby",
      "items": [
        {
          "listingId": "uuid",
          "title": "string",
          "profileName": "string",
          "price": { "amount": 0, "currency": "ARS" },
          "coverImageUrl": "string | null",
          "category": "string | null",
          "distanceKm": 0.0,
          "shareableUrl": "string"
        }
      ]
    }
  ]
}
```

---

### `GET /discover/categories` 🚧

Lista las categorías disponibles con conteo de Listings publicados.

**Response:**
```json
{
  "categories": [
    {
      "id": "string",
      "name": "string",
      "slug": "string",
      "listingCount": 0,
      "coverImageUrl": "string | null"
    }
  ]
}
```

---

### `GET /discover/categories/:slug` 🚧

Retorna Listings de una categoría específica, paginados.
Mismo shape de items que `GET /search` pero filtrado por categoría.

**Query params:** `?page=1&pageSize=20&lat=0.0&lng=0.0`

---

## Search Index (Internal)

> El índice de Search se alimenta a través de integration events. No hay endpoints de escritura directa.

| Event | Acción en índice |
|-------|-----------------|
| `ListingPublished` | Indexar / actualizar |
| `ListingUpdated` | Actualizar |
| `ListingUnpublished` | Desindexar |
| `ListingArchived` | Desindexar |
| `ProfileUpdated` | Actualizar `profileName`, `profileAvatarUrl` en documentos del Provider |

> **v1:** índice implementado con PostgreSQL full-text search (`tsvector`).
> **v2 (futuro):** migración a motor dedicado (Meilisearch / Elasticsearch) si el volumen lo justifica.
