# Temporal Graph Node Taxonomy v1

## Status
Draft v1

## Purpose
This document defines the **nature of nodes** in the VyteMerge temporal graph model.

It exists to answer:
- what kinds of nodes exist
- which ones are real domain entities
- which ones are derived graph projections
- which ones are purely visual markers
- what minimum data each node needs
- what relationships each node can have
- where each node should appear

This is a classification and UI/architecture alignment document.

It is not yet a final database schema.
It is not yet a final design document.
It is the bridge between:
- domain model
- projection layer
- UI rendering model

---

# 1. Core principle

Not every visible node in the graph has the same ontological status.

Some nodes are:
- **real domain entities**
- persisted as part of business truth
- with lifecycle, ownership, and rules

Other nodes are:
- **derived projection nodes**
- created by the graph/read-model layer
- useful for rendering, navigation, and comprehension

Other nodes are:
- **purely visual markers**
- never persisted as first-class business objects
- only part of the UI language

This distinction is critical.

---

# 2. Three node categories

## A. Domain Entity Nodes
Real business/domain entities.

They generally have:
- identity
- lifecycle
- ownership
- rules
- persistence
- auditability
- business meaning independent of UI

## B. Derived Projection Nodes
Nodes built by the projection/read-model layer.

They generally:
- summarize
- group
- connect
- emphasize
- create graph readability
- do not need to exist as primary domain truth

## C. Purely Visual Markers
UI-only tokens.

They generally:
- indicate position
- indicate time progression
- indicate focus
- indicate alignment
- indicate selection
- do not belong in the core domain model

---

# 3. Domain Entity Node Families

## 3.1 Profile Node

### What it represents
A person or actor in the ecosystem.

Examples:
- current user
- provider
- customer
- attendee
- professional
- organizer
- staff
- parent
- reseller
- invited friend

### Category
**Domain entity**

### Why
A profile has:
- identity
- lifecycle
- permissions
- relationships
- activity
- persistence

### Minimum data
- profile_id
- display_name
- avatar/image
- profile_type or contextual role
- relation_to_viewer
- visibility/access context

### Typical relationships
- owns listing
- books event
- attends event
- invited to event
- follows profile
- belongs to organization
- has agreement with organization
- manages resource/timeline (if delegated)

### Typical appearance
- Home
- Explore
- Profile
- Studio
- timeline preview or shared event contexts

---

## 3.2 Organization Node

### What it represents
A business, institution, or organizational actor.

Examples:
- hospital
- school
- football club
- salon
- veterinary clinic
- beauty center
- partner business

### Category
**Domain entity**

### Minimum data
- organization_id
- display_name
- logo/image
- type
- owned places/resources
- access/channel context

### Typical relationships
- owns places/resources
- publishes listings
- employs or works with profiles
- creates agreements
- hosts channels
- governs audience visibility

### Typical appearance
- Explore
- Studio
- Profile
- deeper graph and organization views

---

## 3.3 Service Node

### What it represents
A conceptual service.

Examples:
- manicure
- consultation
- surgery
- photo session
- football field rental
- professional availability by hour
- home plumbing visit

### Category
**Domain entity**

### Minimum data
- service_id
- title
- category
- default duration
- service type
- required resource/place/professional rules

### Typical relationships
- sold through listing
- scheduled as event
- reserved through booking
- occurs at place/resource
- may be bundled with other services/products

### Typical appearance
Usually appears as:
- event/listing context
- Studio
- Explore
- detailed graph views

Not always the main visible node, but must exist as truth.

---

## 3.4 Listing Node

### What it represents
An operational sellable/publishable offering.

Examples:
- service listing
- product listing
- bundle listing
- B2B listing
- private booking listing
- public open booking listing
- channel-only promotion
- VIP drop
- flash opportunity listing

### Category
**Domain entity**

### Minimum data
- listing_id
- title
- media
- type
- price / promo
- visibility scope
- channel/audience scope
- service/product composition
- capacity / minToConfirm if applicable
- place/resource context

### Typical relationships
- published by profile/org
- includes service(s)
- includes product(s)
- booked by booking
- visible to channel/followers/public
- tied to place/resource
- may create slots/events

### Typical appearance
- Home
- Explore
- Studio
- Profile
- sometimes as a graph node in full operational views

### Important note
Listings are first-class commercial graph objects.
They should not be treated only as metadata hidden behind bookings.

---

## 3.5 Product Node

### What it represents
A sellable product or bundle component.

Examples:
- care kit
- food bag
- merch
- bonus product
- bundle product

### Category
**Domain entity**

### Minimum data
- product_id
- title
- media
- price
- availability
- relation to listing/order/event

### Typical relationships
- included in listing
- purchased through order
- associated with booking or promotion

