---
title: Flow 02 - Abrir Listing y crear Booking
description: Un Guest/Usuario abre un listing (o slot) y confirma una reserva que termina creando un Booking y un Event en el Timeline.
---

# Flow 02 — Abrir Listing y crear Booking

## Resumen
- **Goal:** permitir que un usuario reserve un slot (o solicite un horario) y el sistema cree un Booking + Event en el Timeline correspondiente.
- **Actores:** Guest/Usuario, Sistema VyteMerge, Business.
- **Contextos:** Offer (Listing), Supply (Timeline/Slots), Governance (Access), Communication (notifications), (opcional) Payments/Orders.

## Preconditions
- Listing está **Published** y visible para el usuario (channels/visibility + privacy).
- Existe disponibilidad proyectada (Slots). (on-request fuera de scope v1)
- Se conoce Place/timezone para interpretación del horario.

## Main Flow (paso a paso)
1. Usuario abre el Listing (por link, feed, search) o abre un Slot deep link.
2. Sistema valida Access/Visibility:
   - perfil/owner visibility
   - listing visibility (public/unlisted/private)
   - reglas de eligibility (si existen)
3. Usuario selecciona un Slot (o propone alternativa) y completa el Form (intake).
4. Sistema intenta crear la reserva:
   - revalida disponibilidad (para evitar carreras)
   - aplica capacity/shared capacity
   - aplica booking rules (ventanas, políticas, etc.)
5. Sistema crea **Booking** (estado inicial) y crea/actualiza un **Event** en el Timeline del Business (y/o del usuario según diseño).
6. Sistema emite notificaciones:
   - confirmación al usuario
   - aviso al business
7. UI muestra confirmación y detalle (con timezone correcto).

## Domain Trace (Command → Aggregate → Event)
- **Command:** `ViewListing` (lectura)
  - **Read models:** listing details + projected slots
- **Command:** `RequestBooking` / `CreateBooking`
  - **Aggregate:** `Booking`
  - **Invariants:** slot disponible; políticas; capacity; privacy.
  - **Events:** `BookingRequested` / `BookingCreated`
- **Command:** `AttachBookingToTimeline`
  - **Aggregate:** `Timeline`
  - **Events:** `EventCreated` / `EventUpdated` (con referencia a booking)
- **Projections:**
  - `SlotsReprojected` / slot marcado como reservado
  - `NotificationsEnqueued`
  - `DashboardMetricsUpdated` (si aplica)
- **Opcional Payments:**
  - **Command:** `CreateOrder` → **Event:** `OrderCreated` → `PaymentCaptured`

## Edge Cases
- Carreras: dos usuarios intentando el mismo slot (idempotencia + recheck).
- Cancelación/no-show: ver lifecycle.
- Timezone mismatch: slot/hora debe persistirse con reglas claras (DateOnly vs DateTime). 
- Listing Unpublished mientras usuario está en pantalla: bloquear confirmación.

## Links (fuente de verdad)
- Slots & Booking rules (Listing): `docs/00-core-domain/04-bounded-contexts/03.Offer - Catalog & Listings/02.listing/05.booking_rules.md`
- Capacity/shared capacity: `docs/00-core-domain/04-bounded-contexts/03.Offer - Catalog & Listings/02.listing/06.capacity_shared_capacity.md`
- Scheduling / slots: `docs/00-core-domain/04-bounded-contexts/03.Offer - Catalog & Listings/02.listing/04.scheduling_slots.md`
- Timeline conflict rules: `docs/00-core-domain/04-bounded-contexts/02.Supply - Time & Place/01.timelines/02.timeline_conflict_rules.md`
- Notifications overview: `docs/00-core-domain/04-bounded-contexts/05.Communication/02.notifications/01.overview.md`
