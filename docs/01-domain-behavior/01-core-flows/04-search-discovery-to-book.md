---
title: Flow 04 — Search, Discovery y Listing Detail
description: >
  Un usuario busca con intención explícita (Search) o explora sin query (Discovery).
  Ambas superficies llevan al Listing Detail desde donde se ejecuta el booking (Flow 02).
  El índice de búsqueda se mantiene de forma asíncrona mediante proyecciones de eventos.
  El módulo Search es transversal: cubre Listings, Profiles y Places.
status: draft
version: 2
---

# Flow 04 — Search, Discovery y Listing Detail

## 1. Resumen
- **Goal:** que cualquier usuario autenticado pueda encontrar Listings, Profiles y Places
  mediante búsqueda intencional (texto + filtros) o exploración contextual (carruseles,
  categorías, mapa), y llegue al Listing Detail para iniciar una reserva (Flow 02).
- **Actores:**
  - **Primary:** cualquier usuario autenticado.
  - **Secondary:** Sistema VyteMerge — mantiene el índice, aplica eligibilidad/privacidad, rankea resultados.
- **Surfaces:** `vt-search-mfe` (nuevo MFE); barra de búsqueda embebida en el shell global.

---

## 2. Domain Context

### Search vs Discovery
| Modo | Trigger | Superficie | Señal principal |
|------|---------|------------|----------------|
| **Search** | Usuario escribe una query | `/search?q=...` | Intención explícita (texto + filtros) |
| **Discovery** | Usuario explora sin query | `/discover` | Contexto (ubicación, tiempo, follows) |
| **Near Me** | Usuario abre el mapa | `/discover/near-me` | Geolocalización |
| **Category** | Usuario selecciona categoría | `/discover/category/:slug` | Tag/faceta |

### Módulo Search (transversal)
El módulo `Search` es el único responsable del índice. No posee las entidades; las indexa
consultando eventos de otros módulos:

```
ListingPublished / ListingUpdated / ListingArchived
ProfileUpdated
PlaceCreated / PlaceArchived
PostPublished / PostArchived (fase 2)
  → SearchIndexProjection actualiza tsvector en tabla search_index
```

### Eligibilidad y privacidad (Access enforcement)
El índice contiene todo, pero las queries siempre aplican filtros de Access:
- Bloqueos: si A bloquea a B o viceversa, ningún contenido de B aparece para A.
- Perfiles privados: aparecen con metadata mínima + CTA "Solicitar seguir".
- Listings UNLISTED o Archived: excluidos de resultados.
- Listings Partner-only: visibles solo para viewers con Agreement activo.

### PostgreSQL FTS (v1)
El índice usa `tsvector` + `tsquery` sobre la tabla `search_index`.
Diseñado para ser swap-able: el módulo `Search` encapsula toda la lógica de indexación
y puede migrarse a Typesense, Meilisearch o un modelo de IA en fases posteriores
sin cambiar los contratos del resto del sistema.

---

## 3. Preconditions
- Usuario autenticado.
- Al menos un Listing Published en la plataforma para obtener resultados no vacíos.
- Para Near Me con GPS: usuario otorgó permiso de geolocalización.

---

## 4. Trigger

- Usuario pulsa la barra de búsqueda global (shell) y escribe una query.
- Usuario navega a `/discover` desde el home o la navegación principal.
- Usuario pulsa "Cerca mío" desde el home o el mapa.
- Usuario pulsa una categoría/tag desde el Discovery home.

---

## 5. Main Flow

### Capacidad A — Búsqueda con query (Search)

#### A.1 — Autocomplete
1. Usuario comienza a escribir en la barra de búsqueda del shell.
2. Tras 2+ caracteres, sistema consulta `GetAutocompleteSuggestions(prefix, viewerId)`.
3. Sistema retorna sugerencias en ≤ 150ms:
   - Tags/categorías que coinciden con el prefijo.
   - Nombres de Listings que coinciden.
   - Nombres de Profiles que coinciden.
   - Búsquedas recientes del usuario (personales, solo client-side).
4. Usuario selecciona una sugerencia o escribe la query completa y hace submit.

#### A.2 — Resultados de búsqueda
5. Sistema navega a `/search?q=<query>&<filtros>`.
6. Sistema ejecuta `SearchListings(query, filters, viewerId, location?)`.
7. Sistema aplica:
   - **FTS rank** sobre `tsvector` (title, description, tags, profile name, place name).
   - **Filtros activos:** categoría, distancia, disponibilidad, precio, tipo.
   - **Access/eligibility:** excluye contenido no visible para el viewer.
