---
title: Flow 05 — Crear Service
description: >
  Un Business crea un Service en su Catálogo, definiendo reglas operativas
  (duración, precio, buffers, form, policy) que serán fuente de verdad para
  los Listings asociados.
status: draft
version: 1
---

# Flow 05 — Crear Service

## 1. Resumen
- **Goal:** que un Business pueda crear un Service (definición operativa reutilizable)
  en su Catálogo, activarlo y quedar listo para crear Listings sobre él.
- **Actores:** Business (owner del Profile).
- **Surfaces:** Dashboard → Catálogo → Nuevo Service → Service Detail.

---

## 2. Domain Context

### Separación Catalog / Listing (decisión v1)
| Dominio | Entidad | Responsabilidad |
|---------|---------|-----------------|
| **Catalog** | `Service` | Operativo: nombre, duración, precio base, buffers |
| **Listing** | `Listing` | Comercial/discovery: copy, media, tags, slotConfig, confirmationPolicy, visibility, intakeForm, addOns, placeId |

> **Decisiones v1:** `location` (como placeId), `confirmationPolicy`, `intakeForm` y `media`
> pertenecen al Listing, no al Service. Ver data model abajo (líneas 96-101).

Reglas de relación:
- **1 Service → N Listings**
- `Service.basePrice` es la fuente de verdad del precio.
- Listing aplica **modificadores** (`EffectivePrice = basePrice + modifiers`).
- Booking guarda **snapshot** del precio final al momento de la reserva.
- Discovery usa **Listing**. Booking usa **Service**.

> ⚠️ Esto actualiza los docs en `02.services.md` y `01.catalog/01.overview.md`,
> donde precio/buffers/forms figuran como responsabilidad de Listing.
> La decisión correcta (v1) es que pertenecen al Service.

---

## 3. Preconditions
- Business tiene Profile activo.
- No se requiere Timeline ni Place previo para crear el Service
  (esos se vinculan al crear el Listing).

---

## 4. Trigger
Business navega a **Dashboard → Catálogo → "Nuevo Service"**.

---

## 5. Main Flow

1. Business abre la sección Catálogo y pulsa "Nuevo Service".
2. Sistema crea un `Service` en estado **Draft** (asociado al `profileId`).
3. Business completa el formulario:
   - **Requeridos para activar:** `name`, `durationMin` (> 0), `basePrice` (≥ 0).
   - **Opcionales:** `description`, `preBufferMin`, `postBufferMin`, `tags`.
4. Business guarda en Draft (puede guardar parcialmente sin activar).
5. Business pulsa **"Activar"**.
6. Sistema valida que todos los campos requeridos están completos y válidos.
7. Sistema transiciona el Service a **Active**.
8. Sistema muestra **Service Detail** con CTA **"Crear Listing"**.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|---------------|
| Activar con campos incompletos | Validación → error por campo; Service permanece en Draft |
| Actualizar Service Active | Cambio se aplica; Bookings existentes **no se modifican**; Slots ya generados **no se recalculan** |
| Archivar Service | Transición a Archived; no se generan nuevos Slots; Bookings existentes se preservan |
| Eliminar Service con Bookings | **Bloqueado**; el sistema rechaza la operación |
| Eliminar Service sin Bookings | Permitido (soft delete recomendado) |
| Crear Listing desde Service Archived | **Bloqueado**; solo Services Active pueden tener nuevos Listings |

---

## 7. Data Model (v1 minimal)

```
Service {
  id:                   UUID
  profileId:            UUID               -- FK Profile (owner)
  status:               Draft | Active | Archived
  name:                 string             -- requerido para Active
  description:          string?
  durationMin:          int                -- > 0; requerido para Active
  basePrice:            decimal            -- ≥ 0; requerido para Active
  preBufferMin:         int?
  postBufferMin:        int?
  tags:                 Tag[]              -- taxonomía global curada + custom limitado
  -- location: no existe en Service; placeId vive en Listing (decisión v1)
  -- confirmationPolicy: vive en Listing; Service es puramente operativo (decisión v1)
  -- intakeForm: vive en Listing (decisión v1)
  createdAt:            DateTime
  updatedAt:            DateTime
  -- baseMedia: no existe en Service; la media pertenece al Listing (ver decisión v1)
}
```

