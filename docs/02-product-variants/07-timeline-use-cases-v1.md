# Timeline Use Cases v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document gathers **canonical use cases** that validate the Timeline model of VyteMerge.

Its purpose is to:
- test the Timeline concept against real-world scenarios
- ensure the model works beyond a single niche
- validate temporal entities, temporal containers, agreements, visibility, and conflicts
- provide concrete examples for product, UX, and future implementation
- prevent the Timeline from drifting into a generic planner/calendar model

This is a product and conceptual validation document.
It is not a backend spec.

---

# 2. How to read these use cases

Each use case should be read against the current timeline model:

- entities with time
- lines of life/time
- nodes as temporal identity markers
- slot vs event
- shared events and convergence
- resource lines
- agreements as temporal bridges
- privacy-aware intersections
- conflicts as visible decision points

Each use case is valuable not only because it is realistic,
but because it reveals what the model must support.

---

# 3. Canonical use case structure

Each case includes:
- scenario
- temporal entities involved
- what the user should see
- what should remain hidden
- what kind of convergence/conflict exists
- why it validates the model

---

# 4. Use Case A — Sparse personal life timeline

## Scenario

A user has:
- one event in February
- one event in June
- one event in August

Nothing else is loaded.

## Temporal entities involved

- one person
- three personal events

## What the user should see

- three meaningful nodes
- ordered in temporal sequence
- compressed empty time
- almost no dominant line
- clear reading of "my next 3 moments"

## What should remain hidden

Nothing special.
This is fully personal.

## What kind of convergence/conflict exists

None.

## Why it validates the model

This case proves:
- the Timeline should not waste space showing empty time literally
- the line is support, not the main attraction
- sparse life can still be legible and emotionally meaningful
- Timeline is not a calendar grid

---

# 5. Use Case B — Doctor + patient appointment

## Scenario

A doctor has an appointment with a patient.

The doctor sees the commitment in their own Timeline.

The patient may also see the appointment in theirs.

## Temporal entities involved

- doctor (person timeline)
- patient (person timeline)
- appointment (shared event)
- clinic/room (optional resource timeline)

## What the doctor should see

- a committed node on the doctor's line
- the appointment's temporal identity
- patient/context identity if allowed
- the ability to recognize this as a shared moment

## What the patient should see

- the same temporal commitment from their own perspective
- the doctor/context identity if appropriate
- the fact that this moment belongs to their life too

## What should remain hidden

- the doctor's unrelated schedule
- the patient's unrelated life
- any narrative details not permitted

## What kind of convergence/conflict exists

A shared event between two personal lines.
Possibly also a room/resource line if one is involved.

## Why it validates the model

This proves:
- a node does not always represent the owner of the line
- shared moments can exist without sharing full private timelines
- privacy-aware convergence is necessary
- person ↔ person ↔ resource coordination is a core pattern

---

# 6. Use Case C — Football field booking + invited friends

## Scenario

A user reserves a football field sold through another user's listing.
They invite friends.

## Temporal entities involved

- booking owner (person timeline)
- invited friends (person timelines)
- football field (resource timeline)
- sports club or seller (organization/listing context)
- match/event as shared moment

## What the booking owner should see

- the main event node in their line
- the event represented by the field/match identity
- invited friends converging toward that shared event
- the match as a meaningful shared moment

## What invited friends should see

- the shared match in their own line
- their own relation to it
- possibly other visible participants if allowed
- no need to see the full life of the inviter

## What should remain hidden

- unrelated personal events of the inviter
- unrelated lives of other friends
- internal club details not relevant to the booking

## What kind of convergence/conflict exists

- multiple people converging into one shared event
- resource line (field) participating in that same moment

## Why it validates the model

This proves:
- a resource can be the identity of the event
- shared events may belong visually to a field/match rather than a person
- convergence is richer than "I booked something"
- listing-backed events can appear as timeline nodes

---

# 7. Use Case D — Hospital with meeting room and multiple doctors

## Scenario

A hospital has agreements with multiple doctors.
It also creates a timeline for a meeting room.
Two doctors and the room converge on the same meeting.

