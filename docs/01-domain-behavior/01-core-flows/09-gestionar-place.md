---
title: Flow 09 — Gestionar Place
description: >
  Un usuario crea y gestiona Places (ubicaciones físicas, virtuales o comunitarias) que
  sirven de contexto a Listings y Bookings. Un Place puede ser un Venue (contexto de
  ubicación) o un Resource (recurso escaso con Timeline propio). Soporta jerarquía de
  hasta 3 niveles, importación desde Google Maps y referencias a Community Places compartidos.
status: draft
version: 1
---

# Flow 09 — Gestionar Place

## 1. Resumen
- **Goal:** que un usuario pueda crear, configurar y gestionar Places propios, importar
  ubicaciones desde Google Maps, y referenciar Community Places existentes para usarlos
  en sus Listings.
- **Actores:**
  - **Primary:** cualquier usuario autenticado con Profile activo.
  - **Secondary:** Delegado con Agreement activo que incluya permiso `ManagePlaces`.
- **Surfaces:** `vt-places-mfe` — rutas `/places`, `/places/new`, `/places/:id`, `/places/search`.

---

## 2. Domain Context

### Place como "el dónde"
```
El "qué"        → Service / Product
El "cómo vendo" → Listing
El "cuándo"     → Timeline
El "dónde"      → Place
```

Un Place provee contexto confiable de ubicación, timezone y visibilidad a Listings,
Bookings y Events. No decide disponibilidad ni permisos por sí solo.

### Tipos de Place

| Tipo | Descripción |
|------|-------------|
| **Physical** | Dirección real (consultorio, local, cancha). Tiene coordenadas y timezone derivado de la ubicación. |
| **Online** | Ocurre virtualmente. Tiene plataforma (Zoom, Meet, Teams, URL custom). El join link se genera al confirmar el Booking. |
| **Community** | Lugar público sin ownership exclusivo (plaza, parque, espacio público). Importado de Google Maps o creado como community. Compartido — todos referencian el mismo `placeId`. |

### Roles de Place

| Rol | Descripción | Ejemplo |
|-----|-------------|---------|
| **Venue** | Contexto de ubicación solamente. Múltiples servicios ocurren simultáneamente. | "Estética Las Rosas", "Edificio Centro" |
| **Resource** | El recurso físico escaso en sí mismo. Solo un Booking lo usa a la vez. Auto-crea un **resource Timeline**. | "Gabinete 1", "Cancha Sur", "Sala A" |

### Jerarquía (hasta 3 niveles)
```
Gym SuperGym       [Place, Physical, Venue, Public]         ← nivel 1
  └── Sucursal Norte  [SubPlace, Physical, Venue, Public]   ← nivel 2
        └── Sala A    [SubPlace, Physical, Resource, Hidden] ← nivel 3
        └── Sala B    [SubPlace, Physical, Resource, Hidden]
  └── Sucursal Sur  [SubPlace, Physical, Venue, Approximate] ← nivel 2
```

Cada nivel define su propia visibilidad de forma independiente.

### Google Maps integration y deduplicación
Al importar desde Google Maps, el sistema usa el `googlePlaceId` como clave de
deduplicación. Si el Place ya existe en VyteMerge, redirige al existente en lugar
de crear un duplicado.

### Resource Place y resource Timeline
Un Place con `role=Resource` auto-crea un **resource Timeline** al ser creado. Ese
Timeline puede gestionarse tanto desde `vt-places-mfe` como desde `vt-timeline-mfe`.

---

## 3. Preconditions
- Usuario tiene cuenta activa y Profile activo.
- Si el actor es un Delegado: tiene Agreement activo con permiso `ManagePlaces` sobre
  el Profile del Owner.

---

## 4. Trigger
- Usuario navega a `/places` en `vt-places-mfe`.
- Usuario pulsa "Agregar Place" desde el formulario de creación de Listing (deep-link
  a `/places/new?returnTo=listing-form`).
- Delegado con `ManagePlaces` accede al panel de Places del Owner.

---

## 5. Main Flow

### Capacidad A — Crear Place propio (Physical u Online)

1. Usuario navega a `/places/new`.
2. Sistema muestra formulario con campo de búsqueda (Google Maps) y opción "Crear manualmente".

**Via Google Maps:**

