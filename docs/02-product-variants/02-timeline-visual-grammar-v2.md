# Timeline Visual Grammar v2 — VyteMerge

## Estado

Draft v2

---

# 1. Purpose

This document defines the **visual language** of VyteMerge Timeline.

It is not a backend specification, a data model, or a feature list.
It describes how Timeline **looks, behaves, and communicates** as a visual system.

Its goal is to establish a shared understanding of:

- what elements compose a Timeline visually
- how those elements relate to each other
- how time, identity, and convergence are expressed graphically
- how sparse and dense time should be rendered
- how private timelines can intersect without exposing everything
- how Timeline fragments should appear in Home, Feed, and Explore
- how the product can evolve from timeline → graph without losing clarity

This document exists because Timeline is the most visually distinctive part of VyteMerge.
If Timeline looks like a generic calendar, event list, or planning tool, the product loses its identity.

---

# 2. Core visual principles

## 2.1 Timeline is a line of life

A Timeline is a continuous line that represents the passage of time for an entity.

It is not:
- a grid
- a table
- a spreadsheet of hours
- a generic event list

It is a **line of life/time** along which things happen.

The line is the spine.
The moments that matter are expressed as **nodes**.

## 2.2 Nodes are the protagonist

The primary visual unit of Timeline is the **node**.

A node represents a meaningful temporal moment:
- a slot
- an event
- a shared event
- a conflict
- a resource occupation
- a convergence between lines

Nodes are:
- compact
- glanceable
- identity-rich
- time-aware
- interactive

The node is what the user reads first.
The line only provides continuity.

## 2.3 Lines are support, not protagonists

Lines exist to communicate:
- continuity
- sequence
- relation
- convergence

They should be:
- visually subdued
- thinner than nodes
- consistent
- present enough to orient, but never dominant

A user should remember the moments, not the line.

## 2.4 Empty time compresses

VyteMerge does not render time literally.

Empty time should not waste visual space.

If a user has:
- one event in February
- one in June
- one in August

the Timeline should not draw a giant empty line just to represent the gap.

Instead:
- the empty time compresses
- the nodes remain ordered
- the user sees "my next relevant moments"

Timeline is **semantic time**, not literal clock-space.

## 2.5 Identity is always present

Every meaningful temporal moment carries identity.

That identity may belong to:
- the owner of the timeline
- another participant
- a resource
- a place
- an activity
- a commercial offering
- a shared context

Identity is expressed through:
- avatar
- logo
- image
- icon
- color
- connection pattern

## 2.6 Timeline is primary, graph is emergent

VyteMerge should not present a full graph by default.

The primary reading is always:
- timeline
- sequence
- next moments

The graph emerges only where there are:
- convergences
- shared events
- related timelines
- resource connections
- conflicts

The system is timeline-first, graph-capable.

## 2.7 Timeline must scale from simple life to connected life

The same visual grammar must work for:
- a single user with 3 isolated events
- a provider with bookings
- a family timeline
- a hospital with multiple doctors and shared rooms
- a band with members converging on a concert
- a resource like a football field or meeting room

The grammar must be simple at rest and expressive when expanded.

---

# 3. Sparse vs dense timeline

## 3.1 Sparse timeline

A sparse timeline has few meaningful moments separated by long periods of empty time.

Example:
- dentist in February
- meeting in June
- trip in August

Visual behavior:
- show the 3 nodes in order
- compress empty time
- do not draw a giant line through empty months
- preserve a feeling of sequence, not emptiness

The sparse timeline communicates:
> "These are your next relevant moments."

## 3.2 Dense timeline

A dense timeline has many moments close together.

Visual behavior:
- nodes become closer
- continuity becomes more visible
- related moments can cluster
- conflicts and convergences become legible

The dense timeline communicates:
> "This part of your life is busy, active, and coordinated."

## 3.3 Transition between sparse and dense

The same grammar adapts fluidly:
- sparse areas → node-first, compressed line
- dense areas → line and relation become more visible

There is no mode switch between "simple" and "advanced."
The density reveals the complexity naturally.

---

# 4. The node as temporal identity marker

## 4.1 Core idea

A node should behave like a **compact temporal identity marker**.

The center represents:
- who / what / which context

The edge represents:
- when

This is the foundation of the node language.

## 4.2 Anatomy of a node

### Center
The center may contain:
- avatar
- logo
- image
- icon
- contextual identity

It does not need to always represent the owner of the timeline.
It may represent:
- the patient in a doctor's schedule
- the football field for a match
- the meeting room
- the concert
- the salon room
- the dog-grooming task
- the client
- the shared activity

### Edge
The edge encodes time:
- minimal analog-clock feeling
- hour marker
- minute marker
- small number for users who do not parse analog clocks easily