### Typical appearance
More often:
- secondary context node
- bundle detail
- commerce-related graph branch

---

## 3.6 Booking Node

### What it represents
A reservation/commitment created through the system.

### Category
**Domain entity**

### Minimum data
- booking_id
- source_listing_id
- organizer_id / booker_id
- start/end
- booking_state
- participants summary
- payment/commitment summary
- place/resource relation

### Typical relationships
- created from listing
- creates/links to event
- occupies slot/resource/time
- has attendees
- has payment/commitment state

### Typical appearance
Often shown through event-state rendering,
but still conceptually a first-class domain node.

---

## 3.7 Event Node

### What it represents
A fixed scheduled moment.

### Category
**Domain entity**

### Minimum data
- event_id
- title
- start
- end
- duration
- status
- place/resource
- service/listing context
- participant context
- visibility/access

### Typical relationships
- belongs to booking or is created independently
- happens at place/resource
- shared with participants
- can conflict with other events
- may derive media/activity after it happens

### Typical appearance
This is one of the most important fixed temporal destination nodes.

---

## 3.8 Slot Node

### What it represents
An open availability window.

### Category
**Domain entity**

### Minimum data
- slot_id
- start
- end
- duration
- visibility
- linked listing/service
- place/resource
- urgency state

### Typical relationships
- belongs to timeline/resource/listing
- can be consumed by booking
- can become flash availability
- can be part of capacity/minToConfirm logic

### Typical appearance
A lighter opportunity node,
not a commitment node.

---

## 3.9 Place Node

### What it represents
A location where something happens.

### Category
**Domain entity**

### Minimum data
- place_id
- title
- image
- address
- timezone
- type
- owner organization

### Typical relationships
- contains resource
- hosts event
- context for listing/service
- influences timezone rendering

### Typical appearance
Can appear:
- as event identity
- as contextual sub-node
- as profile/organization-associated place

---

## 3.10 Resource Node

### What it represents
A temporally relevant asset.

Examples:
- football field
- cabinet
- room
- consultorio
- operating room
- equipment

### Category
**Domain entity**

### Minimum data
- resource_id
- title
- image/icon
- type
- owner place/org
- availability state
- capacity if relevant

### Typical relationships
- belongs to place
- used by event/booking
- has conflicts
- may have its own timeline
- may be part of multi-resource organization views

### Typical appearance
Important in Studio and deeper operational graph views.

---

## 3.11 Agreement Node

### What it represents
A real access/permission/delegation/business agreement.

### Category
**Domain entity**

### Minimum data
- agreement_id
- source_actor
- target_actor/resource/context
- scope
- visibility grant
- active/revoked state
- validity window if needed

### Typical relationships
- grants access to timeline or nodes
- allows booking or management
- binds professional ↔ organization
- binds reseller ↔ clinic
- binds parent ↔ course timeline

### Typical appearance
Usually not front-and-center in Home,
but critical in Studio and advanced graph logic.

---

## 3.12 Channel / Audience Scope Node

### What it represents
An access audience or visibility grouping.

Examples:
- VIP channel
- followers-only
- parents group
- partner channel

### Category
**Domain entity** or **domain support entity**
(depending on final modeling detail)

### Minimum data
- channel_id / audience_scope_id
- name
- visibility type
- membership
- access rules

### Typical relationships
- determines who sees listings/posts/opportunities
- linked to channels, offers, and segmentation

### Typical appearance
Often contextual rather than primary,
but important in access-aware graph slices.

---

## 3.13 Order / Commitment Node

### What it represents
A commercial/payment/commitment record that materially affects temporal behavior.

Examples:
- payment
- deposit
- stake
- refund
- no-show commitment
- reward/credit

### Category
**Domain entity**

### Minimum data
- order_id / commitment_id
- financial state
- amount
- commitment rule
- refund/reward state
- related booking/event

### Typical relationships
- linked to booking/listing/product/event
- influences attendance consequences
- may alter graph emphasis in operational views

### Typical appearance
Usually secondary or expanded context,
not always a primary graph node in Home.

---

# 4. Derived Projection Node Families

These are not primary domain truths.
They are created by the projection layer to improve graph readability and UX.

## 4.1 Shared Hub Node

### What it represents
A graph-emphasized shared moment where multiple timelines converge.

### Category
**Derived projection node**

### Why
The underlying truth may simply be:
- one event
- many participants/resources

But the graph may choose to render that event as a stronger hub.

### Minimum derived data
- source_event_id
- participant summary
- resource summary
- visibility scope
- convergence importance

### Typical appearance
- enriched event hub
- stronger than a normal event
- may attract curved joins and floating participant bubbles

---

## 4.2 Conflict Node

### What it represents
A derived collision/tension between two or more incompatible temporal commitments.

### Category
**Derived projection node**

