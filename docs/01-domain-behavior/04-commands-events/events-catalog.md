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
**Commands**
- RequestBooking / CreateBooking
- ConfirmBooking (si aplica)
- CancelBooking
- MarkNoShow

**Events**
- BookingRequested / BookingCreated
- BookingConfirmed
- BookingCancelled
- BookingNoShow
- BookingCompleted

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
