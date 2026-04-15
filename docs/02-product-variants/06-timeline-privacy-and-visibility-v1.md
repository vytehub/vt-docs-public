# Timeline Privacy and Visibility v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines how **privacy, visibility, and shared temporal truth** work in VyteMerge Timeline.

Its purpose is to answer questions such as:

- when should another timeline be visible?
- what can be seen without exposing too much?
- how can two lines intersect without revealing all details?
- what does it mean to share time but not narrative?
- how should Home, Feed, Explore, and Timeline handle temporal visibility?
- how do agreements shape what is visible and actionable?

This is a product and UX document.
It is not a backend permission matrix.

Its role is to protect a core truth of VyteMerge:

> **time can be coordinated without requiring full transparency.**

---

# 2. Core privacy principle

## 2.1 Shared time is not shared life

VyteMerge does not assume that because two entities are temporally connected, they are fully visible to each other.

A shared moment may exist between:
- doctor and patient
- football field and players
- hospital and doctors
- family members
- revendedora and professional
- provider and client
- musician and venue
- resource and organization

But this does NOT mean:
- everyone sees everyone's full timeline
- everyone sees the full story behind every event
- every intersecting timeline becomes fully explorable

## 2.2 The user should see what is necessary, not everything

Visibility should be enough to:
- coordinate
- detect conflict
- identify relevant shared moments
- make decisions

Visibility should NOT automatically reveal:
- private life details
- unrelated commitments
- sensitive descriptions
- full timeline history
- internal business context without permission

---

# 3. The three layers of temporal visibility

## Layer 1 — Existence

The system may reveal that:
- something exists at this time
- someone is busy
- a shared moment exists
- a resource is occupied

This is the minimum useful layer.

## Layer 2 — Context

The system may reveal limited context:
- type of event
- tags
- broad category
- public/shared identity
- rough relationship

This gives coordination value without full narrative exposure.

## Layer 3 — Narrative

The system may reveal:
- title
- description
- participants
- notes
- reasons
- sensitive context
- exact private meaning

This is the most restricted layer.

### Rule
The product should assume that:
- existence is easiest to share
- context is selectively shareable
- narrative is highly protected

---

# 4. Visibility levels

## 4.1 None

### Meaning
The other timeline is not visible at all.

### User experience
- no node
- no intersection
- no occupancy hint
- no indication of the hidden line

### Use when
- there is no agreement
- the timeline is fully private
- the user should not even know the timeline exists in this context

---

## 4.2 BusyOnly

### Meaning
The user can see:
- that time is occupied
- that something exists there
- that a shared or overlapping moment is happening

But cannot see:
- title
- participants
- description
- reason
- narrative

### User experience
- generic node or block
- label like "Ocupado"
- no avatar or private identity
- merge or conflict can still be visible if relevant

### Use when
- coordination matters
- privacy must remain strong
- shared scheduling is needed without narrative exposure

---

## 4.3 BusyOnlyDetails

### Meaning
The user can see:
- occupancy
- event type
- category
- some metadata
- possibly a public/shared identity marker

But cannot see:
- sensitive narrative
- notes
- protected details
- private participant data unless allowed

### User experience
- richer than BusyOnly
- still not a full reveal
- enough for coordination and context

### Use when
- the receiver needs better understanding
- but full narrative is still inappropriate

---

## 4.4 Full (context-bound)

### Meaning
The user can see the complete meaning of the moment within allowed scope.

This may include:
- title
- participants
- context
- detail
- notes
- actions

### Important
Full does not mean:
- unlimited access to everything forever
- unlimited exploration of the whole other life
- unrestricted system-level visibility

### Use when
- delegation applies
- the moment is inherently shared
- the user is an authorized participant
- organizational responsibility requires full context

---

# 5. Shared moments vs shared timelines

## 5.1 Shared moment

A user may be allowed to see:
- a specific event
- a specific convergence
- a specific booking
- a specific conflict

without gaining visibility into the whole timeline behind it.

### Example
A doctor sees an appointment with a patient.
The doctor does not automatically see the patient's full life timeline.
They only share that one moment.