The edge is not decorative.
It is how the node becomes temporal.

### Secondary indicators
Optional small signals:
- conflict
- shared
- pending
- confirmed
- resource
- number of participants

## 4.3 Small digital hint

In addition to the analog feel, a tiny numeric hint may exist:
- `9`
- `14`
- `18`
or equivalent compact notation

This helps users who do not read an analog abstraction comfortably.

The digital hint should remain:
- small
- secondary
- supportive

The clock idea remains primary.

---

# 5. The moving avatar ("muñequito")

## 5.1 Purpose

The owner of a timeline can be represented by a small moving avatar marker on or near the line.

This marker communicates:
- where "now" is
- where the owner is in relation to upcoming moments
- that time is flowing

## 5.2 Meaning

The moving avatar is not an event.
It is the **current temporal position** of the owner.

It helps the user understand:
- what is next
- what was before
- where they are in the flow

## 5.3 In related/shared timelines

A moving avatar can also represent:
- the doctor on the doctor's line
- the musician on the musician's line
- the family member on their line

Multiple moving avatars may converge toward a shared node.

## 5.4 Rule

The moving avatar must remain lightweight.
It should never overpower the nodes.

---

# 6. Slot vs event

This distinction must be visually explicit and immediate.

## 6.1 Slot

A slot is **possible time**.

It means:
- available
- open
- reservable
- projected
- not yet committed

### Visual treatment
- hollow / lighter / thinner
- low visual weight
- no participant identity in the center
- no strong connection markers
- reads as "opportunity"

### Interpretation
> "This time could be used."

## 6.2 Event

An event is **committed time**.

It means:
- something real is scheduled
- time is occupied
- one or more entities are already involved

### Visual treatment
- stronger visual weight
- identity present in the center
- more defined time edge
- may show relation markers
- reads as "this is happening"

### Interpretation
> "This moment is real."

## 6.3 Shared event

A shared event is committed time involving more than one temporal entity.

Examples:
- football match
- medical appointment
- meeting in a room
- concert with multiple band members
- family dinner
- dog-grooming task assigned during another personal plan

### Visual treatment
- main event node
- one or more converging actors/resources
- relation visible
- details revealed on expand/hover/tap

### Interpretation
> "This is where multiple timelines meet."

---

# 7. Event node vs slot node — strong visual rule

To simplify the product language:

## Slot
- empty / hollow / possibility
- no converging actors
- no participant ownership visually asserted
- lighter

## Event
- occupied / committed / connected
- can carry context identity
- can attract or connect multiple timelines
- stronger

### Closing distinction
**Slot = available time  
Event = connected time**

That is the core rule.

---

# 8. Personal events vs shared events

## 8.1 Personal event

A personal event is meaningful only on the owner's line.

Examples:
- dentist
- gym
- private reminder
- solo meeting
- personal errand

### Visual behavior
- one main node
- no additional convergence shown
- identity can be the activity/provider/resource itself

## 8.2 Shared event

A shared event belongs to multiple lines at once.

Examples:
- football match
- medical appointment
- meeting room booking
- concert
- family school event

### Visual behavior
- main event node
- approach or convergence from other relevant actors/resources
- additional participants may stay hidden until interaction if privacy demands it

### Important rule
The main event node does not have to represent the owner of the line.

In a doctor's line:
- the node may represent the patient or the appointment context

In a football case:
- the node may represent the field or the match

In a room schedule:
- the node may represent the room or the meeting

---

# 9. Resource nodes

A resource is a full temporal entity.

Examples:
- football field
- meeting room
- consultation room
- equipment
- stage
- plaza
- vehicle

## 9.1 Resource line

A resource can have its own line.

That means:
- a room can have a timeline
- a field can have a timeline
- a stage can have a timeline

## 9.2 Resource node

A resource node may be represented by:
- logo
- contextual image
- icon
- place identity

The resource is not secondary.
It is often the center of a convergence.

## 9.3 Examples

### Football field
The event node may be the field/match, not the booking user.

### Meeting room
The room may attract multiple employee lines.

### Hospital room
The room becomes a shared temporal resource with conflicts and occupancy.

---

# 10. Convergences and merge points

## 10.1 What convergence means

Convergence is when multiple temporal entities point to the same committed moment.

This may happen because:
- multiple people join the same event
- a person and a resource are linked
- a person and a place coincide
- a task, listing, or agreement creates a shared moment

## 10.2 Visual treatment

Convergence should be visible, but not overwhelming.

Base state:
- subtle relation indicators
- small approach markers
- line support

Expanded state:
- visible connecting lines
- participant reveals
- merge point clarity

## 10.3 Privacy-aware convergence

A user may see that something converges without seeing the full private timeline of the other party.

