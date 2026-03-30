# Timeline Visual Grammar v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **visual language** of VyteMerge Timeline.

It is not a backend specification, a data model, or a feature list.
It describes how Timeline **looks, behaves, and communicates** as a visual system.

Its goal is to establish a shared understanding of:

- what elements compose a Timeline visually
- how those elements relate to each other
- what the user perceives at different densities
- how time, identity, and convergence are expressed graphically
- how Timeline fragments appear outside the Timeline surface

This document exists because Timeline is the most visually distinctive part of VyteMerge.
If Timeline looks like a generic calendar, the product loses its identity.

---

# 2. Core visual principles

## 2.1 Timeline is a line of life

A Timeline is a continuous line that represents the passage of time for an entity.

It is not a grid.
It is not a table.
It is not a spreadsheet of hours.

It is a **line** — vertical by default — along which things happen.

The line itself is the spine.
Events, bookings, conflicts, and intersections are expressed as **nodes** on or near the line.

## 2.2 Nodes are the protagonist

The primary visual unit of Timeline is the **node**.

A node represents a moment when something occupies time.

Nodes are:
- compact
- identifiable at a glance
- distinguishable by type (color, icon, shape)
- interactive (tappable, expandable)

Nodes are NOT:
- large editorial blocks
- full cards by default
- indistinguishable from each other

## 2.3 Lines are support, not protagonists

The vertical line (spine) and horizontal connectors (merge lines) exist to give structure.

They should be:
- visually subdued
- thinner than node elements
- consistent in style (solid for own timeline, dashed for shared)

Lines tell the user "there is continuity" and "these things connect."
Lines do NOT compete for attention with nodes.

## 2.4 Time compresses when empty

Empty time should not occupy visual space.

Days without events collapse.
Hours without activity disappear.
The visual density of the Timeline reflects the **density of life**, not the density of a clock.

A quiet week looks short.
A busy week looks rich.

## 2.5 Identity is always present

Every node on a Timeline carries identity:
- whose time this is (the line owner)
- who else is involved (the other participant)
- what kind of event it is (booking, private, shared, conflict)

Identity is expressed through:
- avatars
- colors
- position on or near a line
- badges and status indicators

---

# 3. Sparse vs dense timeline

## 3.1 Sparse timeline

A sparse timeline has few events spread across many days.

Visual behavior:
- Empty days collapse to a minimal line or dot
- Visible nodes are spaced apart, with clear breathing room
- The spine line is prominent because there is nothing else to see
- The overall impression is "calm, open, available"

The sparse timeline communicates: "this person has time."

## 3.2 Dense timeline

A dense timeline has many events in close proximity.

Visual behavior:
- Nodes stack tightly along the spine
- Overlap zones are visible as clusters or merged regions
- Connectors between lines become more prominent
- The overall impression is "busy, structured, coordinated"

The dense timeline communicates: "this person is active and committed."

## 3.3 Transition between sparse and dense

As the user scrolls through time, the visual density changes naturally.

There is no mode switch.
The same visual grammar adapts:
- sparse regions → more space, collapsed days, visible spine
- dense regions → tighter nodes, visible structure, potential conflicts

---

# 4. Node types

## 4.1 Booking node (received)

The user is the provider. Someone booked their time.

Visual cues:
- Primary accent color (green)
- Avatar of the client
- Duration indicator (proportional bar or text)
- Status dot (pending=yellow, confirmed=green)
- Inline CTA if pending: confirm/reject

## 4.2 Booking node (made)

The user booked someone else's time.

Visual cues:
- Secondary accent color (blue)
- Avatar of the provider
- Duration indicator
- Status: confirmed, pending, completed

## 4.3 Private event node

A personal commitment, manually created.