3. Usuario escribe nombre o dirección. Sistema sugiere resultados de Google Places API.
4. Usuario selecciona un resultado.
5. Sistema verifica `googlePlaceId`:
   - **Ya existe en VyteMerge** → redirige al Place existente (no crea duplicado).
   - **No existe** → pre-carga `name`, `address`, `coordinates`, `timezone` desde Google.
6. Usuario revisa y puede editar los datos pre-cargados.
7. Usuario elige `role` (Venue | Resource) y `visibility` (Public | Approximate | Hidden).
8. Usuario guarda → Place creado en estado `Active`.

**Manual:**

3. Usuario completa: `name` (req), `type` (Physical | Online), `role`, `visibility`, campos según tipo.
4. Sistema guarda → Place creado en estado `Active`.

**Si `role = Resource`:**
9. Sistema auto-crea un resource Timeline vinculado al Place (sin intervención adicional del usuario).

### Capacidad B — Crear sub-place (jerarquía)

10. Owner navega al detalle de un Place existente → pulsa "Agregar sub-place".
11. Sistema verifica que la jerarquía no supera 3 niveles.
12. Usuario completa nombre, tipo, rol y visibilidad del sub-place.
13. Sistema crea el sub-place como hijo del Place actual.
14. Si `role = Resource` → auto-crea resource Timeline.

### Capacidad C — Referenciar Community Place

15. Usuario crea un Listing y necesita un Place que no posee.
16. Desde el formulario de Listing (o `/places/search`), busca por nombre o ubicación.
17. Sistema muestra Places públicos y Community Places encontrados.
18. Usuario selecciona uno. Sistema registra la referencia (`placeId` del Community Place)
    sin crear un nuevo Place.

### Capacidad D — Archivar Place

19. Owner navega al detalle del Place → pulsa "Archivar".
20. Sistema verifica:
    - Si es **Resource Place**: ¿tiene Bookings futuros en su resource Timeline?
      - Sí → bloquea el archivado; muestra lista de Bookings pendientes; Owner debe resolver.
    - Si tiene **Listings Published** que lo referencian → muestra advertencia; Owner debe
      reasignar un Place alternativo en esos Listings antes de archivar.
21. Si no hay bloqueos → Place pasa a estado `Archived`.
22. Sistema notifica a Listings afectados que el Place fue archivado.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| Google Place ya existe en VyteMerge | Sistema redirige al Place existente; no crea duplicado |
| Sub-place supera 3 niveles | Sistema rechaza con error descriptivo |
| Place type=Community archivado | Solo el creador puede archivar; en v1 no hay suggest edit; los otros usuarios que lo referenciaban ven advertencia |
| Resource Place archivado con Bookings futuros | Sistema bloquea; muestra lista de Bookings a resolver |
| Listing Published referencia Place archivado | Listing queda en estado degradado (warn); no puede aceptar nuevas reservas hasta que se asigne otro Place |
| Online Place — join link | El link se genera cuando el Booking se confirma (no se almacena en Place) |
| Delegado con ManagePlaces | Puede crear, editar y archivar Places del Owner; sujeto a las mismas reglas |
| Place tipo Community editado | Solo el creador puede editar en v1; suggest edit fuera de scope v1 |
| Profile eliminado | Sus Places owned pasan a estado `Orphaned` (NEEDS-CLARIFICATION) |
| Mismo googlePlaceId importado por dos usuarios | El segundo encuentra el existente y lo referencia; no se crea un segundo Place |

---

## 7. Data Model (v1 minimal)

```
Place {
  id:                UUID
  ownerProfileId:    UUID?       -- null no aplica en v1; Community creado por un usuario
  creatorProfileId:  UUID        -- quien lo creó (relevante para Community)
  parentPlaceId:     UUID?       -- FK Place (para jerarquía; null = nivel raíz)
  type:              Physical | Online | Community
  role:              Venue | Resource
  name:              string
  description:       string?
  visibility:        Public | Approximate | Hidden
  status:            Active | Archived
  timezone:          string      -- IANA (ej: "America/Buenos_Aires")
  googlePlaceId:     string?     -- clave de dedup con Google Maps
  resourceTimelineId: UUID?      -- auto-creado si role=Resource
  createdAt:         DateTime
  updatedAt:         DateTime
}

-- Solo si type=Physical:
PlaceAddress {
  placeId:     UUID
  street:      string
  number:      string?
  city:        string
  state:       string?
  country:     string            -- ISO 3166-1 alpha-2
  postalCode:  string?
  lat:         decimal?
  lng:         decimal?
}

-- Solo si type=Online:
PlaceOnlineConfig {
  placeId:   UUID
  platform:  Zoom | GoogleMeet | MicrosoftTeams | CustomUrl
  -- join link generado al confirmar Booking; no almacenado aquí
}
```