Example:
- doctor sees appointment shared with a patient
- patient exists as a convergence
- the doctor does not see the patient's entire life
- the shared moment is visible, the private rest remains hidden

This is essential to the product.

---

# 11. Conflicts

## 11.1 What is a conflict

A conflict is when two or more temporal claims overlap in a way that requires a decision.

Examples:
- football match overlaps with dog-grooming assignment
- meeting overlaps with family event
- room is occupied by another booking
- provider gets assigned a task during a personal commitment

## 11.2 Visual treatment

A conflict should not just be text.

A conflict should be a visual event:
- warning accent
- overlapping/converging structure
- action affordance
- clear sense of collision

## 11.3 Meaning

A conflict is not a bug.
It is a signal that the system has found a meaningful decision.

VyteMerge is not just a planner.
It is a system for **surfacing and resolving temporal collisions**.

---

# 12. Home timeline glimpse

## 12.1 Purpose

Home should not show the full Timeline.
Home should show a **condensed personal temporal glimpse**.

But not as a boring list.

It should say:
- this is your line
- this is what is coming
- this is what is connecting to your life

## 12.2 Meaning of Home

Home is not a pure feed.
Home is:
# **my line + the things that connect to it**

That includes:
- next personal/shared moments
- opportunities that fit
- things near my context
- items I may add to my life
- moments already converging with my timelines

## 12.3 Visual treatment

The Home glimpse may show:
- a compact line
- 2–4 next meaningful nodes
- moving avatar / current position
- one or two things connecting in
- CTA to open full Timeline

It should feel like:
- a watch glance
- a life glance
- not a mini calendar

---

# 13. Feed and Explore graph fragments

## 13.1 Feed

Feed may show:
- graph fragments
- shared moments
- timeline slices
- public/shared activity moments

Feed is not only posts.
It can contain public temporal fragments.

## 13.2 Explore

Explore should remain simpler and search-first.

But Explore can still show:
- temporal hints
- availability fragments
- event snippets
- public graph moments

## 13.3 Rule

Feed/Explore do not replace the Timeline.
They expose fragments of the temporal network.

Timeline remains the primary surface for:
- acting on time
- seeing one's line
- understanding convergences deeply

---

# 14. View model evolution

## 14.1 V1 — Simple temporal line

- line of life/time
- sparse compression
- compact nodes
- moving avatar
- slot vs event
- personal/shared event distinction

## 14.2 V2 — Related lines

- toggles for related timelines
- family
- agreements
- shared resources
- institutional views

## 14.3 V3 — Graph-capable timeline

- convergences
- merges
- place/resource/actor crossings
- graph fragments in feed/home
- richer conflict resolution

## 14.4 Important warning

The product may become graph-capable.
It should not start by showing a giant graph.

The user should enter through:
- line
- node
- next moments
- local clarity

Graph depth should be revealed progressively.

---

# 15. Example readings

## 15.1 Sparse personal timeline
The user has events in February, June, and August.
The UI should show:
- 3 ordered nodes
- compressed time between them
- no giant empty line

## 15.2 Doctor appointment
The doctor's line shows a committed node.
The node may represent:
- the patient
- the appointment context
- the clinic/resource

The patient may appear as the converging identity, without exposing the patient's full timeline.

## 15.3 Football match
The booking user reserves a football field sold through another user's listing.
The event node may represent:
- the field
- the match
- the field's identity

The invited friends appear as converging participants, visible only when appropriate.

## 15.4 Meeting room
Multiple doctors or staff may converge into a room resource node.
The room can have its own line.
The people can have their own lines.
The event is the intersection.

## 15.5 Conflict case
A player has a football match.
At the same time, via agreement, a veterinary task is assigned.
The conflict emerges visually.
The user can resolve it by canceling attendance or declining/reassigning work.

This is a core VyteMerge behavior.

---

# 16. Design decisions

## Decision 1
**Nodes are the protagonist, not the line.**

## Decision 2
**Empty time compresses.**

## Decision 3
**A node is a temporal identity marker.**
Center = identity, edge = time.

## Decision 4
**Slot ≠ event.**
Slot is possible time; event is connected time.

## Decision 5
**The main node does not always represent the timeline owner.**
It represents the most meaningful identity of the moment.

## Decision 6
**Convergence is the visual signature of VyteMerge.**

## Decision 7
**Timeline is primary, graph is emergent.**

## Decision 8
**Home shows my line and what connects to it.**

## Decision 9
**Feed and Explore may show graph fragments, not just flat content.**

## Decision 10
**Privacy-aware intersections are first-class.**
A shared moment may be visible without exposing the whole other line.

---

# 17. Closing principle

> **The Timeline is not a calendar you look at.
> It is a living line of time made of compact temporal identity markers, where moments, resources, people, and shared contexts can converge, conflict, and reveal how life connects.**
