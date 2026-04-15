# Timeline View Modes v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **view modes** of VyteMerge Timeline.

It explains:
- how the Timeline should be presented at different levels of depth
- how the product moves from simple timeline to connected timeline
- how the graph nature of the system appears progressively
- what the user sees in Home versus in Timeline
- how related timelines, shared events, resources, and conflicts become visible

This is a product/UX document.
It is not a backend specification.

Its purpose is to ensure that Timeline evolves in a controlled way:
- simple first
- richer when needed
- graph-capable without becoming visually chaotic

---

# 2. Core principle

## Timeline must reveal depth progressively

VyteMerge should not start by showing:
- a giant graph
- a dense multi-branch visualization
- every relationship at once

The user should first understand:
- my line
- my next moments
- what connects to me

Then:
- related lines
- shared moments
- conflicts
- merged timelines
- resources
- graph fragments

This means Timeline must have **multiple view modes**, each with a clear purpose.

---

# 3. View model ladder

VyteMerge Timeline should evolve through a ladder of views:

1. **Glimpse**
2. **Personal Line**
3. **Expanded Personal**
4. **Related Timelines**
5. **Merged View**
6. **Graph-leaning View** (future)

The user does not need to experience all of these immediately.
The product should support them conceptually.

---

# 4. View Mode 1 — Timeline Glimpse

## Purpose

A small, embedded temporal preview that appears outside the Timeline tab.

Mainly used in:
- Home
- possibly Profile
- maybe certain Feed/Explore fragments

## What it communicates

- "This is your line"
- "These are your next meaningful moments"
- "Something is connecting to your life"
- "Tap to go deeper"

## What it should show

- 2 to 4 upcoming meaningful nodes
- optional moving avatar / current position marker
- one or two incoming/connected moments if relevant
- compact temporal rhythm
- minimal identity markers

## What it should NOT show

- full detail
- dense relation maps
- every conflict
- every branch
- all participants

## Visual treatment

- compact strip or micro-line
- node-first, not card-first
- behaves like a glance at a watch
- should feel lightweight

## Primary interaction

- tap → opens full Timeline

---

# 5. View Mode 2 — Personal Line

## Purpose

The default Timeline view for most users.

This is the first serious view of the user's temporal life.

## What it communicates

- "This is my line of life/time"
- "These are my next moments"
- "I can read my own time at a glance"

## What it should show

- my own line
- my nodes in order
- sparse compression when empty
- dense structure when busy
- slot vs event distinction
- moving avatar / current position
- minimal conflict indicators if they affect me directly

## What it should NOT show by default

- multiple branches
- large graph overlays
- all related timelines
- heavy convergence UI unless directly relevant

## Visual treatment

- clear personal spine/line
- nodes as primary unit
- sparse and dense areas adapt naturally
- shared moments may be hinted, but not overloaded

## Primary interactions

- tap node → detail
- tap conflict → resolve view
- toggle to expand context

---

# 6. View Mode 3 — Expanded Personal

## Purpose

A deeper version of the personal line where the user can inspect context, participants, and richer temporal meaning.

This is still "my timeline", but with more surface information.

## What it communicates

- "This moment has detail"
- "This thing is shared"
- "This thing is connected to someone/something"
- "This may create consequences"

## What it should show

- richer node detail on expand
- participant previews
- resource identity
- place identity
- small convergence hints
- conflict context
- possible actions:
  - add to timeline
  - cancel
  - reschedule
  - resolve conflict
  - open shared resource/event

## Example

A football match node:
- main node = field or match identity
- expanded = invited friends, place, time, status

A doctor appointment:
- main node = patient or appointment context
- expanded = shared moment context, privacy-preserving detail

## Rule

Expanded Personal must still feel timeline-first, not dashboard-like.

---

# 7. View Mode 4 — Related Timelines

## Purpose

Allow the user to turn on/off timelines related to them.

Examples:
- spouse
- family
- children
- delegated professional timelines
- school
- clinic
- business resources
- agreements

## What it communicates

- "My time is connected to other allowed timelines"
- "I can compare my line with other relevant lines"
- "Some other time contexts affect mine"

## What it should show

- my line as anchor
- one or more related lines
- clear ownership per line
- visibility levels respected
- simple convergence points
- optional toggles:
  - Me
  - Family
  - Work
  - Shared
  - Resources

## What it should NOT show

- every possible connected entity at once
- graph chaos
- unrestricted visibility

## Visual treatment

- multiple lines/branches
- still readable as timeline
- related lines can be dimmer than the active line
- BusyOnly lines can appear as reduced-information branches

## Rule

Related Timelines is the first point where the graph becomes noticeable, but timeline readability must still dominate.

---

# 8. View Mode 5 — Merged View

## Purpose

Show true convergence between multiple timelines and resources in a way that helps coordination and decision-making.