---

## 8. Commands

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `CreatePlace` | `Place` | Usuario autenticado; si parentPlaceId → jerarquía ≤ 3 niveles |
| `ImportPlaceFromGoogleMaps` | `Place` | googlePlaceId no duplicado en VyteMerge |
| `UpdatePlace` | `Place` | Place existe; status = Active; caller = owner o Delegado con ManagePlaces |
| `AddSubPlace` | `Place` | Place padre existe; jerarquía resultante ≤ 3 niveles |
| `ArchivePlace` | `Place` | No hay Bookings futuros (si Resource); Listings reasignados |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `PlaceCreated` | `CreatePlace` / `ImportPlaceFromGoogleMaps` | Si role=Resource → Timeline module crea resource Timeline |
| `PlaceUpdated` | `UpdatePlace` | — |
| `PlaceArchived` | `ArchivePlace` | Listing module: marca Listings afectados como degradados |
| `SubPlaceCreated` | `AddSubPlace` | Si role=Resource → Timeline module crea resource Timeline |

---

## 10. Invariants

1. La jerarquía de Places tiene máximo 3 niveles.
2. Un Place `Archived` no puede asignarse a nuevos Listings.
3. Un Resource Place no puede archivarse si su resource Timeline tiene Bookings futuros.
4. No pueden existir dos Places con el mismo `googlePlaceId` en VyteMerge.
5. Un Place de tipo `Community` no puede tener `role=Resource` (los recursos son siempre owned).
6. Un Community Place solo puede ser editado por su `creatorProfileId` (v1; suggest edit = v2).
7. El `resourceTimelineId` de un Resource Place es inmutable una vez creado.
8. Un Place `Online` no almacena el join link; se genera al confirmar el Booking.
9. La visibilidad de cada nivel de la jerarquía es independiente (no hereda del padre).
10. Un Place puede existir sin Listings (setup previo a publicar).

---

## 11. Outputs

- `Place` en estado `Active`, disponible para ser referenciado en Listings.
- Si `role=Resource`: resource Timeline auto-creado y gestionable desde `vt-places-mfe` o `vt-timeline-mfe`.
- Community Place: `placeId` compartido reutilizable por múltiples Profiles.
- Google Maps data importada y deduplicada (timezone, coordenadas, dirección).

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo:** `Places` (nuevo; separado de `Timeline`)
- **DbContext:** `PlacesDbContext` (schema: `places`)

```
src/Modules/Places/
├── Places.Application/
│   └── Commands/
│       ├── CreatePlace/
│       ├── ImportPlaceFromGoogleMaps/
│       ├── UpdatePlace/
│       ├── AddSubPlace/
│       └── ArchivePlace/
├── Places.Domain/
│   ├── Places/Place.cs
│   ├── Places/PlaceAddress.cs
│   ├── Places/PlaceOnlineConfig.cs
│   └── Places/Events/
├── Places.Infrastructure/
│   ├── PlacesDbContext.cs       (schema: places)
│   ├── GoogleMapsClient.cs      (integración Google Places API)
│   └── Migrations/
├── Places.IntegrationEvents/
│   ├── PlaceCreatedIntegrationEvent.cs
│   └── PlaceArchivedIntegrationEvent.cs
└── Places.Presentation/
    ├── Endpoints/
    └── Consumers/
```

**Endpoints:**
```
GET    /places                         → lista de Places del autenticado (owned + referenced)
POST   /places                         → CreatePlace
GET    /places/search?q=...            → buscar Places públicos y Community
GET    /places/google?q=...            → buscar en Google Maps API (proxy)
POST   /places/import                  → ImportPlaceFromGoogleMaps
GET    /places/:id                     → detalle
PATCH  /places/:id                     → UpdatePlace
POST   /places/:id/sub-places          → AddSubPlace
POST   /places/:id/archive             → ArchivePlace
```

