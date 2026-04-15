# Timeline UI Prototyping Rules v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **rules for prototyping Timeline in UI/UX-only mode**.

It exists to make sure that when Claude Code or the team prototypes Timeline:

- the result stays aligned with the product vision
- the UI does not collapse into a generic calendar or event list
- complexity is introduced progressively
- mock data tells the correct story
- the prototypes remain usable, reviewable, and reusable
- visual experimentation happens inside clear boundaries

This document is not a backend or implementation spec.
It is a **design-operational standard** for building Timeline UI iterations safely.

---

# 2. Golden rule

## Prototype the product truth, not a convenient calendar

Every Timeline prototype must reinforce:

- line of life/time
- nodes as protagonists
- semantic compression of empty time
- privacy-aware convergence
- slot vs event distinction
- graph emergence only where it matters

If the prototype looks like:
- Google Calendar
- a Kanban board
- a task list
- a dashboard of cards
- a generic event stream

then the prototype failed the product truth.

---

# 3. Scope of prototyping

Timeline prototypes in this phase should explore:

- visual grammar
- node design
- sparse vs dense rendering
- convergence
- conflict presentation
- Home glimpse
- related timeline toggles
- merged/expanded reading
- visibility/privacy handling

Timeline prototypes in this phase should NOT depend on:

- real backend integration
- real booking logic
- real conflict engines
- real agreement resolution
- production-grade data models

This is a **UI/UX exploration layer**, not backend-first implementation.

---

# 4. Mock-first rules

## 4.1 Mocked data is mandatory

Every Timeline prototype must render with believable, curated mock data.

No empty states unless the prototype is explicitly about empty states.

## 4.2 Mock data must tell temporal stories

Mock data should not be random.

Each prototype should make a scenario legible, for example:
- sparse life (3 moments far apart)
- doctor/patient shared event
- football match + invited friends
- resource line (meeting room)
- family conflict
- institutional convergence
- booking vs slot distinction

## 4.3 Mock data should be intentional

Use:
- realistic names
- meaningful time ranges
- believable contexts
- identities that reinforce the product model

Do not use:
- lorem ipsum
- "Test Event 1"
- meaningless placeholders
- random unrelated content

---

# 5. Allowed prototype shapes

## 5.1 Timeline glimpse prototype

Small embedded strip or compact line:
- 2 to 4 next moments
- current position marker
- light relation hints

Use for:
- Home experimentation
- "my line + things connecting to it"

## 5.2 Personal line prototype

Single line:
- sparse or dense
- clear node language
- current position marker
- slot vs event difference

Use for:
- baseline timeline language

## 5.3 Shared moment prototype

A focused composition showing:
- one event
- more than one actor or resource
- privacy-aware relation

Use for:
- convergence experiments
- patient/doctor
- football field + friends
- room + staff

## 5.4 Related timelines prototype

My line + 1 or 2 additional lines:
- family
- work
- shared resource
- agreement-based line

Use for:
- comparison
- context
- conflict awareness

## 5.5 Conflict prototype

A focused composition where:
- two temporal claims collide
- the user can perceive the collision quickly
- the conflict suggests an action

Use for:
- veterinary task vs football match
- family obligation vs work slot
- room already occupied

---

# 6. Forbidden prototype patterns

## 6.1 Literal calendar grids

Do not prototype Timeline primarily as:
- month grid
- week grid
- day planner grid

These may exist as fallback or legacy support, but not as the main design language.

## 6.2 Full graph from the start

Do not jump directly to:
- many branching lines
- giant node-link diagrams
- 3D graph-like views
- dense relation maps

The graph must emerge progressively.

## 6.3 Card overload

Do not prototype Timeline as a stack of large editorial cards.

Nodes are the protagonist.
Cards may appear on expand, but not as the default unit of reading.

## 6.4 Control-panel overload

Do not fill the timeline prototype with:
- too many toggles
- too many filters
- too many persistent controls
- too many CTA buttons

If the user spends more time reading controls than reading time,
the prototype is wrong.

## 6.5 Fake precision

Do not draw time too literally if it makes the prototype worse.

If there are 4 months of empty time:
- compress them

If there is one quiet week:
- make it quiet

This is semantic time, not literal chronology scaling.

---

# 7. Node prototyping rules

## 7.1 Node-first

Every prototype must start from the node language.

Before prototyping:
- define what this node is
- define whether it is slot or event
- define whether it is personal/shared/resource/conflict
- define whose identity is in the center
- define how time is encoded on the edge

## 7.2 Use the temporal identity marker idea

Node anatomy should support:
- center = identity
- edge = time
- optional secondary indicators

## 7.3 Node does not have to equal line owner

Do not assume the node shows the owner.
In many cases, the main identity of the node is:
- patient
- room
- field
- concert
- match
- assignment
- dog-grooming task
- school event

## 7.4 Slot and event must be visibly different

Slot:
- possibility
- hollow / lighter
- no participant convergence

Event:
- commitment
- stronger
- connected

This distinction must be readable at a glance.

---

# 8. Line prototyping rules

## 8.1 Line is support

A line should:
- orient
- sequence
- relate

A line should not:
- dominate
- visually outweigh the nodes
- make empty time too heavy

## 8.2 Sparse compression is required

At least one prototype should explicitly test:
- three far-apart moments
- compressed empty time
- node sequence without giant dead space

## 8.3 Dense clustering is required

At least one prototype should explicitly test:
- many nearby commitments
- compact sequence
- clear local hierarchy
- conflict readability