## Temporal entities involved

- doctor 1 (person timeline)
- doctor 2 (person timeline)
- meeting room (resource timeline)
- hospital (organization context)
- secretary or coordinator (delegated user)

## What the coordinator should see

- multiple related lines
- the meeting room line
- the doctor lines
- the convergence point where all of them meet
- enough information to coordinate the meeting

## What the doctors should see

- the meeting in their own line
- the room or meeting identity
- only the allowed contextual visibility of the other lines

## What should remain hidden

- doctors' unrelated private commitments
- room-internal details not relevant beyond the event
- private narrative from lines that are only shared as BusyOnly

## What kind of convergence/conflict exists

- people + resource convergence
- possible room conflict if another meeting overlaps
- possible personal/professional conflict if one doctor has another event

## Why it validates the model

This proves:
- resources can have timelines
- merged views matter
- agreements enable graph-capable coordination
- institutions need timeline + graph, not just booking tables

---

# 8. Use Case E — Music band + concert + autograph session + food truck

## Scenario

A user explores the timeline of a band.
They see three band members converging on a concert.
Then they follow the singer and see that the singer will also be signing autographs at a plaza.
At the same plaza and time, a favorite food truck will also be there.

## Temporal entities involved

- band members (person timelines)
- concert (shared event)
- plaza (place or place-like temporal context)
- autograph session (shared/public event)
- food truck (business/resource-like temporal entity)

## What the user should see

- a public/shared graph fragment of the band's temporal moment
- the possibility of following the singer's timeline
- another related temporal opportunity in the same place/time
- a sense that lives/activities/places connect

## What should remain hidden

- unrelated private life of band members
- private schedule beyond what is shared publicly
- internal non-public events

## What kind of convergence/conflict exists

- multiple person lines converging on a concert
- later convergence around a place
- opportunity graph emerging from public/shared fragments

## Why it validates the model

This proves:
- timeline fragments can become discovery surfaces
- feed/explore can reveal graph fragments
- places can matter as temporal contexts
- "everything connects with everything" can create meaningful discovery, not noise

---

# 9. Use Case F — Family timeline affecting professional life

## Scenario

A parent shares a family timeline with their spouse.
A school event appears on the family line.
That school event overlaps with work availability.

## Temporal entities involved

- parent 1 (person timeline)
- parent 2 (person timeline)
- family timeline
- school (organization/shared source)
- professional offering or listing

## What the parent should see

- the school event as part of the family/shared temporal context
- a conflict indicator if it affects work
- enough information to understand what is colliding
- a path to decide what to do

## What the spouse should see

- their own relation to the family event
- only work-related information if explicitly shared

## What should remain hidden

- unrelated private work details
- narrative not needed for the family context

## What kind of convergence/conflict exists

- family line influencing personal line
- personal line influencing professional availability
- possible conflict between school obligation and listing-backed work

## Why it validates the model

This proves:
- related timelines are not optional edge cases
- agreements can bridge personal and family time
- conflict rules can observe other lines
- the product is not just about appointments, but about connected life decisions

---

# 10. Use Case G — Football match vs veterinary assignment conflict

## Scenario

A player is going to a football match.
At the same time, through an agreement with a veterinary business, they get assigned a dog-grooming task.
Their conflict rules detect the collision.
They may need to cancel the match or reject/reassign the task.

## Temporal entities involved

- player (person timeline)
- football match (shared event)
- invited friends (other person timelines)
- football field (resource timeline)
- veterinary business (organization)
- assigned pet-care task (work event)
- agreement bridge between worker and veterinary business

## What the user should see

- the match as a shared event on their line
- the incoming veterinary assignment
- a visible conflict between the two
- enough context to decide

## What should remain hidden

- unrelated veterinary internal scheduling beyond what is necessary
- unrelated friend timelines
- unrelated details of other players' lives

## What kind of convergence/conflict exists

- one personal/shared event
- one work assignment
- both competing for the same time
- conflict emerges because the user's life is connected to multiple systems

## Why it validates the model

This proves:
- conflicts are first-class
- agreements create meaningful temporal bridges
- commercial and personal life collide inside one temporal grammar
- timeline is an interface for decision, not just display

