# Timeline Graph, Projection Layer, and Presentation Model v1

## Status
Draft v1

## Purpose
This document captures the current thinking around:
- the full timeline graph concept
- why the graph should not be the first UI layer to solve
- the idea of a second-order / higher-order graph projection
- how the product can evolve from operational truth toward graph expression
- how Home, Studio, and future graph views relate to the same underlying temporal model

This document is intentionally conceptual and architectural.
It is not a final technical implementation spec.

---

# 1. Core idea

VyteMerge is not just a timeline product.

It is a product where:
- people
- organizations
- services
- listings
- resources
- places
- bookings
- events
- agreements
- audiences
- opportunities

all connect through time.

The deepest conceptual model is therefore not “a line”.
It is better understood as:

# **a temporal graph**

---

# 2. What “temporal graph” means here

A temporal graph is a graph where:
- nodes represent meaningful business/temporal entities
- edges represent temporal, operational, social, commercial, or access relationships
- time affects:
  - visibility
  - relevance
  - progression
  - conflicts
  - convergence
  - focus

It is not merely:
- a calendar
- a list of events
- a schedule
- a graph browser

It is the deeper structural truth behind the product.

---

# 3. The timeline is singular in the model

There is **one real timeline / temporal graph** in the system.

That means:
- the graph is singular in the domain model
- it is not “one timeline for Home” and “another timeline for Studio”
- it is the same underlying reality

What changes is:
- how much of that graph is visible
- which slice is rendered
- what context the user is in
- what permissions they have
- what the UI is trying to help them do

---

# 4. One timeline, many views

The same timeline/graph can appear differently depending on surface.

## Home
Home shows:
- a compact preview
- a local focus
- hooks into shared moments
- next relevant moments
- flash opportunities
- conflict hints
- “things connected to my life/time”

Home does **not** show the full graph.

## Studio
Studio shows:
- the full operational view
- the richer scheduling layer
- services
- listings
- resources
- deeper calendar/timeline logic
- the most complete practical slice of the graph

## Future graph/depth views
These may later show:
- stronger branching
- shared hubs
- conflict clusters
- multi-resource convergence
- local graph navigation
- organization-level multi-timeline slices

---

# 5. Why not solve the graph UI first

A key decision was reached:

## The graph should not be the first UI layer to solve.

Why:
- because the graph depends on solid domain truth
- because visual brilliance without operational clarity becomes fake
- because the system still needs its core product experience defined
- because services/listings/bookings/resources/access must be understood first

In other words:

## first:
- product experience
- services
- listings
- studio
- explore
- profile
- operational scheduling
- calendar/timeline simple views

## later:
- graph projection
- graph-first UI
- advanced visual convergence / conflict rendering

---

# 6. Operational-first, graph-later

This is the current strategic direction.

## First layer
Build a complete and usable product with:
- Home
- Explore
- Studio
- Profile
- services
- listings
- resources
- bookings
- operational calendar/timeline
- audience/access logic
- social/commercial feed

## Second layer
Once that foundation is solid:
- project the same underlying data into graph-oriented views
- show richer convergence
- show shared hubs
- show conflict geometry
- show local graph focus windows
- elevate the same data to a higher-order visual language

So the graph is not abandoned.
It is postponed to a better layer.

---

# 7. Source of truth vs projection

One of the most important ideas discussed is:

# **The graph does not need to be built from scratch on every request.**

Instead, the system can have:

## A. transactional source of truth
The operational/business model:
- profiles
- organizations
- services
- listings
- products
- bookings
- events
- slots
- places
- resources
- agreements
- orders
- audience/channel logic

## B. projection layer
A programmatic layer that builds graph-oriented views from that truth:
- graph nodes
- graph edges
- Home preview slices
- Studio full timeline slices
- shared hub derivations
- conflict derivations
- future graph-specific read models

This means the graph can be:
- maintained
- updated incrementally
- optimized for reading
- stable for UI consumption

---

# 8. Projection instead of ad hoc runtime assembly

The current recommendation is:

## do not calculate the entire graph ad hoc in the UI or from many raw tables on every request.

Instead:
- maintain a projection
- update it when domain changes happen
- query that projection for specific surfaces

This is useful because:
- it is faster
- it is easier to reason about
- it separates business truth from rendering truth
- it lets different surfaces consume different graph slices
- it allows evolution toward richer graph UIs later

---

# 9. What events would feed the projection

At a conceptual level, the projection would react to domain changes such as:

- service created/updated
- listing published/unpublished
- booking created
- booking confirmed
- booking cancelled
- slot opened
- slot released
- slot consumed
- event created
- event rescheduled
- event cancelled
- attendee invited
- attendee joined
- attendee declined
- resource reserved
- resource released
- agreement granted
- agreement revoked
- audience/channel membership changed
- payment/commitment state changed
- flash availability created