8. Sistema rankea resultados con el **algoritmo v1**:
   ```
   score = text_relevance(query, item)
         × distance_boost(item.place, viewer.location)
         × recency_boost(item.publishedAt | item.updatedAt)
         × follow_boost(viewer, item.authorId)
   ```
9. Sistema devuelve resultados en tres secciones:
   - **Listings** (sección principal, paginada).
   - **Profiles** (hasta 5 resultados inline; "Ver todos" abre `/search?q=...&type=profiles`).
   - **Places** (hasta 3 resultados inline; "Ver todos" abre `/search?q=...&type=places`).
10. Usuario aplica o ajusta filtros desde el panel lateral/drawer.
    - Cada cambio de filtro actualiza los resultados sin recargar la página.

#### A.3 — Zero-result recovery
11. Si no hay resultados:
    - Sistema sugiere relajar filtros específicos ("Intentá sin el filtro de precio").
    - Sistema sugiere términos alternativos (tags relacionados al texto de la query).
    - Sistema muestra carruseles de fallback: "Popular en tu zona" / "Esta semana".

---

### Capacidad B — Discovery (sin query)

12. Usuario navega a `/discover`.
13. Sistema muestra el Discovery Home con carruseles contextuales:

| Carrusel | Señal | Contenido |
|---------|-------|---------|
| **Cerca tuyo** | Geolocalización del usuario | Listings con Place dentro del radio configurado |
| **Esta semana** | Disponibilidad próxima (próximos 7 días) | Listings con slots disponibles |
| **Popular** | Reactions + bookings recientes (global) | Listings con mayor engagement reciente |
| **Nuevos** | `publishedAt` reciente | Listings publicados en los últimos 7 días |
| **Porque seguís a X** | Social graph (follows) | Listings de perfiles que el usuario sigue |

14. Cada carrusel incluye su "explicación" ("Cerca tuyo", "Porque seguís a @masajes_maria").
15. Contenido patrocinado (si aplica) aparece etiquetado explícitamente como "Patrocinado".
16. Usuario puede hacer tap/clic en cualquier card → va al Listing Detail.

---

### Capacidad C — Near Me (mapa)

17. Usuario navega a `/discover/near-me`.
18. Sistema solicita permiso de geolocalización (GPS del dispositivo).
    - Si el usuario acepta → usa coordenadas GPS.
    - Si rechaza → muestra campo para ingresar dirección manualmente.
    - Fallback: última ubicación conocida (inferida del último Booking o Place del Timeline).
19. Sistema muestra mapa (Google Maps) con pins de Listings en el área visible.
20. Panel lateral muestra la lista de los mismos Listings ordenados por distancia.
21. Al hacer zoom/pan en el mapa, los resultados se actualizan automáticamente
    según el bounding box visible.
22. Al pulsar un pin o un item de la lista → abre el preview del Listing (bottom sheet).
23. Desde el preview → CTA "Ver Listing" → va al Listing Detail.

---

### Capacidad D — Category / Tag landing

24. Usuario pulsa una categoría desde el Discovery Home o desde un resultado.
25. Sistema navega a `/discover/category/:slug`.
26. Sistema muestra Listings filtrados por ese tag/categoría.
27. Ordenamiento: por relevancia contextual (distancia + disponibilidad + engagement).
28. Usuario puede agregar filtros adicionales desde el panel.

---

### Capacidad E — Listing Detail (puente con Flow 02)

29. Usuario pulsa cualquier Listing card desde Search, Discovery, Near Me o Category.
30. Sistema navega a `/listings/:id` (manejado por `vt-listings-mfe`).
31. Listing Detail muestra:
    - Media (fotos/video).
    - Descripción, precio, Place, proveedor.
    - Slots disponibles proyectados (próximos N días).
    - Reactions count + CTA para reaccionar.
    - CTA principal: **"Reservar"** → inicia Flow 02.