## 5.2 Shared timeline

A user may be allowed to see:
- another timeline partially or fully
- recurring occupancy
- related commitments
- active temporal context

This only happens when an agreement explicitly grants it.

### Rule
The product should strongly prefer:
- shared moments first
- shared timelines only when needed and explicitly granted

---

# 6. Privacy-aware intersections

## 6.1 What the user may see

A user may legitimately see that:
- their line intersects another line
- their event overlaps another event
- their resource is occupied by someone else
- a shared event exists
- a conflict exists

without seeing:
- the rest of the other line
- the full reason
- the private content

This is one of the most important UX principles in the product.

## 6.2 Visual rule

An intersection may be visible even when the connected timeline is only partially visible.

That means:
- the merge point exists visually
- the relationship is legible
- the hidden side remains abstract or reduced

### Example
My match conflicts with another assignment.
I may see:
- that another assignment exists
- who or what is broadly involved if permitted
- why this matters to me

I may not see:
- the full unrelated chain behind that assignment

---

# 7. Ownership and visibility

## 7.1 Timeline owner

Every timeline belongs to an owner:
- person
- resource
- organization context
- family/shared line
- temporal container if modeled as such

The owner controls:
- whether it is shareable
- with whom
- at which level
- under what conditions

## 7.2 Visibility is granted, not inferred

Following someone does not imply:
- timeline access
- occupancy access
- conflict access
- narrative access

Booking something does not imply:
- full timeline access to the provider
- full timeline access to the client

Being temporally connected does not imply total visibility.

---

# 8. Agreements as visibility bridges

## 8.1 Agreements do not only allow actions
They also define:
- what part of time is visible
- what depth of meaning is visible
- whether coordination is read-only or operational

## 8.2 Agreement outcomes

An agreement may grant:
- no visibility
- BusyOnly
- BusyOnlyDetails
- Full
- read-only
- operational management

## 8.3 Important principle
The bridge should be as narrow as possible while still making coordination work.

Do not over-share just because it is easier technically.

---

# 9. Surface-specific visibility behavior

## 9.1 Home

### What Home may show
- my own line
- my own next moments
- things that connect to my life
- limited connected signals

### What Home should avoid
- exposing private narratives of others
- heavy detail about other timelines
- institutional overexposure
- too many visible connected lines at once

### Rule
Home should feel personal and relevant, not voyeuristic.

---

## 9.2 Feed

### What Feed may show
- public/shared fragments
- moments intentionally shared
- temporal opportunities
- graph fragments that are meant to be seen

### What Feed should avoid
- leaking private schedule structure
- implying that a user's full timeline is public
- showing hidden intersections that were not meant for public context

### Rule
Feed may show temporal fragments, but only when they are:
- public
- intentionally shared
- commercially/publicly relevant

---

## 9.3 Explore

### What Explore may show
- public availability
- public schedule fragments
- discoverable opportunities
- public/shared event signals

### What Explore should avoid
- deep private line inspection
- non-consensual timeline browsing
- raw visibility into others' life structures

### Rule
Explore is for discoverability, not surveillance.

---

## 9.4 Timeline tab

### What Timeline may show
- my own line fully
- related lines according to agreements
- shared moments
- BusyOnly or BusyOnlyDetails for connected timelines
- conflicts that matter to me
- resource lines if I am allowed to see them

### Rule
Timeline is where privacy complexity is most carefully handled.
It must reveal enough to support decisions, but not too much.

---

# 10. BusyOnly UX rules

## 10.1 BusyOnly must still feel real

BusyOnly should not disappear entirely.
If it matters to coordination, it should be visible.

### Good BusyOnly treatment
- generic node or temporal block
- "Ocupado"
- line color or ownership tint
- maybe time bounds
- no narrative detail

### Bad BusyOnly treatment
- showing nothing
- showing too much
- making the user think the time is free when it is not

## 10.2 BusyOnly in convergence

A shared moment can still show:
- the fact of convergence
- the fact of occupation
- the fact that another line participates

