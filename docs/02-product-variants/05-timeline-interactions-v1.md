# Timeline Interactions v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **interaction model** of VyteMerge Timeline.

It explains:
- what the user can do with timeline nodes
- how interactions differ by surface
- what actions belong to Home, Feed, Explore, and Timeline
- how the product moves from passive reading to active time management
- how shared moments, conflicts, and related timelines should be explored
- how the UI can stay simple while supporting a graph-capable future

This is a product and UX document.
It is not a backend or implementation spec.

Its goal is to make Timeline actionable without turning it into:
- a generic calendar tool
- a dashboard full of buttons
- a graph browser that overwhelms the user

---

# 2. Core interaction principle

## Time is not just observed — it is acted upon

The Timeline is not a passive artifact.

The user must be able to:
- inspect moments
- understand what is connecting to their life
- add things to their time
- resolve conflicts
- explore shared moments
- move from glimpse → decision → action

However:
- the base interaction must stay simple
- complexity must appear progressively
- not every surface should expose full temporal power

---

# 3. Interaction layers

VyteMerge Timeline interactions should be understood in layers:

## Layer 1 — Glance
The user sees:
- what is next
- what is connected
- whether something deserves attention

No deep action yet.

## Layer 2 — Inspect
The user taps or expands a node to understand:
- what it is
- who is involved
- where it happens
- whether it affects them

## Layer 3 — Act
The user takes a meaningful temporal action:
- add to timeline
- reserve
- accept
- reject
- cancel
- resolve conflict
- open full view

## Layer 4 — Navigate relation
The user explores:
- related timelines
- shared moments
- resource contexts
- conflicts
- allowed graph fragments

This layered model should apply consistently across surfaces.

---

# 4. Interaction rules by surface

## 4.1 Home

### Purpose of interaction
Home should help the user:
- notice what is coming
- see what is connecting to their life
- act on discovery
- jump into full Timeline when needed

### Allowed interaction types
- tap compact timeline glimpse
- tap suggested item that connects to the user's time
- add an opportunity to timeline
- reserve from a discovery item
- follow a node/timeline/actor if supported
- open full Timeline

### Not allowed / discouraged
- deep temporal editing
- full graph navigation
- complex branch management
- detailed conflict resolution UI

### Home rule
Home is where the user notices and begins.
It is not where they manage everything.

---

## 4.2 Feed

### Purpose of interaction
Feed should let the user:
- encounter graph fragments
- inspect public/shared moments
- decide whether something matters to them
- move a relevant moment into their life

### Allowed interaction types
- tap node fragment
- expand a shared/public moment
- reserve if the fragment is bookable
- save / add to timeline
- follow actor / follow timeline / follow context (future)
- open related profile, listing, or full Timeline

### Not allowed / discouraged
- full timeline editing
- branch toggles
- operational conflict management
- heavy multi-line manipulation

### Feed rule
Feed is about temporal discovery and engagement, not management.

---

## 4.3 Explore

### Purpose of interaction
Explore should let the user:
- search first
- inspect matching moments/entities
- refine what they are looking for
- bring a relevant thing into their timeline

### Allowed interaction types
- type search
- refine with tags/filters
- tap result
- inspect temporal hint
- add to timeline
- reserve
- open deeper detail

### Not allowed / discouraged
- rigid upfront mode selection
- complex graph visualization
- heavy branch toggling
- advanced conflict workflows

### Explore rule
Explore is search-first, not graph-first.

---

## 4.4 Timeline tab

### Purpose of interaction
Timeline is where the user:
- understands their temporal life
- inspects shared moments
- sees conflicts clearly
- resolves or changes what happens
- moves from "notice" to "decide"

### Allowed interaction types
- tap node
- expand node
- inspect convergence
- inspect related lines
- resolve conflict
- cancel / reschedule / accept / reject when allowed
- toggle related timelines
- jump to merged view
- jump to source detail (profile, listing, resource, agreement context)

### Timeline rule
Timeline is the primary action surface for time.

---

# 5. Base interactions on nodes

## 5.1 Tap

### Meaning
The default interaction.

### What it should do
Tap on a node should reveal:
- what this moment is
- who or what it relates to
- the exact time
- whether it is personal/shared/conflicting
- the most relevant next action

