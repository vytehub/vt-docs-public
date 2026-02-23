---
title: Flow 04 - Search/Discovery → abrir Listing → reservar
description: Búsqueda y descubrimiento llevan a abrir un listing y ejecutar Flow 02, respetando elegibilidad/privacidad.
---

# Flow 04 — Search/Discovery → abrir Listing → reservar

## Resumen
- **Goal:** que un usuario descubra listings (search o feed) y termine reservando.
- **Actores:** Usuario, Sistema VyteMerge.
- **Contextos:** Discovery/Search, Offer (listing), Governance (access), Supply (slots), Social (signals opcionales).

## Preconditions
- Listings publicados e indexables.
- Políticas de elegibilidad/privacidad definidas.

## Main Flow
1. Usuario abre Search o Discovery Feed.
2. Sistema construye el set candidato:
   - index (text + tags)
   - señales sociales (follows, interacción)
   - reglas de privacy/eligibility
3. Usuario aplica filtros (tags, distancia, fecha, precio, etc.) **NEEDS CLARIFICATION**
4. Usuario abre un Listing desde results.
5. Sistema muestra slots proyectados (respetando timezone y privacy).
6. Usuario ejecuta Booking (ver Flow 02).

## Domain Trace
- **Command:** `SearchListings`
  - **Read models:** SearchIndex
  - **Events:** (normalmente ninguno; puede haber telemetry)
- **Projection:** `ApplyEligibilityAndPrivacyFilters`
- **Command:** `OpenListing` → read models (listing + slots)
- **Command:** `CreateBooking` → ver Flow 02

## Links
- Search overview: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/04.search_discovery/03.search.md`
- Discovery overview: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/04.search_discovery/04.discovery.md`
- Eligibility & privacy: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/04.search_discovery/05.eligibility-privacy.md`