without revealing:
- who exactly
- why exactly
- what private story sits behind it

This allows the graph to remain truthful without violating privacy.

---

# 11. Shared events with partial privacy

## 11.1 Example — doctor and patient

The doctor sees:
- there is an appointment
- there is a patient
- the time is committed

The doctor may see:
- patient identity, if appropriate
- the appointment context, if appropriate

The doctor should not automatically see:
- unrelated patient schedule
- private timeline beyond the appointment

## 11.2 Example — football field and invited friends

The booking owner sees:
- the match node
- invited friends connected to the event

A friend may see:
- the shared event
- maybe the field or match identity
- their own relation to it

A friend does not need to see:
- all the inviter's timeline
- all other friends' unrelated commitments

---

# 12. Resources and institutional visibility

## 12.1 Resource timelines may be widely visible
A room or field may have broader visibility than a person's private line.

But even then:
- the resource can be visible
- while the private reason behind some occupancy remains protected

### Example
Meeting room occupied:
- the room is busy
- maybe the department is visible
- the private internal reason may not be

## 12.2 Institutional merged views

Hospitals, schools, clinics, and organizations may need:
- merged lines
- room visibility
- staff occupancy
- conflict awareness

But even there:
- staff family life is not fully visible
- private patient narratives are not fully visible
- the product shows what is necessary for operations

---

# 13. Conflict visibility vs privacy

## 13.1 A conflict can be real without full explanation

The user may need to know:
- something conflicts
- another timeline or assignment is involved
- they must choose

without seeing:
- the entire other context

## 13.2 UX principle
Conflict is a visible signal.
Narrative remains permission-based.

This is especially important in:
- professional vs family overlap
- institution vs personal life
- reseller vs final clients
- patient privacy
- partner/shared household views

---

# 14. Interaction rules under privacy

## 14.1 If visibility is None
- no interaction
- no reveal
- no hidden clickable placeholder

## 14.2 If visibility is BusyOnly
Tap may reveal:
- "Ocupado"
- approximate temporal context
- no deep story

## 14.3 If visibility is BusyOnlyDetails
Tap may reveal:
- limited category
- tags
- broad event type
- controlled metadata

## 14.4 If visibility is Full
Tap/expand may reveal:
- all relevant detail
- actions if permissions allow

### Rule
Interaction must never imply more access than the visibility level grants.

---

# 15. Public vs private vs shared temporal content

## Public temporal content
Content intentionally meant to be discovered:
- public event
- public availability
- public listing slot
- public business timeline fragment

## Private temporal content
Content only for the owner or explicitly delegated users.

## Shared temporal content
Content visible to a bounded set of participants or connected entities.

### Rule
The product must keep these categories clear.
A shared moment is not automatically public.
A public fragment is not automatically a full timeline.
A private line can still intersect meaningfully with shared/public surfaces.

---

# 16. V1 / V2 / future

## V1
- clear visibility levels
- BusyOnly signals
- privacy-aware intersections
- no accidental overexposure
- realistic Home/Feed/Explore boundaries

## V2
- richer shared moment treatment
- better participant abstraction
- more nuanced institutional visibility
- cleaner merge behavior with privacy-preserving context

## Future
- user-controlled visibility presets
- per-node privacy overrides
- advanced shared-context permissions
- public/private hybrid timeline publishing for businesses

---

# 17. Design decisions

## Decision 1
**Time can be coordinated without full transparency.**

## Decision 2
**A shared moment does not imply a shared life.**

## Decision 3
**BusyOnly must still be visually truthful.**
Hidden time should not look free.

## Decision 4
**Intersections may be visible even when the connected line remains private.**

## Decision 5
**Visibility is granted explicitly, never inferred from following or generic connection.**

## Decision 6
**Every surface must respect privacy differently according to its role.**
Home ≠ Feed ≠ Explore ≠ Timeline.

## Decision 7
**The product must reveal enough to support coordination, and no more than that.**

---

# 18. Closing principle

> **VyteMerge should let people coordinate time, see meaningful intersections, and act on shared moments without forcing full exposure of private life.**