---

# 11. Use Case H — Laser clinic / rented treatment rooms / resellers / independent professionals

## Scenario

A beauty business has 3 laser treatment rooms.
Two rooms include internal staff; one room is only the equipment.
The business rents blocks of time to resellers, who then resell appointments to final clients.
Independent professionals may also publish availability to work with multiple clinics.

## Temporal entities involved

- beauty clinic (organization)
- treatment rooms (resources / timelines)
- resellers (person/business timelines)
- independent professionals (person timelines)
- final clients (person timelines, private)
- treatment blocks (temporal containers)
- final appointments inside the rented block
- agreements between clinic ↔ reseller and reseller ↔ professional

## What the clinic should see

- room occupancy
- rented blocks
- operational status
- broad usage context
- no sensitive final-client details if ownership belongs to the reseller

## What the reseller should see

- rented temporal block
- final appointments inside that block
- linked independent professional if needed
- conflicts between room/professional/final appointments

## What the independent professional should see

- their assignments
- required resource/time coordination
- conflicts across multiple clinics or assignments

## What should remain hidden

- clinic should not see sensitive final-client details owned by the reseller
- unrelated reseller business data should not leak
- unrelated professional commitments should not leak unless shared

## What kind of convergence/conflict exists

- room/resource + professional + reseller block + final clients
- time is nested inside time
- multiple ownership layers
- conflict if room and professional do not align

## Why it validates the model

This proves:
- resources are temporal entities
- temporal containers are necessary
- one block of time can contain subordinate activity
- ownership and visibility can differ between layers
- the model scales to B2B + B2C mixed scheduling

---

# 12. Use Case I — Meeting room as a private timeline

## Scenario

A company creates a private timeline for a meeting room instead of treating it as a passive resource record.
Employees book it for meetings.
Shared events converge into the room line.

## Temporal entities involved

- room (resource timeline)
- employees (person timelines)
- company (organization)
- meeting event (shared node)

## What the user should see

- the room as an entity with time
- the room line behaving like any other temporal entity
- converging people on that room event
- conflict if two meetings overlap

## Why it validates the model

This proves:
- the distinction between "resource" and "timeline" can collapse productively
- resources are not second-class citizens
- anything that meaningfully occupies time can become a line

---

# 13. Cross-case truths

Across all cases above, the following truths keep appearing:

## Truth 1
The timeline owner is not always the identity shown in the node.

## Truth 2
A meaningful moment may belong to:
- a person
- a resource
- a place
- a match
- a room
- a patient
- a task
- a commercial context

## Truth 3
Shared moments do not require full visibility into all connected lines.

## Truth 4
Conflicts are part of the product's value, not an exception.

## Truth 5
Resources need their own temporal reality.

## Truth 6
Feed/Home/Explore can show graph fragments, but Timeline remains the primary temporal surface.

## Truth 7
The same visual grammar must support:
- personal life
- family life
- work
- institutions
- commercial scheduling
- hybrid arrangements

---

# 14. What these use cases imply for the product

These cases imply that VyteMerge must support:

- sparse timelines
- dense timelines
- personal/shared nodes
- resources as lines
- temporal containers
- privacy-aware intersections
- conflict resolution
- graph fragments outside Timeline
- related timeline toggles
- merged views
- ownership layers
- agreement-based visibility

Any design or implementation that cannot support these cases is too weak for the product vision.

---

# 15. Priority use cases for prototyping

Not every case should be prototyped at once.

The recommended order for UI/UX exploration is:

## Phase 1
- sparse personal timeline
- doctor/patient
- football field + invited friends

## Phase 2
- family + work conflict
- hospital + room + doctors
- meeting room as resource line

## Phase 3
- band + public graph fragments
- laser clinic / reseller / independent professionals

This allows the product to grow from:
- personal
to
- shared
to
- institutional / market-scale

---

# 16. Closing principle

> **VyteMerge Timeline is validated when it can represent not just one person's schedule, but the intersections, constraints, and opportunities that emerge when people, resources, places, and agreements all share time.**
