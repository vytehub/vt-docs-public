---
title: Flow 06 — Crear Listing
description: >
  Un Business (o Staff/Delegado con permisos) crea un Listing sobre un Service activo,
  configura reglas comerciales, de scheduling y de disponibilidad, y lo publica para
  habilitarlo en los canales seleccionados.
status: draft
version: 1
---

# Flow 06 — Crear Listing

## 1. Resumen
- **Goal:** que un Business pueda crear un Listing sobre un Service activo, configurarlo
  y publicarlo para que sea reservable en los canales correspondientes.
- **Actores:**
  - **Primary:** Business (owner del Profile).
  - **Secondary:** Staff / Delegado con Agreement que incluya permisos de gestión de Listings.
- **Surfaces:** Service Detail (vt-catalog-mfe) → Formulario Crear Listing (vt-listings-mfe) → Listing Detail.

---

## 2. Domain Context

### Separación Catalog / Listing (decisión v1 — ver Flow 05)
| Dominio | Entidad | Responsabilidad |
|---------|---------|-----------------|
| **Catalog** | `Service` | Operativo: duración, precio base, confirmationPolicy, intakeForm, location |
| **Offer** | `Listing` | Comercial/discovery: copy, media, precio efectivo, slotConfig, visibility, add-ons |

Reglas de relación:
- **1 Service → N Listings** (misma oferta, múltiples canales, precios, plazas o contextos)
- `Service.basePrice` es referencia; `Listing.price` sobreescribe el precio efectivo (ADR-0004 v1).
- `EffectivePrice = Listing.price` (si está definido) o `Service.basePrice` (fallback en Draft).
- `priceModifiers[]` por Place/modalidad → reservado para v2 (ver ADR-0004).
- Booking guarda **snapshot** del precio final al momento de la reserva.
- Discovery usa **Listing**. Booking usa **Service** (snapshot).

### Scope v1
- Solo **ServiceListing** (tipo reservable con Slots).
- ProductListing independiente queda para v2.
- Un Listing puede incluir **Add-ons** (productos o extensiones de servicio dentro del mismo checkout).

### SlotConfig ownership (ADR-0001: accepted)
SlotConfig pertenece al Listing (BC Offer). Supply lee SlotConfig para proyectar Slots,
pero no lo posee. `CreateListing` / `UpdateListingRules` llevan SlotConfig como parte del payload.

---

## 3. Preconditions
- Business tiene Profile activo.
- El Service referenciado está en estado **Active** (regla de Flow 05 / invariante 2).
- Existe al menos un Place (Lugar) asociado al Profile (o se crea en este flujo).
  - Si el Service usa `location` como free-text o PlaceId, debe resolverse antes de publicar.
    (NEEDS-CLARIFICATION: resolución del campo `location` en Service — ver Flow 05 item abierto)

---

## 4. Trigger
Business navega a **Service Detail** y pulsa **"Crear Listing"**
(o accede a `vt-listings-mfe` directamente con `?serviceId=...`).

---

## 5. Main Flow

1. Business abre Service Detail (estado Active) y pulsa **"Crear Listing"**.
2. Sistema crea un `Listing` en estado **Draft** vinculado a `serviceId` y `profileId`.
3. Business completa el formulario:
   - **Requeridos para publicar:**
     - `title` — nombre visible de la oferta.
     - `description` — descripción de la oferta.
     - `placeId` — dónde ocurre el servicio (referencia a Place existente del Profile).
     - `price` — precio efectivo (≥ 0); sobreescribe `Service.basePrice`.
     - `confirmationPolicy` — `AutoConfirm | ManualConfirm | RequestOnly`.
     - `visibility` — `Public` o `Private`.
     - `slotConfig` — reglas de scheduling (ADR-0001):
       - `durationMin` (> 0)
       - `bookingWindow` (ventana mínima y máxima de anticipación)
       - `preBufferMin` y `postBufferMin` (opcionales en Draft, requeridos para publicar si Service los tiene)
   - **Opcionales:**
     - `media[]` — fotos/video del listing (fallback a Service.media si está vacío).
     - `intakeForm` — lista de `FieldDefinition`; preguntas al cliente al reservar.
     - `addOns[]` — productos o extensiones agregables al checkout.
     - `tags[]` — etiquetas de discovery.
     - `capacity` — cupos por slot (default: 1 para servicios 1:1).
4. Business guarda en Draft (puede hacerlo de forma parcial sin completar todos los campos requeridos).
5. Business pulsa **"Publicar"**.
6. Sistema valida que todos los campos requeridos están completos y válidos.
7. Sistema transiciona el Listing a **Published**.
8. Sistema:
   - Dispara `ProjectSlots(listingId, dateRange)` → Supply proyecta Slots.
   - Indexa el Listing en Discovery (`DiscoveryIndexUpdated`).
   - Actualiza eligibilidad en Feed (`FeedEligibilityUpdated`).