### Rule
Tap should never explode into a huge unrelated experience.
It should feel local, contextual, and temporal.

---

## 5.2 Hover (desktop only, optional)

### Meaning
A lightweight preview.

### What it may reveal
- time
- label
- participant count
- resource name
- small conflict hint
- whether the node is expandable

### Rule
Hover is enhancement, not dependency.
The product must work fully on touch devices.

---

## 5.3 Expand

### Meaning
Reveal richer context.

### Expand should show
- title / contextual identity
- participant previews
- place/resource
- time
- relation to other visible lines
- CTA block
- privacy-aware details

### Rule
Expand should preserve the local context of the timeline.
Do not push the user away too fast unless they intentionally choose to drill deeper.

---

## 5.4 Long press / advanced gesture (future)

### Possible future meanings
- quick add to timeline
- quick reschedule
- drag toward another area
- reveal mini action sheet

### Rule
Do not depend on gesture novelty in v1.
Long press is optional future depth.

---

# 6. Action families

## 6.1 Add to Timeline

### Meaning
A discovered thing becomes part of my temporal life.

### Typical sources
- Home
- Feed
- Explore
- Profile fragments
- public/shared event fragment

### Result
The system creates:
- a personal event
- a tentative event
- a saved temporal intention
- or a booking flow start

### Rule
"Add to Timeline" is one of the most important actions in VyteMerge.
It is the bridge from discovery → life.

---

## 6.2 Reserve / Book

### Meaning
I commit to time offered by another temporal entity.

### Typical sources
- listing-backed moments
- offer cards
- public schedule fragments
- resource availability

### Rule
Booking is stronger than "add to timeline."
It is not just a saved interest.
It is a temporal commitment.

---

## 6.3 Save / Keep for later

### Meaning
This matters, but I am not committing yet.

### Typical sources
- Feed
- Explore
- Home discovery blocks

### Rule
Save is softer than add-to-timeline.
It preserves interest without temporal commitment.

---

## 6.4 Follow actor / follow timeline / follow context

### Meaning
I want this line, actor, or context to remain relevant to me.

### Possible objects of following
- person
- business
- timeline
- place
- resource
- recurring context (future)

### Rule
This is a future-expansion action.
Do not over-design it before the base Timeline model is stable.

---

## 6.5 Resolve conflict

### Meaning
Two temporal claims collide and I must decide.

### Typical actions
- cancel one
- reschedule one
- override
- open related timeline
- inspect who/what is involved

### Rule
Conflict resolution should feel easy and local.
The system identifies; the user decides.

---

## 6.6 Open source detail

### Meaning
I want to understand the source context of this moment.

### Possible destinations
- profile
- listing
- place
- resource
- agreement context
- full event detail

### Rule
Source detail should be one tap away from an expanded node.

---

# 7. Home-specific timeline interactions

## 7.1 Home timeline glimpse

### Allowed actions
- tap the glimpse → open Timeline
- tap a specific visible mini-node → open relevant moment
- tap a connected opportunity → add to timeline / reserve / inspect

### Rule
The Home glimpse must remain lightweight.
It should not become a mini-management panel.

## 7.2 "Connected things" block

Because Home means:
# my line + things that connect to it

The user should be able to:
- inspect the connection
- add it to their life
- reserve it
- dismiss it
- open full detail

### Example
- "Free chess class" fits an open slot in my week
- action:
  - add to timeline
  - reserve
  - see why it fits

---

# 8. Feed-specific timeline interactions

## 8.1 Public/shared graph fragment

A fragment in Feed may represent:
- a public event
- a shared moment
- a temporal opportunity
- a convergence around a place/resource

### Allowed actions
- tap fragment
- expand context
- reserve
- save
- add to timeline
- open source detail

### Rule
Feed is where graph fragments create desire and curiosity.
The heavy action moves to Timeline or booking flow.

---

# 9. Explore-specific timeline interactions

## 9.1 Search-first

The primary interaction in Explore is:
- search
- then refine
- then inspect
- then act

### Rule
Do not force users into upfront rigid segmentation.

## 9.2 Result interactions