32. Si el viewer no tiene permiso (Access) → muestra mensaje de acceso restringido.
33. Si el Listing fue archivado después de que el usuario llegó al link → muestra "no disponible".

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| Query con typo leve (ej: "mazaje") | FTS con `pg_trgm` (trigram similarity) tolera typos menores; NEEDS-CLARIFICATION: threshold |
| Búsqueda de Listing cuyo proveedor bloqueó al viewer | Listing excluido de resultados (Access enforcement) |
| GPS no disponible y sin dirección manual | Discovery Near Me usa última ubicación conocida; si ninguna: pide al usuario ingresar una |
| Listing publicado hace 5 minutos — ¿ya aparece? | Indexación asíncrona: demora hasta 30s; dentro de la misma sesión puede no aparecer aún |
| Carrusel "Cerca tuyo" sin geolocalización | No se muestra el carrusel; se reemplaza por "Popular esta semana" |
| Carrusel "Porque seguís a X" sin follows | No se muestra ese carrusel |
| Filtro de disponibilidad "hoy" pero no hay slots | Resultados vacíos; zero-result recovery activo |
| Deep-link a `/search?q=masajes&lat=...&lng=...` | Carga directamente con query y ubicación pre-definidas; útil para compartir búsquedas |
| Usuario sin perfil con Place o Timeline | Near Me solo puede usar GPS o dirección manual |
| Listing con `visibility=Private` aparece indexado | Access enforcement lo excluye en runtime; nunca llega al viewer |

---

## 7. Data Model (v1)

```
-- Módulo: Search (schema: search)

SearchIndex {
  id             UUID
  entityType     listing | profile | place
  entityId       UUID
  profileId      UUID          -- owner del objeto (para follow_boost)
  searchVector   tsvector      -- índice FTS (title + description + tags + profile name)
  title          string        -- para mostrar en resultados
  subtitle       string?       -- precio, categoría, etc.
  imageUrl       string?
  placeId        UUID?         -- para distance_boost y Near Me
  lat            decimal?
  lng            decimal?
  tags           string[]
  publishedAt    DateTime
  updatedAt      DateTime
  eligibility    jsonb         -- { visibility, partnerOnly } -- channels diferidos post-v1 (ADR-0007)
  -- GIN INDEX on searchVector
  -- GIN INDEX on tags
  -- INDEX (lat, lng)  -- para búsquedas geoespaciales
}

AutocompleteEntry {
  id        UUID
  text      string            -- texto sugerido
  type      tag | listing | profile
  entityId  UUID?
  weight    float             -- popularidad del término
  updatedAt DateTime
  -- INDEX (text text_pattern_ops)  -- para prefix matching
}

-- Read model: location del viewer (para near me y distance_boost)
ViewerLocation {
  viewerId    UUID
  source      gps | manual | inferred
  lat         decimal
  lng         decimal
  updatedAt   DateTime
}
```

---

## 8. Algoritmo de Ranking v1

```
score(item, viewer, query) =
  text_relevance(query, item.searchVector)     -- ts_rank de PostgreSQL
  × distance_boost(item.location, viewer.location)
  × recency_boost(item.publishedAt, item.updatedAt)
  × follow_boost(viewer.followingIds, item.profileId)

-- distance_boost:
  distancia ≤ 2km   → 2.0
  distancia ≤ 5km   → 1.5
  distancia ≤ 20km  → 1.2
  distancia ≤ 50km  → 1.0
  distancia > 50km  → 0.7
  sin Place         → 1.0

-- recency_boost:
  publicado/actualizado < 7 días  → 1.3
  < 30 días                       → 1.1
  > 30 días                       → 1.0

-- follow_boost:
  viewer sigue al profileId       → 1.5
  no sigue                        → 1.0

-- Señales v2 (no implementar en v1):
  popularity_boost  (bookings_count, reactions_count)
  completeness_boost (tiene media, descripción completa)
  rating_boost      (rating promedio)
```

---

## 9. Commands / Queries

| Query | Módulo | Descripción |
|-------|--------|-------------|
| `SearchListings(query, filters, viewerId, location)` | Search | Resultados paginados con score |
| `SearchProfiles(query, viewerId)` | Search | Perfiles que coinciden con la query |
| `SearchPlaces(query, viewerId, location)` | Search | Places cercanos o que coinciden |
| `GetAutocompleteSuggestions(prefix, viewerId)` | Search | Sugerencias para autocomplete |
| `GetDiscoveryHome(viewerId, location)` | Search | Payload de todos los carruseles |
| `GetNearMeListings(viewerId, bbox)` | Search | Listings dentro del bounding box del mapa |
| `GetCategoryListings(slug, viewerId, filters, location)` | Search | Listings filtrados por categoría |

**Proyecciones (comandos internos del módulo Search):**

| Proyección | Trigger | Acción |
|-----------|---------|--------|
| `IndexListing` | `ListingPublished` / `ListingUpdated` | Inserta/actualiza `SearchIndex` |
| `DeindexListing` | `ListingArchived` | Elimina de `SearchIndex` |
| `IndexProfile` | `ProfileUpdated` | Actualiza `SearchIndex` del Profile |
| `IndexPlace` | `PlaceCreated` / `PlaceArchived` | Inserta o elimina `SearchIndex` del Place |
| `UpdateViewerLocation` | Evento de geolocalización del cliente | Actualiza `ViewerLocation` |