Visual cues:
- Neutral color (gray or muted)
- No avatar (or owner's avatar)
- Title and duration
- No CTA (edit via tap-to-expand)

## 4.4 Shared event node

An event visible through a shared timeline (Agreement Sharing).

Visual cues:
- Color of the shared branch
- Opacity reduced if BusyOnly (shows "Occupied" without details)
- Dotted connector to the shared line
- No actions available (read-only)

## 4.5 Conflict node

Two or more events overlap in time across branches.

Visual cues:
- Red accent or warning indicator
- Visual stacking of the conflicting nodes
- Connector line between the conflicting elements
- CTA: "Resolve"

## 4.6 Slot node (projected availability)

A window of bookable time, projected from listing rules.

Visual cues:
- Subtle, low-contrast (lighter than event nodes)
- Dashed border or outline style
- No avatar
- No status dot
- Communicates "this time could be used" without asserting it is used

---

# 5. Slot vs event

This distinction is fundamental and must be visually unambiguous.

## Event = something real occupies time

- Solid visual treatment
- Full opacity
- Clear identity markers (avatar, title, color)
- Actionable (tappable, expandable, may have CTAs)

## Slot = something could occupy time

- Subtle visual treatment
- Reduced opacity or outline-only style
- No identity markers (no avatar, no participant)
- Not directly actionable (represents potential, not commitment)

The user must never confuse "I have a booking" with "I have an available slot."

---

# 6. Shared events and convergence

## 6.1 What convergence means

Convergence is when two or more timelines have activity at the same moment.

This can be:
- a booking that involves both parties (natural convergence)
- two unrelated events that happen to overlap (conflict convergence)
- a shared event visible from multiple perspectives

## 6.2 How convergence is shown

When two lines converge, a horizontal connector bridges them.

The connector:
- is thin and subdued
- connects the relevant nodes horizontally
- optionally carries a label ("shared", "conflict")

In stream view, convergence is shown as stacked cards with a merge indicator.
In graph view, convergence is shown as horizontal edges between branch columns.

## 6.3 Merge points

A merge point is a specific moment where two branches share an event.

Visual treatment:
- Both nodes visible (one per branch)
- Horizontal line connecting them
- Shared time range highlighted
- If conflict: red accent on the connector

Merge points are the visual signature of VyteMerge.
They are what makes this not a calendar.

---

# 7. Resource nodes

A Timeline can belong to a resource (room, equipment, court), not just a person.

Resource nodes behave like person nodes but with:
- An icon instead of an avatar (or a resource-type icon)
- A resource label instead of a name
- The same temporal behavior (slots, events, conflicts)

In multi-branch view, resource timelines appear as their own columns/branches alongside person timelines.

This allows a secretary to see: "Dr. Pérez has a booking at 10am in Consultorio A" as two connected nodes on two branches.

---

# 8. Conflict nodes

## 8.1 What conflicts look like

A conflict is always shown as a compound visual:
- Two or more overlapping nodes
- A warning indicator (red border, icon, or badge)
- A connector line between the conflicting elements
- A visible CTA ("Resolver")

## 8.2 Conflict severity is implicit

The visual system does not grade conflicts by severity.

All conflicts are shown with the same warning treatment.
The user decides importance by context, not by system-assigned priority.

## 8.3 Resolved conflicts

Once resolved, the conflict indicator disappears.
The nodes remain but return to their normal visual state.
No visual scar — resolved means resolved.

---

# 9. Privacy-aware intersections

## 9.1 The problem

Timeline Sharing allows others to see your occupied time.
But not all visibility levels expose the same information.

The visual grammar must respect:
- **BusyOnly**: the node exists, it has a time range, but it has no title, no avatar, no details
- **BusyOnlyDetails**: the node shows some metadata (tags, type) but not full content
- **None**: the node does not exist visually

## 9.2 Visual treatment of BusyOnly nodes

- Generic shape (rounded block, no distinguishing icon)
- Label: "Ocupado"
- Branch color maintained (so the user knows whose time it is)
- No avatar, no title, no description
- Cannot be expanded (tap does nothing or shows "Details not available")

## 9.3 Convergence with privacy

A shared event between a BusyOnly timeline and the viewer's own timeline:
- The viewer's node is fully visible
- The shared node shows as "Ocupado"
- The merge connector still exists (convergence is visible)
- But the content of the other side is private

This allows the system to communicate: "something is happening at the same time on another timeline" without revealing what.

---

# 10. Home timeline glimpse

## 10.1 What it is

Home may show a condensed fragment of the user's timeline.

This is not the full Timeline surface.
It is a **glimpse** — a small, embedded representation that says:
- "You have something coming up"
- "Your day looks like this"
- "Tap to see your full timeline"

## 10.2 Visual treatment

The glimpse is:
- A compact horizontal or vertical strip
- 2-4 upcoming nodes, shown as mini-nodes
- Positioned within the Home surface among other content blocks
- Tappable → navigates to `/timeline`

The glimpse should feel like peeking at a watch, not opening a calendar.

## 10.3 The moving avatar

The timeline owner's position in time can be represented by a small avatar marker ("muñequito") on the spine line.

This marker:
- Sits at the current moment on the line
- Moves forward as real time passes
- Communicates "you are here" on your own timeline
- Is only shown in the full Timeline surface or in the glimpse

The moving avatar anchors the user in time without needing a "Now" label.

---

# 11. Feed/Explore graph fragments

## 11.1 Timeline is not limited to the Timeline tab

Other surfaces may show **graph fragments** — small visual pieces of timeline data rendered in context.

Examples:
- A listing card showing "next available slot" as a mini timeline node
- A profile card showing "currently available" vs "busy" as a status dot derived from timeline
- An activity card showing "booking confirmed for tomorrow" as a temporal marker
- Discover showing providers with availability as a compact timeline strip

## 11.2 Rules for fragments outside Timeline

- Fragments are read-only (no inline actions)
- Fragments never show private timeline details of others
- Fragments use the same visual grammar (nodes, colors, status dots) at a smaller scale
- Fragments link back to the full Timeline or to the relevant detail view

## 11.3 Timeline remains the primary temporal surface

Fragments in Feed, Explore, or Profile do not replace the Timeline tab.
They are teasers that say "there is temporal information available."

The Timeline tab is where the user goes to **act** on their time.
Everything else is a glimpse.

---

# 12. V1 / V2 / long-term vision

## V1 — Skeleton (current)

What exists now:
- Stream view: vertical scroll with date headers, event cards
- Week view: grid-based calendar (functional but not aligned with the visual grammar)
- Basic event cards (booking, private event, conflict)
- Single-branch experience (no multi-timeline merge)
- Empty state with contextual CTAs

What V1 should close:
- Stream view feels like a timeline, not a calendar
- Nodes are compact and identifiable
- Empty time compresses visually
- Slot and event are visually distinct
- The vocabulary "Timeline" consistently communicates temporal coordination, not calendar management

## V2 — Visual grammar applied

What V2 introduces:
- Multi-branch view (personal + shared timelines as parallel lines)
- Merge points visible in stream view
- Conflict detection and inline resolution
- BusyOnly privacy treatment for shared timelines
- Home timeline glimpse (compact strip with upcoming nodes)
- Moving avatar marker on the spine
- Graph view as an alternative to stream (git-merge visualization)

What V2 requires:
- Shared timelines via Agreements (backend exists, frontend not yet wired)
- Conflict rule evaluation visible in the UI
- New toolkit components for merge indicators, branch selectors, privacy masks

## Long-term vision

- Timeline fragments embedded in Feed, Explore, Profile
- Provider availability strips on listing cards
- Real-time timeline updates (WebSocket or polling)
- Zoom-to-detail: pinch on graph view to explore temporal granularity
- AI-assisted conflict resolution suggestions
- Cross-timezone visualization with dual time display
- Timeline as a shareable, embeddable artifact (public timeline for businesses)
- Integration with external calendars (Google, Apple, Outlook) as additional branches

---

# Design decisions

## Decision 1
**Nodes are the protagonist, not lines.**
Lines provide structure. Nodes carry meaning.

## Decision 2
**Empty time compresses.**
A quiet timeline should look quiet, not empty.

## Decision 3
**Slot ≠ Event. Always.**
The visual distinction must be immediate and unambiguous.

## Decision 4
**Privacy is visual, not hidden.**
A BusyOnly event exists visually ("Ocupado") — it is not invisible.

## Decision 5
**Convergence is the visual signature.**
Merge points, shared events, and conflict indicators are what make VyteMerge distinct.

## Decision 6
**Timeline leaks into other surfaces as glimpses, not as full views.**
Home, Feed, Explore may show fragments. Timeline is the primary surface.

## Decision 7
**The moving avatar anchors the user in time.**
"You are here" is more intuitive than "Now: 14:35".

---

# Closing principle

> **The Timeline is not a calendar you look at.
> It is a living representation of your time, your commitments, and the moments where your life intersects with others.**
