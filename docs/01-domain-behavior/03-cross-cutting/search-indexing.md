---
title: Search Indexing & Discovery Surfaces
description: Qué se indexa, cuándo se actualiza el índice y en qué surfaces de discovery se usa.
---

# Search Indexing & Discovery Surfaces

## Qué se indexa

| Tipo | Condición |
|---|---|
| **Listing** | Solo `Published` + eligibilidad pública (visibility ≠ UNLISTED/PRIVATE, no archivado) |
| **Profile** | Todos los perfiles activos; privados se muestran con metadata mínima |
| **Place** | Todos los Places activos asociados a un Profile |
| **Tags** | Viven en `AutocompleteEntry` para autocomplete (prefix matching) |

> Eligibilidad (bloqueos, Partner-only, etc.) se evalúa en **runtime** en cada query, no en la indexación — garantiza reflejo inmediato de cambios de permisos sin re-indexar (NFR-04).

## Cuándo se actualiza el índice

| Evento | Acción en índice |
|---|---|
| `ListingPublished` | Upsert en `search_index` (índice tsvector + geo) |
| `ListingUpdated` | Upsert si cambió título, descripción, tags, place o precio — proyección `IndexListingHandler` |
| `ListingArchived` | DELETE del `search_index` para ese `entityId` |
| `ListingUnpublished` | DELETE del `search_index` para ese `entityId` |
| `ProfileUpdated` | Upsert en `search_index` para `entityType=profile` |
| `PlaceCreated` | Upsert en `search_index` para `entityType=place` |
| `PlaceArchived` | DELETE del `search_index` para ese `entityId` |

**Lag:** el índice se actualiza de forma asíncrona vía proyecciones de eventos. Lag máximo aceptable: ≤ 30 segundos (NFR del módulo Search).

## Surfaces (dónde se usa el índice)

| Surface | Ruta | Query usada |
|---|---|---|
| **Barra de búsqueda global** | shell (header) → `SearchBarComponent` | `GetAutocompleteSuggestions` (prefix, ≤ 150ms) |
| **Resultados de búsqueda** | `/search?q=...` | `SearchListings` + `SearchProfiles` + `SearchPlaces` |
| **Discovery Home** | `/discover` | `GetDiscoveryHome` (5 carruseles contextuales) |
| **Near Me** | `/discover/near-me` | `GetNearMeListings` (bounding box geoespacial) |
| **Category landing** | `/discover/category/:slug` | `GetCategoryListings` |
| **Zero-result fallback** | `/search` (sin resultados) | Carruseles Popular + Esta semana desde `GetDiscoveryHome` |

## Módulo propietario

El índice vive en el módulo `Search` (schema: `search`, entidades: `SearchIndex`, `AutocompleteEntry`, `ViewerLocation`).
El motor v1 es PostgreSQL FTS (tsvector/tsquery). La interfaz `ISearchService` permite swap por Typesense / Meilisearch / AI sin cambiar handlers ni endpoints.

## Links
- Flow 04 — Search & Discovery: `01-domain-behavior/01-core-flows/04-search-discovery-to-book.md`
- Spec-pack: `specs/DRAFT-search-discovery/`
- Entities indexed: `00-core-domain/04-bounded-contexts/04-social-discovery/04.search_discovery/02.entities-indexed.md`
- Search BC: `00-core-domain/04-bounded-contexts/04-social-discovery/04.search_discovery/03.search.md`