## 8.4 Current position marker is optional but recommended

The moving avatar/"muñequito" is useful when:
- showing "where I am now"
- anchoring the user in their time
- helping Home glimpse feel alive

Do not overuse it if it competes with the nodes.

---

# 9. Convergence and shared event prototyping rules

## 9.1 Show convergence locally first

When prototyping shared events:
- begin with one event
- one or two actors/resources
- one convergence

Do not start with 20 connected lines.

## 9.2 Shared event is a stronger node

A shared event should feel like:
- a special moment
- multiple lines meeting
- something more consequential than a solo event

## 9.3 Privacy must shape the prototype

If another line is only partially visible:
- show the convergence
- abstract the other side
- do not expose full private detail

This is mandatory in prototypes, not optional.

---

# 10. Conflict prototyping rules

## 10.1 Conflict is a product opportunity

A conflict prototype should communicate:
- collision
- consequence
- choice

It should not just show a red icon and stop there.

## 10.2 Conflict must remain readable

Do not create conflict UIs that require too much explanation.
The user should quickly understand:
- these two things overlap
- they both matter
- I need to decide

## 10.3 Use real scenarios

Preferred conflict prototypes:
- football match vs veterinary assignment
- school event vs work booking
- room double-booking
- provider listing slot vs personal commitment

Avoid fake/artificial conflict cases.

---

# 11. Surface-specific prototyping rules

## 11.1 Home

Home prototypes should focus on:
- timeline glimpse
- my line
- what is connecting to it
- not full management

### Good Home prototype
- compact line
- upcoming nodes
- one or two connected opportunities
- CTA into full Timeline

### Bad Home prototype
- giant mini-calendar
- too many branches
- heavy graph complexity
- too many actions

## 11.2 Feed

Feed prototypes should focus on:
- graph fragments
- public/shared moments
- temporal discovery

### Good Feed prototype
- timeline-derived fragment in a richer content card
- temporal hint that leads somewhere

### Bad Feed prototype
- full raw graph embedded in feed
- timeline management controls inside feed

## 11.3 Explore

Explore prototypes should focus on:
- search-first
- simple result reading
- temporal hints
- progressive refinement

### Good Explore prototype
- search bar
- simple result set
- lightweight temporal cues

### Bad Explore prototype
- rigid dashboard
- too many tabs upfront
- graph browser behavior

## 11.4 Timeline

Timeline prototypes should focus on:
- strongest temporal identity
- clearest node/line language
- convergence and conflict
- relation depth when needed

---

# 12. Review checklist for every prototype

Before showing a prototype, ask:

## Product truth
- Does this still feel like a line of life/time?
- Did we accidentally drift into a calendar or event list?
- Are nodes the protagonist?

## Temporal semantics
- Is slot vs event obvious?
- Is sparse time compressed?
- Is dense time legible?

## Relation
- Is convergence visible where needed?
- Is graph emergence progressive rather than overwhelming?
- Are related lines understandable?

## Privacy
- Does the prototype reveal only what it should?
- Can shared moments exist without full line exposure?

## Surface fit
- Does this belong in Home / Feed / Explore / Timeline?
- Did we accidentally overload the wrong surface?

## Actionability
- Is the next user action clear?
- If there is conflict, can the user understand what to do?

---

# 13. Recommended prototype sequence

To evolve Timeline safely, prototype in this order:

## Step 1
Personal sparse timeline

## Step 2
Personal dense timeline

## Step 3
Slot vs event comparison

## Step 4
Shared event / convergence

## Step 5
Conflict

## Step 6
Home glimpse

## Step 7
Related timelines

## Step 8
Merged view

## Step 9
Feed/Explore graph fragments

This sequence prevents the team from jumping into complexity too early.

---

# 14. What Claude should optimize for

When Claude prototypes Timeline UI, it should optimize for:

- clarity over novelty
- strong product identity over generic convenience
- local meaning over total graph exposure
- believable mock stories over empty screens
- progressive disclosure over feature density
- visual rhythm over control clutter

Claude should NOT optimize for:
- calendar familiarity
- dashboard complexity
- theoretical completeness
- backend correctness before UX truth

---

# 15. Exit criteria for a good prototype

A Timeline prototype is good enough to move forward when:

- the user can explain what they are seeing without help
- slot vs event is obvious
- a shared moment reads as shared
- a conflict reads as conflict
- the timeline does not feel like a generic planner
- the prototype tells a believable temporal story
- the result gives new clarity about the product

A prototype is not good enough if:
- it is technically correct but visually generic
- it needs too much explanation to make sense
- it looks like a list/calendar/dashboard
- it exposes too much private detail
- it overwhelms the user with graph structure too early

---

# 16. Design decisions

## Decision 1
**Prototype from node language outward, not from layout inward.**

## Decision 2
**Use mock data that tells temporal stories, not random placeholders.**

## Decision 3
**Keep the timeline primary and let graph emerge only where necessary.**

## Decision 4
**Do not prototype full graph complexity before validating personal line clarity.**

## Decision 5
**Every prototype must respect privacy-aware intersections.**

## Decision 6
**Surface matters: Home, Feed, Explore, and Timeline must prototype different interaction depth.**

## Decision 7
**Prototypes are successful when they make the product easier to understand, not just more visually interesting.**

---

# 17. Closing principle

> **A good Timeline prototype makes time, identity, convergence, and conflict feel natural — without collapsing into a generic calendar or exploding into an unreadable graph.**