---

## 10. Invariants

1. Ningún objeto con Access restringido llega al viewer sin pasar por el filtro de eligibilidad.
2. Objetos UNLISTED nunca aparecen en resultados de Search ni en carruseles de Discovery.
3. El índice de búsqueda es eventual: puede demorar hasta 30s en reflejar cambios recientes.
4. La lógica de indexación y ranking está encapsulada en el módulo `Search`; otros módulos no conocen el índice.
5. El módulo `Search` no posee Listings, Profiles ni Places; los indexa por ID y metadata.
6. `follow_boost` se calcula en runtime contra el Social Graph; no se denormaliza en el índice.
7. Resultados patrocinados siempre se etiquetan explícitamente y respetan Access.
8. La geolocalización del usuario nunca se almacena permanentemente sin consentimiento explícito.

---

## 11. Outputs

- Lista paginada de resultados relevantes (Listings + Profiles + Places).
- Carruseles de Discovery contextuales para el usuario.
- Mapa con pins de Listings en el área visible.
- Sugerencias de autocomplete en tiempo real.
- Listing Detail accesible → entrada a Flow 02 (Booking).

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo nuevo:** `Search`

```
src/Modules/Search/
├── Search.Application/
│   ├── Queries/
│   │   ├── SearchListings/
│   │   ├── SearchProfiles/
│   │   ├── SearchPlaces/
│   │   ├── GetAutocompleteSuggestions/
│   │   ├── GetDiscoveryHome/
│   │   ├── GetNearMeListings/
│   │   └── GetCategoryListings/
│   └── Projections/
│       ├── IndexListingHandler/         -- consume ListingPublished, ListingUpdated
│       ├── DeindexListingHandler/       -- consume ListingArchived
│       ├── IndexProfileHandler/         -- consume ProfileUpdated
│       └── IndexPlaceHandler/           -- consume PlaceCreated, PlaceArchived
├── Search.Domain/
│   ├── SearchIndex.cs
│   └── AutocompleteEntry.cs
├── Search.Infrastructure/
│   ├── SearchDbContext.cs               -- schema: search
│   └── PostgresFtsSearchService.cs      -- encapsula tsvector/tsquery; swap-able
├── Search.IntegrationEvents/            -- ninguno emitido; solo consumidos
└── Search.Presentation/
    ├── SearchListingsEndpoint.cs
    ├── SearchProfilesEndpoint.cs
    ├── AutocompleteEndpoint.cs
    ├── DiscoveryHomeEndpoint.cs
    ├── NearMeEndpoint.cs
    └── CategoryListingsEndpoint.cs
```

**Endpoints:**
```
-- Search
GET  /search?q=&type=&tags=&lat=&lng=&radiusKm=&from=&to=&priceMin=&priceMax=
GET  /search/autocomplete?prefix=&limit=
GET  /search/profiles?q=
GET  /search/places?q=&lat=&lng=

-- Discovery
GET  /discover                              → GetDiscoveryHome (todos los carruseles)
GET  /discover/near-me?lat=&lng=&bbox=      → GetNearMeListings
GET  /discover/category/:slug?<filters>     → GetCategoryListings

-- Viewer location
PUT  /viewer/location                       → UpdateViewerLocation { source, lat, lng }
```

**Filtros soportados en `/search`:**
```
q          string     query de texto
type       listing | profile | place   (default: all)
tags       string[]   filtro por categoría/tag
lat, lng   decimal    coordenadas del viewer
radiusKm   int        radio de búsqueda (default: 10km)
from       date       disponibilidad desde
to         date       disponibilidad hasta
priceMin   decimal    precio mínimo
priceMax   decimal    precio máximo
serviceType service | product
cursor     string     paginación
limit      int        default 20
```

### Frontend — `vt-search-mfe` (nuevo MFE)

```
/search                     → página de resultados (con query bar, filtros, secciones)
/discover                   → discovery home (carruseles)
/discover/near-me           → mapa Google Maps + lista
/discover/category/:slug    → resultados por categoría
```

**Componente remoto embebido en shell:**
```
SearchBarComponent (remote de vt-search-mfe)
  → embebido en el header global del shell
  → al submit: navega al MFE con /search?q=...
  → muestra dropdown de autocomplete inline
```

**UI States:**

