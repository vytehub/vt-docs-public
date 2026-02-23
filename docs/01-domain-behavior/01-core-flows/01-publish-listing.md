---
title: Flow 01 - Publicar Listing y habilitar Slots
description: Desde un Business crea/pulsa un Listing, configura Timeline y el sistema proyecta Slots para booking.
---

# Flow 01 — Publicar Listing y habilitar Slots

## Resumen
- **Goal:** que un negocio pueda publicar una oferta (Listing) y exponer disponibilidad proyectada (Slots) basada en su Timeline + reglas.
- **Actores:** Business (owner), Sistema VyteMerge.
- **Contextos:** Offer (Catalog/Listings), Supply (Timelines/Places), Governance (Access), Discovery (index), Communication (notifications opcional).

## Preconditions
- Business tiene Profile activo.
- Existe (o se crea) un **Service/Product** en el Catálogo.
- Existe un **Place** asociado (con timezone) si el listing es presencial o si afecta cálculo.
- Business tiene un **Timeline** (propio o acordado vía Agreements) donde aplica disponibilidad.

## Main Flow (paso a paso)
1. Business crea o selecciona un **Service/Product** (qué).
2. Business crea un **Listing** en estado **Draft** (cómo se vende/ofrece):
   - duración / precio / reglas de booking / capacidad
   - place / modalidad
   - visibility (public/unlisted/private)
   - form (intake) si aplica
3. Business configura disponibilidad en su **Timeline** (cuándo):
   - events existentes (bloqueos)
   - reglas de disponibilidad (si aplica)
4. Sistema recalcula/proyecta **Slots** (disponibilidad derivada) para ese Listing:
   - aplica conflict rules del timeline
   - aplica reglas de capacity/assignment
5. Business publica el Listing (**Published**).
6. Sistema:
   - habilita el listing para surfaces (link, channels, discovery/feed según reglas)
   - (si corresponde) indexa el listing en Search/Discovery.

## Domain Trace (Command → Aggregate → Event)
- **Command:** `CreateServiceOrProduct` (si no existe)
  - **Aggregate:** `CatalogItem`
  - **Events:** `ServiceCreated` / `ProductCreated`
- **Command:** `CreateListing`
  - **Aggregate:** `Listing`
  - **Events:** `ListingCreated` (Draft)
- **Command:** `ConfigureTimelineAvailability` (o `UpsertEvents`/`UpsertRules`)
  - **Aggregate:** `Timeline`
  - **Events:** `TimelineUpdated`
- **Projection:** `ProjectSlotsForListing`
  - **Inputs:** Timeline + Listing rules + Place timezone
  - **Outputs:** `SlotsProjected` (o actualización de read model de slots)
- **Command:** `PublishListing`
  - **Aggregate:** `Listing`
  - **Invariants:** listing válido; reglas mínimas completas; permisos ok.
  - **Events:** `ListingPublished`
- **Projections:** 
  - `DiscoveryIndexUpdated`
  - `FeedEligibilityUpdated` (si aplica)
  - `Link/ChannelVisibilityUpdated`

## Edge Cases
- Timeline con conflictos: slots se recortan o se marcan como no disponibles.
- Timezone: cálculos de slots deben usar el timezone del Place/Timeline (no “UTC a ojo”).
- Listing Published pero sin slots: permitido (para “on request”) o bloqueado (decisión de producto). **NEEDS CLARIFICATION**.

## Links (fuente de verdad)

- [Core Model](../../00-core-domain/03-core-model/core-model.md)
- [Listing Overview/Rules/Lifecycle](../../00-core-domain/04-bounded-contexts/03.Offer%20-%20Catalog%20&%20Listings/02.listing/01.overview.md)
- [Listing Status & Lifecycle](../../00-core-domain/04-bounded-contexts/03.Offer%20-%20Catalog%20&%20Listings/02.listing/13.status_lifecycle.md)
- [Timeline + conflict rules](../../00-core-domain/04-bounded-contexts/02.Supply%20-%20Time%20&%20Place/01.timelines/02.timeline_conflict_rules.md)
- [Place + timezone](../../00-core-domain/04-bounded-contexts/02.Supply%20-%20Time%20&%20Place/02.places/04.timezone.md)

