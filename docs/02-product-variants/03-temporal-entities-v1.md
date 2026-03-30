# Temporal Entities v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the conceptual model of **temporal entities** in VyteMerge.

It answers the question: **what things in VyteMerge have a relationship with time?**

Not just users.
Not just bookings.
Not just calendars.

VyteMerge models a network of entities that can occupy, offer, reserve, share, or conflict over time. This document names those entities, describes their temporal behavior, and explains how they relate to each other.

This is a product and conceptual document, not a backend spec. Its goal is to ensure that everyone — designers, developers, product thinkers — shares the same mental model of how time works in the system.

---

# 2. Definition of temporal entity

A **temporal entity** is anything in VyteMerge that has a meaningful relationship with time.

A temporal entity can:
- **Occupy** time (events, bookings, private commitments)
- **Offer** time (availability, slots, scheduling patterns)
- **Reserve** time (bookings made or received)
- **Share** time (visibility into another entity's temporal state)
- **Conflict** over time (overlapping commitments that require resolution)

Not every entity in VyteMerge is temporal. A post is not temporal. A reaction is not temporal. A tag is not temporal.

But a **person**, a **room**, a **football field**, a **listing**, a **family calendar**, and a **medical practice** — these are all temporal entities because their behavior depends on, produces, or consumes time.

### The key insight

Time in VyteMerge does not belong to a single owner.

Time is a shared dimension. Multiple entities can claim, observe, offer, or conflict over the same interval. The system's job is to coordinate these claims and make the relationships visible.

---

# 3. Types of temporal entities

## 3.1 Person

The most fundamental temporal entity.

A person has a Timeline — a continuous record of their temporal commitments.

A person can:
- create private events (block time)
- receive bookings (provider role)
- make bookings (consumer role)
- share their temporal state with others
- have conflicts between personal and professional time
- delegate temporal management to someone else

A person may have **multiple Timelines** (personal, professional, family) to model distinct temporal contexts without forcing everything into one stream.

## 3.2 Resource

A resource is a physical or logical thing that can be occupied over time.

Examples:
- a consultation room
- a football field
- a meeting room
- a camera or equipment
- a vehicle

A resource has its own Timeline. It can be booked, blocked, and its availability can be projected.

Resources differ from people in that:
- they do not act on their own (no agency)
- they are managed by a person or organization
- their availability is determined by scheduling rules, not by personal choice
- they can participate in capacity pools (e.g., "3 equivalent rooms")

A resource Timeline answers: "is this thing available at this time?"

## 3.3 Organization

An organization is a composite temporal entity.

It does not have a single personal timeline. Instead, it coordinates the timelines of its **members** (people) and its **resources**.

An organization:
- manages listings that consume time from its members' or resources' timelines
- delegates temporal management via Agreements (staff, secretaries)
- may have its own "operational timeline" (office hours, closures, holidays)
- coordinates capacity across staff and resources

An organization answers: "what can we offer and when?"

## 3.4 Place

A place is spatially grounded and temporally relevant.

A place has a timezone, which makes it temporally significant for scheduling.

A place may:
- have operating hours (temporal availability)
- have closures or exceptions (temporal blocks)
- host multiple resources or people simultaneously

A place is not the same as a resource. A place is "where" something happens. A resource is "what" is being used. The same place may contain multiple resources (e.g., a clinic with 5 consultation rooms).

A place answers: "what timezone and availability context applies here?"

## 3.5 Listing (offering)

A listing is a commercialized temporal entity.

A listing defines:
- what is being offered (the service)
- how it maps to time (scheduling pattern, slot duration, gaps)
- where it happens (place reference)
- whose timeline it consumes (the provider's or a resource's)

A listing produces **slots** — projected windows of bookable time.

Slots are not real events. They are projections based on the listing's scheduling rules and the current state of the underlying timeline.

A listing answers: "what can be booked, when, for how long, and at what price?"

## 3.6 Family or group

A family or group is an informal composite temporal entity.

It is not an organization (no staff, no listings, no commercial intent). It is a set of people who share temporal visibility to coordinate life.

A family timeline:
- is shared via Agreements (Sharing type)
- contains events that affect multiple members (school events, trips, appointments)
- may trigger conflict rules on members' professional timelines

A family answers: "what do we all have going on?"

---

# 4. Temporal containers

Some windows of time behave as **containers** — bounded temporal regions that hold structure inside them.

## 4.1 Event as container

A multi-hour event (e.g., a conference day, a tournament) is a temporal container. Inside it, there may be sub-events, sessions, or bookable slots.

The container defines the outer boundary. The contents define the inner structure.

## 4.2 Booking window as container

A listing may define a booking window: "bookable from March 1 to March 31." This window is a temporal container that holds all projected slots for that period.

Outside the window, no slots exist.
Inside the window, slots are generated according to the scheduling pattern.

## 4.3 Recurrence as container generator

A recurrence rule (weekly, biweekly, custom) generates a sequence of temporal containers — one per occurrence. Each container holds slots according to the day's schedule.

Exceptions modify or replace individual containers in the sequence.

## 4.4 Operating hours as container

A place or organization may define operating hours. These hours are a temporal container: availability exists inside them, not outside.

Operating hours are not the same as a scheduling pattern. They define "when the entity is open." The scheduling pattern defines "when slots exist within those hours."

---

# 5. Agreements as temporal bridges

Agreements are the mechanism by which temporal entities connect their time.

Without Agreements, each entity's time is isolated. Agreements create **bridges** that enable:
- **Visibility**: one entity can see another's temporal state
- **Coordination**: conflict rules can observe shared timelines
- **Delegation**: one entity can manage another's temporal commitments
- **Convergence**: shared events become visible as merge points

## 5.1 Sharing Agreement

Creates a one-way visibility bridge.

Entity A shares its temporal state with Entity B. Entity B can see when A is occupied (BusyOnly) or what A is doing (BusyOnlyDetails).

The bridge is read-only. B cannot modify A's time.

Use cases:
- Family calendar shared with a parent
- School timeline shared with parents
- Medical professional sharing availability with a clinic

## 5.2 Delegation Agreement

Creates a management bridge.

Entity A delegates temporal management to Entity B. B can confirm, reject, reschedule, or block time on A's behalf.

The bridge is operational. B acts on A's time with explicit permissions.

Use cases:
- Secretary managing a doctor's appointments
- Assistant managing a consultant's agenda
- Organization managing resource timelines

## 5.3 Agreement as temporal filter

An Agreement can also serve as a filter: it determines which parts of a timeline are visible and actionable for the connected party.

A Sharing Agreement with `BusyOnly` visibility creates a bridge that transmits temporal occupancy without content. The receiver knows "this time is taken" but not "by whom or why."

This is a fundamental design choice: **temporal truth can be shared without revealing the narrative behind it.**

---

# 6. Ownership and visibility

## 6.1 Temporal ownership

Every Timeline has an **Owner** — the entity whose time it represents.

Ownership means:
- the Owner controls what happens on the timeline
- the Owner decides who can see or manage it
- the Owner can revoke access at any time (immediate effect)

Ownership is not transferable. A timeline always belongs to its Owner.

## 6.2 Visibility levels

| Level | What is visible | What is hidden |
|-------|----------------|----------------|
| **None** | Nothing | Everything |
| **BusyOnly** | Time blocks (occupied intervals) | Title, description, participants, details |
| **BusyOnlyDetails** | Time blocks + tags, type, metadata | Notes, sensitive descriptions |
| **Full** (via Delegation) | Everything | Nothing (within permitted scope) |

## 6.3 Visibility is always permission-based

No temporal information leaks without an explicit Agreement.

- Following someone does not reveal their temporal state.
- Booking from someone reveals only the slot you booked, not their full timeline.
- Sharing requires an active Agreement with explicit visibility level.

## 6.4 The privacy principle

**Temporal coordination does not require full transparency.**

The system can detect conflicts, coordinate schedules, and optimize availability using only BusyOnly signals. Full transparency is a choice, not a requirement.

---

# 7. Resource timelines

Resources deserve special attention because they are temporal entities without agency.

## 7.1 What a resource timeline represents

A resource timeline is a record of when a resource is occupied, available, or blocked.

It behaves exactly like a person's timeline:
- Events occupy time
- Slots project availability
- Conflict rules detect overlaps
- Capacity pools govern concurrent usage

## 7.2 Resource capacity

A resource may have capacity > 1.

A football field has capacity 1 (one game at a time).
A classroom may have capacity 1 (one class at a time).
But a parking lot may have capacity 50 (50 cars at a time).

Capacity is evaluated per interval: how many units are occupied right now vs. how many are available.

## 7.3 Resource pools

Multiple equivalent resources may form a **pool**.

3 identical consultation rooms form a pool of capacity 3. A booking consumes one unit from the pool, not from a specific room.

The system decides which specific resource is assigned (or leaves it unassigned if it doesn't matter).

## 7.4 Resource + person coordination

A booking may require both a person and a resource.

A dentist appointment requires:
- the dentist's time (person timeline)
- the consultation room (resource timeline)

Both timelines must be available for the booking to be valid. The system checks both.

This is where temporal entities form a **network**: the booking connects two temporal entities through a shared time interval.

---

# 8. Listing/service relation to time

## 8.1 A listing is a temporal projection engine

A listing does not own time. It **projects** availability by combining:
- a scheduling pattern (when slots could exist)
- a timeline reference (whose time is consumed)
- conflict rules (what makes a slot unavailable)
- capacity (how many concurrent bookings are allowed)

The listing says: "given these rules and this timeline's current state, here are the windows where you could book."

## 8.2 Service vs listing temporal roles

A **Service** is a definition: what is offered, how long it takes, what buffers surround it.

A **Listing** is operational: when it's available, at what price, with what rules, against which timeline.

The Service defines temporal shape (duration, buffers).
The Listing defines temporal presence (scheduling, availability, conflicts).

## 8.3 Multiple listings, one timeline

Multiple listings can project slots on the same timeline.

A yoga instructor who offers both "Vinyasa" and "Restorative" classes has two listings projecting onto the same personal timeline. A booking for one affects availability for the other.

This is not a conflict — it's shared temporal resource usage.

## 8.4 Multiple timelines, one listing

A listing backed by a staff pool may project slots across multiple person timelines.

A clinic listing for "General Consultation" can be fulfilled by any of 3 doctors. The listing checks availability across all 3 timelines and shows slots where at least one doctor is free.

The booking is assigned to a specific doctor (or left unassigned), consuming time from that person's timeline.

---

# 9. Conflict and resolution

## 9.1 What is a conflict

A temporal conflict occurs when two or more claims on time overlap in a way that cannot be simultaneously fulfilled.

Types:
- **Hard conflict**: two events on the same timeline at the same time (physical impossibility)
- **Soft conflict**: two events on different timelines that the owner prefers not to overlap (e.g., personal event during work hours)
- **Cross-entity conflict**: a booking requires a person and a resource, but only one is available

## 9.2 How conflicts are detected

Conflict Rules observe timelines and detect overlaps.

A rule specifies:
- **Source**: which timeline(s) to observe
- **Target**: which slots or events are affected
- **Action**: what to do (hide slots, notify, block)

Rules can observe timelines beyond the owner's — through Sharing Agreements. This is how a school event on a family timeline can block a professional's availability.

## 9.3 How conflicts are resolved

Resolution is always human-decided. The system detects and surfaces. The user resolves.

Options:
- Cancel one of the conflicting events
- Reschedule one of the conflicting events
- Accept the conflict (override)
- Delegate resolution to someone else

The system does not auto-resolve conflicts. It highlights them and makes resolution easy.

## 9.4 Conflict as information, not punishment

A conflict is not an error. It is a signal.

The visual grammar shows conflicts clearly (red indicators, merge points, stacked nodes). But the user is never blocked from seeing or acting on their timeline because of a conflict.

Conflicts are informational. The user decides what to do.

---

# 10. Examples

## 10.1 Medical practice

**Temporal entities:**
- Dr. Pérez (person, timeline)
- Dr. López (person, timeline)
- Consultation Room A (resource, timeline)
- Consultation Room B (resource, timeline)
- The clinic (organization, coordinates all)
- Secretary María (person, delegation agreement)

**Temporal relationships:**
- The clinic has listings ("Consulta General", "Pediatría") that project onto doctors' timelines
- Each doctor's timeline is shared with the clinic (Delegation: ManageSchedule)
- Secretary María has a Delegation Agreement to manage bookings for both doctors
- Rooms are resources in a pool (capacity 2)
- A booking for "Consulta General" requires one doctor + one room
- Dr. Pérez has a personal timeline with a family event → creates a soft conflict with clinic hours
- Conflict rule: "If Dr. Pérez has a personal event tagged #family, hide his consultation slots"

**What the system does:**
María sees both doctors' timelines merged in the Timeline tab. She can confirm bookings, see which rooms are available, and notice when Dr. Pérez has a family commitment that blocks his slots — without seeing the details of the commitment.

## 10.2 Football field

**Temporal entities:**
- Cancha 1 (resource, timeline, capacity 1)
- Cancha 2 (resource, timeline, capacity 1)
- Club deportivo (organization)
- Multiple listings: "Fútbol 5", "Fútbol 7", "Entrenamiento"

**Temporal relationships:**
- Each listing projects onto one or both field timelines
- "Fútbol 5" has BookingCapacity 10 (attendees) but resource capacity 1 (one game at a time)
- A booking blocks the field for 90 minutes + 30 min buffer
- "Entrenamiento" can be booked for a field during off-peak hours only (scheduling rule)
- Two simultaneous "Fútbol 5" bookings are possible only if both fields are free

**What the system does:**
The club manager sees both field timelines in parallel. Customers see available slots per listing. A booking for one field affects the other only if they share a capacity pool.

## 10.3 Meeting room

**Temporal entities:**
- Sala Azul (resource, timeline, capacity 1)
- 5 team members (persons, timelines)
- The company (organization)

**Temporal relationships:**
- Sala Azul has a listing ("Reservar sala") available to all team members
- Each team member shares their timeline (BusyOnly) with the company
- When scheduling a meeting, the system checks: is Sala Azul free AND are the required attendees free?
- Conflict rules: if a team member has a booking, their BusyOnly signal blocks the meeting slot for that combination

**What the system does:**
A meeting organizer sees the room's availability and participants' busy/free signals. The system highlights time windows where room + all people are available. No one's private details are revealed.

## 10.4 Independent professional

**Temporal entities:**
- Laura (person, one timeline for everything)
- Multiple listings: "Sesión de fotos", "Edición de video", "Consultoría creativa"

**Temporal relationships:**
- All listings project onto Laura's single timeline
- "Sesión de fotos" takes 2 hours + 1 hour buffer
- "Edición de video" takes 3 hours, no buffer
- Both compete for the same time
- Laura has a personal event (dentist) → conflict rule hides all listing slots during that time
- Laura shares BusyOnly with her partner's family timeline

**What the system does:**
Laura sees one timeline with all her commitments. Her clients see separate listings with available slots. A booking for one service automatically reduces availability for the others. Her dentist appointment blocks everything. Her partner can see she's busy without knowing why.

## 10.5 Family timeline

**Temporal entities:**
- Mamá (person, personal + family timelines)
- Papá (person, personal + family timelines)
- Hijo (dependent, events on family timeline)
- Colegio (organization, school timeline shared with parents)

**Temporal relationships:**
- The school shares its timeline (BusyOnlyDetails) with both parents
- Family timeline is shared between both parents (full visibility)
- Mamá has a conflict rule: "If family timeline has #school event, hide my consultation slots"
- Papá has the same rule for his business listings
- A school event on Tuesday at 10am automatically affects both parents' professional availability

**What the system does:**
Both parents see the school events on their merged Timeline view. Their professional listings automatically hide slots that overlap with school obligations. The school sees nothing of the parents' professional lives. The bridge is one-directional and privacy-preserving.

---

# Closing principle

> **VyteMerge is not a booking platform between two people.
> It is a network of temporal entities — people, resources, places, organizations, and offerings — connected through Agreements, coordinated through conflict rules, and made visible through a shared temporal grammar.**