This is where VyteMerge becomes clearly more than a calendar.

## Typical use cases

- hospital + doctors + room
- band members + concert
- family + school event
- football match + invited friends
- provider + resource + booking
- professional + second work assignment causing conflict

## What it communicates

- "These lines meet here"
- "These people/resources share this moment"
- "This is coordinated time, not isolated time"

## What it should show

- multiple lines visible
- merge points
- shared event nodes
- resource line participation
- conflict zones
- enough information to understand the relationship
- not necessarily every detail of each participant's life

## Visual treatment

- line convergence
- merge indicators
- horizontally connected or spatially aligned nodes
- event-centric or resource-centric reading possible

## Important rule

Merged View is not "show everything".
It is "show the relationships relevant to this moment or time window".

---

# 9. View Mode 6 — Graph-leaning View (future)

## Purpose

A more exploratory mode where the temporal graph becomes explicit.

This is not the starting point of the product.
It is a future advanced mode.

## When it might be useful

- institutional coordination
- event networks
- location-based timeline exploration
- public timeline discovery
- browsing the connected life of places, people, and resources
- advanced temporal graph exploration

## What it communicates

- "Time is a network"
- "Lives, places, resources, and opportunities intersect"
- "I can follow these intersections"

## Important warning

This mode should not replace:
- Glimpse
- Personal Line
- Related Timelines

Those remain the default mental model.
Graph view is advanced and optional.

---

# 10. Home vs Timeline

## Home

Home should show:
- my line in miniature
- what is coming soon
- what is connecting to my life
- opportunities, suggestions, and relevant content around my time

Home is not the full Timeline.
It is:
# my line + connected discovery

## Timeline tab

Timeline is where the user goes to:
- understand their time deeply
- inspect shared moments
- act on conflicts
- compare with related lines
- manage temporal life

### Rule

Home previews.
Timeline explains and enables action.

---

# 11. Feed / Explore / Profile relation

## Feed

Feed may contain graph fragments:
- public timeline slices
- shared moments
- public temporal signals
- availability hints

But Feed is not the timeline itself.

## Explore

Explore should remain search-first and simpler.
It may expose:
- timeline fragments
- public availability
- event hints
- graph-derived relevance

But it should not become a full graph browser in v1.

## Profile

Profile may show:
- condensed timeline fragments
- activity markers
- availability hints
- identity + offer + temporal presence

But Profile is not the primary action surface for time.

---

# 12. Progressive disclosure rules

## Rule 1 — Start with my line

The first thing the user should understand is:
- my time
- my next moments
- my life line

## Rule 2 — Reveal relation only when useful

Convergences, merges, and graph-like behavior appear:
- on hover
- on tap
- on expand
- when toggling related lines
- in explicit merged view

## Rule 3 — Complexity must feel earned

The user should feel:
- "I can go deeper if I need to"
not:
- "The product is throwing too much at me"

## Rule 4 — Privacy shapes what can be expanded

Even in deeper modes, visibility depends on:
- agreements
- access
- sharing level
- BusyOnly / BusyOnlyDetails / Full

---

# 13. Sparse / dense behavior by view mode

## Glimpse
- heavily compressed
- only meaningful moments
- minimal relation

## Personal Line
- adaptive sparse/dense
- clean sequence
- direct ownership

## Expanded Personal
- more detail
- more relation context
- node detail reveals depth

## Related Timelines
- sparse and dense can coexist across branches
- the active line should remain the visual anchor

## Merged View
- local density matters more than global density
- only relevant merge structures should be emphasized

---

# 14. Conflict visibility by mode

## Glimpse
- optional subtle alert only
- not full conflict UI

## Personal Line
- visible if directly relevant

## Expanded Personal
- enough context to understand the collision
- direct CTA

## Related Timelines
- show that another line contributes to the conflict

## Merged View
- clearly show the competing lines/resources
- make the conflict legible as a connected temporal situation

---

# 15. Recommended v1 delivery order

If prototyping the product progressively, the order should be:

## Step 1
Glimpse + Personal Line

## Step 2
Expanded Personal

## Step 3
Related Timelines

## Step 4
Merged View

## Step 5
Graph-leaning view (future)

This prevents the product from over-designing complexity before it earns it.

---

# 16. Decision summary

## Decision 1
Timeline should not start as a graph browser.

## Decision 2
The first mental model is always:
- my line
- my next moments

## Decision 3
Graph depth appears progressively through convergence, related lines, and merged view.

## Decision 4
Home uses timeline as glimpse and context, not as full management surface.

## Decision 5
Related timelines and merged views are key to differentiation, but must remain readable.

## Decision 6
Every deeper mode must preserve the core simplicity of the line of life.

---

# 17. Closing principle

> **VyteMerge Timeline should feel simple when personal, richer when shared, and graph-capable when needed — without ever losing the user's sense of where they are in time.**