**`/search`:**
- Panel de filtros (lateral en desktop, drawer en mobile).
- Tres secciones colapsables: Listings / Perfiles / Lugares.
- Estado vacío con zero-result recovery (sugerencias + carruseles fallback).
- Toggle lista/mapa en sección Listings.

**`/discover`:**
- Carruseles horizontales con scroll: Cerca tuyo / Esta semana / Popular / Nuevos / Porque seguís a X.
- Cada carrusel tiene etiqueta de "por qué" + CTA "Ver todos".
- Carruseles condicionales: "Cerca tuyo" solo si hay geolocalización; "Porque seguís" solo si hay follows.

**`/discover/near-me`:**
- Google Maps con pins de Listings (color por categoría).
- Panel lateral con lista ordenada por distancia.
- Bottom sheet al pulsar un pin: preview del Listing + CTA "Ver Listing".
- Re-fetch automático al mover el mapa (debounce 500ms).

---

## 13. Acceptance Criteria

- [ ] Al escribir 2+ caracteres en la search bar, se muestran sugerencias de autocomplete en ≤ 150ms.
- [ ] Al buscar por texto, los resultados aparecen divididos en secciones: Listings, Perfiles, Lugares.
- [ ] Los filtros de categoría, distancia, disponibilidad, precio y tipo funcionan correctamente.
- [ ] Un Listing archivado o con Access restringido nunca aparece en resultados.
- [ ] Un Listing UNLISTED nunca aparece en resultados ni en carruseles de Discovery.
- [ ] Perfiles privados aparecen con metadata mínima y CTA "Solicitar seguir".
- [ ] Resultados de perfiles bloqueados por el viewer están ausentes.
- [ ] Al aplicar filtros, los resultados se actualizan sin recargar la página.
- [ ] Con zero resultados, el sistema muestra sugerencias de relajar filtros y carruseles de fallback.
- [ ] `/discover` muestra todos los carruseles contextuales.
- [ ] El carrusel "Cerca tuyo" solo aparece si hay geolocalización disponible.
- [ ] El carrusel "Porque seguís a X" solo aparece si el usuario sigue al menos un perfil con Listings.
- [ ] `/discover/near-me` muestra el mapa con pins de Listings en el área visible.
- [ ] Al mover el mapa, los pins se actualizan según el bounding box visible.
- [ ] Al pulsar un pin, se muestra el preview del Listing con CTA "Ver Listing".
- [ ] Al publicar un Listing, aparece en resultados de búsqueda en menos de 30 segundos.
- [ ] Al archivar un Listing, desaparece de resultados en menos de 30 segundos.
- [ ] Al hacer clic en un Listing desde cualquier superficie, el usuario llega al Listing Detail.
- [ ] El Listing Detail muestra slots disponibles y CTA "Reservar" que inicia Flow 02.
- [ ] Deep-links con query y coordenadas pre-cargadas funcionan correctamente.

---

## 14. NEEDS-CLARIFICATION

- **Typo tolerance:** ¿usamos `pg_trgm` (trigram similarity) para tolerar typos, o en v1 solo FTS exacto con stems? Trigram agrega complejidad al índice pero mejora notablemente la UX.
- **Indexación inicial:** al lanzar el módulo Search en un sistema con Listings existentes, ¿cómo se hace el backfill del índice? Recomiendo un comando `ReindexAll` de ejecución única.
- **Geolocalización — persistencia:** ¿almacenamos `ViewerLocation` en la DB (para usarlo en requests siguientes sin re-pedir GPS), o es solo session-side en el frontend? Tiene implicancias de privacidad y GDPR.
- **Radio de búsqueda default:** ¿cuántos km es el radio default para "Cerca tuyo" si el usuario no lo ajusta? (propongo 10km, ajustable por el usuario).
- **Carrusel "Porque seguís a X":** ¿el carrusel agrupa Listings de todos los follows en un solo carrusel, o hay un carrusel por perfil seguido?
- **Sponsored placement v1:** ¿hay contenido patrocinado en v1? Si sí, ¿cuál es el modelo de ranking para mezclarlo con resultados orgánicos?
- **Tags/categorías — taxonomía:** el módulo `Tags` está aún como stub. Search necesita una taxonomía definida para el filtro de categorías y los carruseles por categoría. ¿Quién define el árbol de categorías (admin, provider, o ambos)?
- **Mapa sin permisos GPS en iOS/Android:** iOS especialmente limita el acceso GPS en web. ¿Tenemos fallback de geolocalización por IP como último recurso?