> **Media (decisión v1):** Service no almacena media. La pantalla "Crear Service" puede
> incluir upload de imágenes para mejor UX, pero internamente esa media se almacena en el
> **Listing** auto-creado. Esto permite múltiples Listings con diferentes visuales y
> habilita A/B tests y creatividades de campaña.

---

## 8. Commands

| Command | Aggregate | Precondition |
|---------|-----------|--------------|
| `CreateService` | `Service` | Profile activo |
| `UpdateService` | `Service` | Status ≠ Archived |
| `ActivateService` | `Service` | Todos los campos requeridos presentes y válidos |
| `ArchiveService` | `Service` | Status = Active |

---

## 9. Events

| Event | Disparado por |
|-------|--------------|
| `ServiceCreated` | `CreateService` |
| `ServiceUpdated` | `UpdateService` |
| `ServiceActivated` | `ActivateService` |
| `ServiceArchived` | `ArchiveService` |

---

## 10. Invariants

1. Un Service pertenece a exactamente un Profile.
2. `name` no puede ser vacío.
3. `durationMin` debe ser > 0 para activar.
4. `basePrice` debe ser ≥ 0 para activar.
5. Un Service no puede eliminarse si tiene Bookings asociados.
6. Cambios en Service Active no modifican Bookings existentes ni recalculan Slots retroactivamente.
7. Solo Services Active pueden tener nuevos Listings creados sobre ellos.

---

## 11. Outputs

- `Service` en estado **Active**, asociado al Profile del Business.
- Service Detail visible con CTA **"Crear Listing"** (entrada al Flow 06).

---

## 12. Technical Mapping (Draft)

### Backend
- **Módulo nuevo:** `Catalog` (separado del módulo `Offering` existente).
  - El módulo `Offering` es la versión anterior; la separación Catalog/Listing es la decisión v1.
- **Aggregate:** `Service`
- **DbContext:** `CatalogDbContext` (schema: `catalog`)
- **Endpoints draft:**
  - `POST   /catalog/services`               → `CreateService`
  - `GET    /catalog/services/{id}`          → detalle
  - `GET    /catalog/services`               → lista por profile
  - `PATCH  /catalog/services/{id}`          → `UpdateService`
  - `POST   /catalog/services/{id}/publish`  → `PublishService` (activa el Service; UX label = "Activar")
  - `POST   /catalog/services/{id}/archive`  → `ArchiveService`
- **Persistence:** tabla `catalog.services`

### Frontend
- **Rutas draft:**
  - `/catalog`                  → listado de Services del Business
  - `/catalog/services/new`     → formulario de creación (Draft)
  - `/catalog/services/:id`     → Service Detail + CTA "Crear Listing"
- **Toolkit components:** NEEDS-CLARIFICATION (inventario de vt-toolkit pendiente)
- **UI states:** catálogo vacío · formulario Draft · errores de validación ·
  confirmación de activación · Service Detail Active · estado Archived (read-only)

---

## 13. Acceptance Criteria

- [ ] Business puede crear un Service en Draft sin completar todos los campos.
- [ ] Sistema bloquea la activación si faltan campos requeridos, indicando cuáles.
- [ ] Business puede activar el Service una vez los campos requeridos están completos.
- [ ] Service activado aparece en Catálogo del Business.
- [ ] Actualizar un Service Active no modifica Bookings existentes ni Slots generados.
- [ ] No se puede eliminar un Service con Bookings asociados.
- [ ] Service Detail muestra CTA "Crear Listing" solo si status = Active.
- [ ] Service Archived no permite crear nuevos Listings.

---

## 14. NEEDS-CLARIFICATION

- **Tags:** cantidad máxima de tags custom permitidos por Service.
- **Módulo Offering:** ¿se renombra, convive con Catalog, o se refactoriza?
  Impacta migraciones y módulos existentes. Ver ADR-0003.
- **Toolkit components:** pendiente inventario de `vt-toolkit` para formulario.
