---
title: Commands & Events Catalog (draft)
description: Catálogo inicial de comandos/eventos mencionados en Domain Behavior para mantener consistencia.
---

# Commands & Events Catalog (draft)

> Este catálogo es **mínimo** y se expande a medida que se curan flows.

## Offer / Catalog / Listing
**Commands**
- CreateServiceOrProduct
- CreateListing
- PublishListing
- UnpublishListing
- ArchiveListing
- UpdateListingRules
- UpdateListingVisibility

**Events**
- ServiceCreated / ProductCreated
- ListingCreated
- ListingPublished
- ListingUnpublished
- ListingArchived
- ListingUpdated

## Supply / Timeline
**Commands**
- UpsertEvents
- ConfigureAvailabilityRules
- ShareTimeline (via Agreements)

**Events**
- TimelineUpdated
- TimelineShared
- EventCreated / EventUpdated / EventCancelled
- SlotsProjected (projection event)

## Booking

> **RequestBooking vs CreateBooking — Distinción importante**
>
> La elección del comando depende de `Listing.confirmationPolicy`:
>
> | `confirmationPolicy` | Comando a usar | Booking inicial | Evento emitido |
> |---|---|---|---|
> | `ManualConfirm` \| `RequestOnly` | `RequestBooking` | `Pending` (espera aprobación) | `BookingRequested` |
> | `AutoConfirm` | `CreateBooking` | `Confirmed` (auto-confirmado) | `BookingCreated` |
>
> No son sinónimos: representan flujos distintos con estados de inicio diferentes.

**Commands**
- `RequestBooking` — cuando `confirmationPolicy = ManualConfirm | RequestOnly`; el proveedor debe confirmar manualmente.
- `CreateBooking` — cuando `confirmationPolicy = AutoConfirm`; confirmación automática inmediata.
- `ConfirmBooking` — el proveedor aprueba un booking en estado `Pending`.
- `CancelBooking` — cancela desde cualquier estado activo (`Pending` o `Confirmed`).
- `MarkNoShow` — marca no-show desde estado `Confirmed`.
- `CompleteBooking` — cierra el booking desde estado `Confirmed`.

**Events**
- `BookingRequested` — booking creado en estado `Pending`, pendiente de confirmación manual.
- `BookingCreated` — booking creado y auto-confirmado.
- `BookingConfirmed` — proveedor aprobó un `BookingRequested`.
- `BookingCancelled` — cancelado (desde `Requested` o `Confirmed`).
- `BookingNoShow` — marcado como no-show.
- `BookingCompleted` — cerrado exitosamente.

## Social
**Commands**
- RequestFollow
- ApproveFollow / RejectFollow
- CreatePost
- ReactToPost

**Events**
- FollowRequested
- FollowApproved / FollowRejected
- PostCreated
- PostReacted

## Discovery
**Commands**
- SearchListings
- BuildDiscoveryFeed

**Events**
- DiscoveryIndexUpdated (projection)
- FeedEligibilityUpdated (projection)
