---
title: Notación & Plantillas
description: Convenciones para describir flujos end-to-end en VyteMerge (Command → Aggregate → Event).
---

# Notación & Plantillas

## Objetivo
Que cualquier persona (dev/PM/diseño/IA) pueda leer un flujo y entender:
- quién inicia la acción (**actor**)
- qué se intenta hacer (**command**)
- qué reglas se aplican (**aggregate + invariants**)
- qué hechos quedan registrados (**events**)
- qué superficies se actualizan (**read models**)

## Plantilla de Flow

### 1) Resumen
- **Nombre del flow**
- **Goal**
- **Actores**
- **Contextos involucrados** (Offer / Supply / Social / Discovery / Communication / Governance)

### 2) Preconditions
- Estados requeridos (ej: Listing = Published)
- Permisos / visibilidad
- Place / timezone si aplica

### 3) Main Flow (paso a paso)
Lista numerada con resultados observables.

### 4) Domain Trace (command/event)
Formato recomendado:
- **Command:** `X`
  - **Aggregate:** `Y`
  - **Invariants:** `...`
  - **Event(s):** `A`, `B`
- **Projection(s):** `...` (slots, search index, feed, notifications)

### 5) Edge Cases
- conflictos de timeline
- overbooking/capacity
- privacidad / opt-in
- timezones
- idempotencia (reintentos)

### 6) Acceptance Criteria
Qué significa “done” a nivel de producto.

## Convenciones de nombres (sugeridas)
- Commands: `VerbNoun` (CreateListing, PublishListing, RequestBooking, ApproveFollow, CreatePost, ...)
- Events: `NounVerbPastTense` (ListingCreated, ListingPublished, BookingRequested, BookingConfirmed, FollowApproved, PostCreated, ...)

> Nota: los nombres exactos pueden variar, pero la intención debe ser consistente.