Each result may allow:
- quick temporal hint reading
- open detail
- add to timeline
- reserve
- save

### Rule
Explore should feel natural, not like a control panel.

---

# 10. Timeline-specific advanced interactions

## 10.1 Toggle related timelines

### Purpose
Reveal lines that matter to me:
- family
- work
- shared resources
- delegated timelines
- institutional contexts

### Rule
The user should be able to turn relation on/off cleanly.
Do not dump all related timelines at once.

---

## 10.2 Inspect convergence

### Meaning
See where lines meet.

### User should be able to
- tap the shared node
- see who/what converges there
- understand whether the relation is:
  - personal
  - institutional
  - commercial
  - resource-based
  - conflict-based

---

## 10.3 Enter merged view

### Purpose
Zoom from "my line" to "this shared temporal situation"

### Trigger
- tap merge indicator
- toggle merged mode
- inspect a shared event deeply

### Rule
Merged view must remain event-centric or context-centric.
Do not become abstract graph spaghetti.

---

# 11. Privacy-aware interactions

## 11.1 BusyOnly

If another line is shared only as BusyOnly:
- the user may see that a moment exists
- may see that it converges with theirs
- may NOT inspect the narrative details

### Interaction
tap may show:
- "Ocupado"
- approximate time context
- no deeper reveal

## 11.2 BusyOnlyDetails

Tap may reveal:
- limited metadata
- tags
- type
- partial context

But still not the full private story.

## 11.3 Full visibility

If allowed by agreement:
- user can inspect full details
- user may act if permissions allow

### Rule
Interactivity must respect visibility, not bypass it.

---

# 12. Conflict interaction model

## 12.1 Passive conflict signal
The user notices:
- warning color
- overlap
- icon
- merge/collision structure

## 12.2 Active inspection
Tap conflict →
- what is conflicting
- which lines/resources are involved
- why it matters

## 12.3 Resolution
User can choose:
- cancel attendance
- decline assignment
- reschedule
- keep both (override)
- inspect source detail before deciding

### Example
Football match vs veterinary assignment:
- conflict signal
- tap
- see both claims on time
- choose what to do

---

# 13. Suggested interaction hierarchy

## For v1
Prioritize:
1. Tap
2. Expand
3. Add to timeline
4. Reserve
5. Resolve conflict
6. Open source detail

## Defer
- drag-and-drop
- long-press advanced menu
- full graph manipulation
- follow node/timeline as a core mechanic
- complex multi-select or scheduling gestures

---

# 14. Home / Feed / Explore / Timeline interaction summary

| Surface | Primary interaction | Secondary interaction | Avoid |
|--------|----------------------|-----------------------|-------|
| Home | Notice + add to life | Open full Timeline | Management overload |
| Feed | Inspect graph fragments | Reserve / save / add | Heavy editing |
| Explore | Search + inspect + act | Refine progressively | Rigid dashboards |
| Timeline | Understand + decide | Resolve / compare / merge | Generic calendar UX |

---

# 15. V1 / V2 / future

## V1
- tap
- expand
- reserve
- add to timeline
- simple conflict resolution
- open source detail

## V2
- related timeline toggles
- merged view entry
- richer participant reveal
- stronger Home/Feed graph fragments
- follow line/context

## Future
- gesture-based drag into life
- long-press quick scheduling
- richer graph navigation
- AI-assisted temporal recommendations
- timeline composition and scenario planning

---

# 16. Interaction design decisions

## Decision 1
**Tap is the default temporal interaction.**

## Decision 2
**Home is for noticing and beginning, not managing.**

## Decision 3
**Timeline is the primary action surface for time.**

## Decision 4
**Add to Timeline is a first-class action.**

## Decision 5
**Conflict is revealed visually, resolved intentionally.**

## Decision 6
**Explore stays search-first, not graph-first.**

## Decision 7
**Intersections can be visible without exposing the whole private line.**

## Decision 8
**Complexity appears progressively through expand, toggle, and merged view.**

---

# 17. Closing principle

> **VyteMerge Timeline interactions should let the user move naturally from noticing time, to understanding time, to acting on time — without ever making the product feel like a generic planner or a chaotic graph tool.**
