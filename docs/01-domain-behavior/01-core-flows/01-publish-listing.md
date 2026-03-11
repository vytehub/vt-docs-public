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
- Listing Published pero sin slots: **permitido** — el Business debe configurar disponibilidad en su Timeline. Si no hay Timeline configurado, el Listing se publica sin Slots proyectados. (on-request como modo distinto queda fuera de scope v1)