9. UI muestra **Listing Detail** con estado Published y el link compartible del Listing.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|---------------|
| Publicar con campos incompletos | Validación campo a campo; Listing permanece en Draft con errores inline |
| Service pasa a Archived mientras Listing está en Draft | Sistema bloquea `PublishListing`; muestra error "el Service referenciado no está activo" |
| Staff/Delegado crea el Listing | Sistema verifica Agreement: debe incluir permiso `listing:create`. Si no tiene permiso de publicar (`listing:publish`), solo puede guardar en Draft. |
| Price modifier para un Place no configurado | Sistema usa `Service.basePrice` como fallback; muestra advertencia al Business. |
| Publicar sin slotConfig completo | Validación bloquea la publicación; `durationMin` es mínimo obligatorio. |
| Listing Published — editar campos comerciales | Cambio se aplica; Bookings existentes **no se modifican** (usan snapshot); Slots se reproyectan si cambió slotConfig. |
| Archivar Listing Published | Transición a Archived; Slots existentes se eliminan/invalidan; Bookings existentes se preservan. |
| Intentar reactivar Listing Archived | **Bloqueado**; para re-vender se crea un nuevo Listing. |
| Timeline sin disponibilidad al publicar | Listing se publica; Slots proyectados quedan vacíos. El Business debe configurar disponibilidad en su Timeline. (No se bloquea la publicación — la disponibilidad es responsabilidad del Timeline, no del Listing) |

---

## 7. Data Model (v1 minimal)

```
Listing {
  id:                UUID
  profileId:         UUID                  -- FK Profile (owner)
  serviceId:         UUID                  -- FK Service (source); Service debe estar Active al publicar
  type:              ServiceListing        -- v1: siempre ServiceListing
  status:            Draft | Published | Unpublished | Archived
  title:             string                -- requerido para Published
  description:       string?               -- requerido para Published
  placeId:           UUID                  -- requerido para Published (ServiceListing v1)
  price:             decimal               -- requerido para Published; ≥ 0; override de Service.basePrice
  -- priceModifiers[]: reservado para v2 (ADR-0004)
  confirmationPolicy: AutoConfirm | ManualConfirm | RequestOnly  -- requerido para Published
  visibility:        Public | Private      -- requerido para Published
  slotConfig:        SlotConfig            -- requerido para Published (ADR-0001)
  intakeForm:        FieldDefinition[]     -- opcional; preguntas al cliente al reservar
  addOns:            AddOn[]               -- opcional; productos o extensiones de servicio
  media:             Media[]               -- opcional; fallback a Service.media
  capacity:          int                   -- default 1; cupos por slot
  tags:              Tag[]                 -- opcional
  createdAt:         DateTime
  updatedAt:         DateTime
}

SlotConfig {                               -- owned by Listing (ADR-0001)
  durationMin:       int                   -- > 0; requerido para Published
  preBufferMin:      int?
  postBufferMin:     int?
  bookingWindow: {
    minNoticeHours:  int                   -- anticipación mínima para reservar
    maxAdvanceDays:  int                   -- horizonte máximo de disponibilidad
  }
  -- NEEDS-CLARIFICATION: campos adicionales de 04.scheduling_slots.md
}

FieldDefinition {                          -- ítem del intakeForm; owned by Listing
  label:             string                -- texto de la pregunta
  type:              text | select | bool  -- tipo de campo
  required:          bool                 -- obligatorio responder antes de completar el booking
  options:           string[]?            -- solo para type = select
}

-- PriceModifier: reservado para v2 (ADR-0004)
-- { placeId, modality, price } — precios por Place/modalidad → fuera de scope v1
```

---

## 8. Commands

| Command | Aggregate | Precondition |
|---------|-----------|--------------|
| `CreateListing` | `Listing` | Service Active; Profile activo |
| `UpdateListing` | `Listing` | Status ≠ Archived |
| `PublishListing` | `Listing` | Todos los campos requeridos presentes y válidos; Service Active |
| `UnpublishListing` | `Listing` | Status = Published |
| `ArchiveListing` | `Listing` | Status ≠ Archived |

---

## 9. Events

| Event | Disparado por |
|-------|--------------|
| `ListingCreated` | `CreateListing` |
| `ListingUpdated` | `UpdateListing` |
| `ListingPublished` | `PublishListing` → triggers `ProjectSlots` en Supply |
| `ListingUnpublished` | `UnpublishListing` → triggers invalidación de Slots en Supply |
| `ListingArchived` | `ArchiveListing` → triggers eliminación de Slots en Supply |

---

## 10. Invariants