### Why
The source truth may be:
- two overlapping events
- or a resource contention
- or a policy collision

The graph may create a conflict node/zone/cluster to help the user understand it.

### Minimum derived data
- source_event_ids / resource_ids
- conflict type
- severity
- affected actor/resource
- resolution needed

### Typical appearance
- collision zone
- alert cluster
- branch impossibility
- operational warning

---

## 4.3 Flash Opportunity Node

### What it represents
A newly opened slot/opportunity emphasized for speed and discovery.

### Category
**Derived projection node**
(built from real slot/listing/event changes)

### Minimum derived data
- source_slot_id / listing_id
- opened reason
- urgency window
- target audience
- CTA state

### Typical appearance
- Home
- quick rails
- social flash moments
- high urgency discovery

---

## 4.4 Preview Hook Node

### What it represents
A compact teaser/hook into something larger in the graph.

### Category
**Derived projection node**

### Why
Home often needs:
- not the whole thing
- but a meaningful hook

### Examples
- shared moment hint
- conflict hint
- “you have 2 connected things tonight”
- “this slot fits your week”

---

## 4.5 Aggregation / Cluster Node

### What it represents
A grouped summary of multiple nearby or related nodes.

### Category
**Derived projection node**

### Examples
- 3 nearby slots
- several old past nodes compressed
- multiple doctor timelines summarized
- several attendees grouped before expand

### Typical appearance
Useful when:
- zooming out
- compressing
- keeping the graph readable

---

# 5. Purely Visual Markers

These are not domain entities and usually not projection entities either.
They are UI markers.

## 5.1 Current User Marker / Owner Marker

### What it represents
The current user’s position / focus / “you are here” in the visible graph slice.

### Category
**Purely visual marker**

### Why
It indicates:
- current focus
- temporal progression
- user alignment with events

### Minimum UI data
- current user identity
- current time position
- maybe current branch/focus

---

## 5.2 Alignment State

### What it represents
The visual state when the moving user marker aligns with an event node.

### Category
**Purely visual marker / state**

### Why
This expresses:
- “the moment is happening”
- temporal synchronization

---

## 5.3 Focus Window Boundary

### What it represents
The idea that the user is only seeing a local region of a larger graph.

### Category
**Purely visual / interaction-level concept**

### Why
The graph is larger than what is shown.

---

## 5.4 Ghosted Past State

### What it represents
Past nodes visually faded / desaturated.

### Category
**Purely visual state**
(applied to many node types)

### Why
It helps communicate:
- past vs active relevance
- reduced emphasis

---

## 5.5 Selection / Hover / Expand State

### What it represents
Focused node interaction state.

### Category
**Purely visual state**

### Why
Different details may appear at rest vs focus.

---

# 6. Node taxonomy matrix

## Domain entity nodes
- Profile
- Organization
- Service
- Listing
- Product
- Booking
- Event
- Slot
- Place
- Resource
- Agreement
- Channel/Audience
- Order/Commitment

## Derived projection nodes
- SharedHub
- Conflict
- FlashOpportunity
- PreviewHook
- Aggregation/Cluster

## Purely visual markers
- CurrentUserMarker
- AlignmentState
- FocusWindow
- GhostedPastState
- Selection/ExpandState

---

# 7. Where nodes typically appear

## Home
Mostly:
- PreviewHook
- FlashOpportunity
- current user context
- compact Event/Slot projections
- social/listing hooks

## Explore
Mostly:
- Listing
- Profile
- Organization
- Place
- Event hints
- service context

## Studio
Mostly:
- Event
- Slot
- Listing
- Resource
- Place
- Booking
- Agreement context
- SharedHub / Conflict projections
- organization/multi-track slices

## Profile
Mostly:
- Profile
- Listing
- Event previews
- activity/media nodes
- social + offer nodes

---

# 8. Working rules for future design work

Whenever a new node appears in conversation, ask:

1. Is this a **real domain entity**, a **derived projection**, or a **purely visual marker**?
2. Does it need lifecycle, ownership, and persistence?
3. Is it visible at Home preview, Studio full view, or both?
4. Is it media-first / identity-first enough?
5. Is it a fixed temporal destination, an opportunity, a relationship emphasis, or a UI marker?

---

# 9. Immediate next use

This document should be used together with:
- timeline-graph-projection-model-v1.md
- temporal-graph-surface-inventory-v1.md

And then applied case-by-case when defining:
- event with no invitation
- invited event
- shared hub
- slot
- flash opportunity
- listing-backed booking
- bundle booking
- resource-bound event
- conflict
- org/hospital/course/multi-timeline slices

---

# 10. Closing principle

> A clear temporal graph UX depends on a clear node taxonomy. If we do not distinguish domain entities, derived graph projections, and purely visual markers, the system will become conceptually muddy very quickly.