These changes would update:
- visible nodes
- visible edges
- local focus slices
- shared hubs
- conflicts
- Home hooks
- Studio views

---

# 10. Real entity vs derived projection

A key distinction also emerged:

## some nodes are real domain entities
Examples:
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
- Order

These have:
- lifecycle
- ownership
- rules
- persistence
- auditability

## some nodes are derived graph concepts
Examples:
- shared hub
- conflict node
- graph slice
- local focus window
- flash opportunity projection
- aggregated preview hooks

These may not need to exist as source-of-truth entities.
They can be projections or read-model constructs.

## some markers are purely visual
Examples:
- current user marker
- alignment state
- zoom/focus affordances
- present position indicators

These belong to rendering, not domain persistence.

---

# 11. Nature of the graph UI

The graph UI is not intended to become:
- a raw Neo4j browser
- a developer graph explorer
- a technical analytics tool
- a giant fully expanded network

The graph UI should be:
- local
- meaningful
- context-aware
- elegant
- human-readable
- tied to actual business/useful product states

This leads to the concept of:

# **focus windows**

The full graph exists,
but the user normally sees:
- a local slice
- a practical zoom level
- only the relevant nearby graph region

---

# 12. Home as graph preview, not graph browser

Home should not attempt to render the full graph.

Instead, Home can show:
- a compact temporal preview
- a few meaningful hooks into shared moments
- flash opportunities
- upcoming moments
- a minimal graph-aware rail or context panel
- hints that more exists underneath

Home is a:
# **graph-aware social/product preview**

not the graph itself.

---

# 13. Studio as the deepest practical view

Studio is currently the most natural place for the deeper full timeline.

Why:
- services live there
- listings live there
- resources live there
- slots and bookings make sense there
- operational time management belongs there

So the current conceptual direction is:

## Home = compact graph-aware preview
## Studio = full operational timeline / richest practical graph slice

This is more coherent than forcing a separate top-level Timeline tab too early.

---

# 14. Timeline graph as a future elevation layer

The timeline graph remains part of the vision.

But it now sits as:
- a future elevation of the operational foundation
- not the first thing to force visually

In other words:

## the graph is still the truth of the vision
but not the first deliverable of the interface

This makes it:
- more feasible
- less fake
- easier to ground
- better supported by real data later

---

# 15. Relationship to visual design

This decision also affects visual design.

Because if the graph is not the first UI problem to solve,
then:
- Home can focus on feed/social/commercial energy
- Studio can focus on operational coherence
- Explore can focus on search/discovery
- Profile can focus on identity and activity

And the graph can later emerge from:
- richer data
- clearer relationships
- stronger service/listing/resource models
- more grounded operational states

---

# 16. Why this is useful

This approach helps avoid:
- over-designing a graph too early
- inventing visual complexity before the model is stable
- building beautiful but unsupported graph states
- mixing operational and graph concerns in the wrong order

It also helps preserve:
- product clarity
- technical sanity
- UX evolution path
- backend evolution path

---

# 17. Current working thesis

The current working thesis is:

1. Build the complete product experience first
   - social
   - service
   - listing
   - studio
   - explore
   - profile
   - calendar / operational scheduling
   - feed
   - preview rails / hooks

2. Keep the graph concept documented
   - temporal graph
   - one timeline, many views
   - projection layer
   - focus windows
   - shared hubs
   - conflict nodes

3. Later project the operational truth upward
   into:
   - graph-aware timeline views
   - richer convergence displays
   - multi-resource / multi-timeline slices
   - local graph navigation

---

# 18. Architecture direction (high-level)

Recommended conceptual stack:

## Layer 1 — Domain / transactional truth
Operational entities and workflows.

## Layer 2 — Projection layer
Programmatically maintained graph-oriented read models.

## Layer 3 — Surface views
- Home preview
- Studio operational timeline
- Explore hints
- Profile hints
- future graph view(s)

---

# 19. Immediate implication

The immediate implication is:

## stop forcing deep graph design right now
and instead:
- continue shaping the product experience
- continue building the operational model
- keep documenting graph assumptions
- preserve the graph as a second-order projection target

---

# 20. Summary

The current decision is not:
- “forget the graph”

It is:
# **build the product foundation first, then elevate it into a graph projection**

This means:
- the graph remains central to the long-term vision
- the UI does not need to fully express it yet
- a projection layer can later make that graph fast, consistent, and useful
- Home and Studio are different views over the same underlying temporal reality

---

# 21. Closing principle

> The temporal graph is the deeper structural truth of VyteMerge, but the product should first be built as a coherent operational and social system. The graph should emerge as a maintained projection layer over that reality, not as the first UI problem to solve.