**Integración Timeline:** `PlaceCreated` (role=Resource) → Timeline module crea resource Timeline con `placeId` vinculado.

**Integración Google Maps:** `GoogleMapsClient` llama a Google Places API (autocomplete + place detail) para importar `name`, `address`, `coordinates`, `timezone`.

### Frontend

**MFE:** `vt-places-mfe` (nuevo)

```
/places                      → lista de Places propios (owned + referenced)
/places/new                  → crear Place (búsqueda Google Maps + formulario manual)
/places/:id                  → detalle / editar Place
/places/:id/sub-places/new   → crear sub-place
/places/search               → buscar Places públicos/community
```

**Toolkit components (candidatos):** NEEDS-CLARIFICATION — pendiente inventario `vt-toolkit`:
- Google Maps autocomplete input
- Map preview (pin en mapa)
- Place card (nombre, tipo, rol, visibility badge)
- Hierarchy tree (padre → hijos)
- Online platform selector (Zoom | Meet | Teams | CustomUrl)
- Resource Timeline inline (vista compacta de disponibilidad del recurso)
- Status badge (Active | Archived)

**UI States:**
- **`/places`:** estado vacío + CTA "Crear Place"; lista con Places propios y referenciados; badge de tipo/rol
- **`/places/new`:** búsqueda Google Maps con sugerencias; formulario manual; detección de duplicado
- **`/places/:id`:** detalle con mapa (si Physical); sub-places como árbol colapsable; CTA "Archivar" con validaciones
- **`/places/search`:** buscador con filtros tipo/visibility; resultados con "Usar este Place" CTA

---

## 13. Acceptance Criteria

- [ ] Usuario puede crear un Place tipo Physical con nombre, dirección y timezone.
- [ ] Usuario puede crear un Place tipo Online con plataforma seleccionada.
- [ ] Al buscar con Google Maps, el sistema sugiere lugares reales y auto-carga sus datos.
- [ ] Si el googlePlaceId ya existe en VyteMerge, el sistema redirige al Place existente sin crear duplicado.
- [ ] Un Place con role=Resource auto-crea un resource Timeline al ser creado.
- [ ] El resource Timeline del Resource Place es gestionable desde vt-places-mfe y desde vt-timeline-mfe.
- [ ] Usuario puede crear sub-places de hasta 3 niveles de profundidad.
- [ ] El sistema rechaza crear un sub-place que supere 3 niveles.
- [ ] Usuario puede buscar y referenciar Community Places sin crear uno nuevo.
- [ ] Al archivar un Resource Place con Bookings futuros, el sistema bloquea y muestra los Bookings a resolver.
- [ ] Al archivar un Place con Listings Published, el sistema advierte y requiere reasignación antes de archivar.
- [ ] Un Place archivado no puede asignarse a nuevos Listings.
- [ ] Solo el creador puede editar un Community Place (v1).
- [ ] Delegado con `ManagePlaces` puede crear, editar y archivar Places del Owner.
- [ ] Architecture tests del módulo Places pasan sin errores.

---

## 14. NEEDS-CLARIFICATION

- **Profile eliminado → Places orphaned:** ¿qué pasa con los Places owned de un Profile que se elimina? ¿Se archivan automáticamente, se transfieren, o quedan en estado Orphaned?
- **Google Maps API key:** ¿VyteMerge gestiona la API key de Google Places directamente, o hay un proveedor intermedio (proxy)?
- **Community Place moderation:** en v1 solo el creador puede editar. ¿Qué mecanismo de moderación/corrección se planifica para v2?
- **Online join link generation:** ¿el link de videollamada lo genera VyteMerge vía API (Zoom API, Google Meet API), o el proveedor lo carga manualmente? Para v1 recomiendo manual; integraciones API = v2.
- **Place en discovery:** ¿los Places públicos (Venue) aparecen como resultados de búsqueda de primera clase (como un Profile), o solo como filtro dentro de Listings?
- **Resource Place de tipo Community:** actualmente el invariante lo prohibe. ¿Es definitivo, o podría haber recursos comunitarios (ej: "Cancha pública del barrio")?
- **Toolkit map component:** ¿existe componente de mapa en vt-toolkit o hay que construirlo?