1. Un Listing pertenece a exactamente un Profile.
2. Solo Services en estado **Active** pueden tener nuevos Listings creados sobre ellos.
3. `title` no puede estar vacío para publicar.
4. `placeId` es requerido para ServiceListing v1.
5. `slotConfig.durationMin` debe ser > 0 para publicar.
6. `visibility` es requerido para publicar.
6a. `confirmationPolicy` es requerido para publicar.
7. El `profileId` del Listing debe coincidir con el `profileId` del Service referenciado
   (salvo Agreement con `listing:create` habilitado).
8. Cambios en campos comerciales de un Listing Published **no modifican Bookings existentes**
   ni recalculan Slots retroactivamente (Bookings guardan snapshot del precio/términos).
9. Un Listing **Archived no puede volver a Published**; para re-vender se crea un nuevo Listing.
10. Si el Service referenciado pasa a Archived mientras el Listing está en Draft,
    `PublishListing` queda **bloqueado**.

---

## 11. Outputs

- `Listing` en estado **Published**, asociado al Profile y al Service.
- Slots proyectados disponibles en Supply (si el Timeline tiene disponibilidad configurada).
- Listing indexado en Discovery y elegible para Feed.
- Link compartible del Listing disponible para el Business.

---

## 12. Technical Mapping (Draft)

### Backend
- **Módulo nuevo:** `Listing` (BC: Offer; separado de `Catalog` — ver ADR-0003)
- **Aggregate:** `Listing`
- **DbContext:** `ListingDbContext` (schema: `listing`)
- **Endpoints draft:**
  - `POST   /listings`                     → `CreateListing`
  - `GET    /listings/{id}`                → detalle
  - `GET    /listings?profileId={id}`      → lista por profile
  - `PATCH  /listings/{id}`                → `UpdateListing`
  - `POST   /listings/{id}/publish`        → `PublishListing`
  - `POST   /listings/{id}/unpublish`      → `UnpublishListing`
  - `POST   /listings/{id}/archive`        → `ArchiveListing`
- **Integración Supply:** al publicar, `ListingPublished` event dispara `ProjectSlots(listingId, dateRange)` en el módulo Supply.

### Frontend
- **MFE nuevo:** `vt-listings-mfe`
- **Rutas draft:**
  - `/listings`                            → listado de Listings del Business
  - `/listings/new?serviceId={id}`         → formulario de creación (con Service pre-seleccionado desde Flow 05 CTA)
  - `/listings/{id}/edit`                  → edición de Listing (Draft o Published)
  - `/listings/{id}`                       → Listing Detail (Published, owner view)
  - `/listings/{id}/preview`               → preview (Draft, solo para owner)
- **Toolkit components:** NEEDS-CLARIFICATION (inventario de vt-toolkit pendiente)
- **UI states:**
  - catálogo vacío
  - formulario Draft (paso a paso o una sola pantalla)
  - errores de validación inline por campo
  - confirmación de publicación
  - Listing Detail Published con link compartible
  - estado Archived (read-only)

---

## 13. Acceptance Criteria

- [ ] Business puede crear un Listing en Draft vinculado a un Service Active.
- [ ] Business puede guardar un Listing en Draft de forma parcial (sin todos los campos requeridos).
- [ ] Sistema bloquea la publicación si faltan campos requeridos, indicando cuáles.
- [ ] Business puede publicar el Listing una vez completados todos los campos requeridos.
- [ ] Al publicar, el sistema dispara la proyección de Slots en Supply.
- [ ] Al publicar, el Listing aparece en Discovery según su `visibility`.
- [ ] Listing con `priceModifiers` muestra el precio correcto según el Place asociado; fallback a `Service.basePrice` si no hay modifier.
- [ ] Actualizar campos comerciales de un Listing Published no modifica Bookings existentes.
- [ ] No se puede publicar un Listing si su Service referenciado está Archived.
- [ ] Staff/Delegado sin permiso `listing:publish` puede crear en Draft pero no publicar.
- [ ] Listing Archived no puede volver a Published; el sistema rechaza la operación.
- [ ] Listing Detail Published muestra el link compartible del Listing.

---

## 14. NEEDS-CLARIFICATION

- **`slotConfig` completo:** campos adicionales de `04.scheduling_slots.md` que deben ser requeridos para publicar
  vs opcionales (ej: ¿bookingWindow es requerida o tiene defaults razonables?).
- **`AddOns` estructura:** ¿un AddOn referencia un ProductId del Catálogo, o puede ser un item custom de texto+precio?
  Ver `03.add_ons_bundles.md`.
- **Permisos de Staff/Delegado:** ¿qué operaciones exactas incluye el permiso `listing:create`? ¿Solo crear Draft,
  o también editar y archivar? ¿`listing:publish` es un permiso separado?
- **ADR-0003:** Listing como módulo separado de Catalog — propuesto, pendiente de aceptación formal.
- **`priceModifiers[]` v2:** cuando se implemente, ver ADR-0004 para modelo por Place/modalidad.
