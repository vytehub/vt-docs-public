# UI/UX Roadmap v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **practical roadmap** for the current UI/UX-only stage of VyteMerge.

Its purpose is to answer:

- what the team should design first
- what must be deferred
- how to translate the current documentation into action
- what kind of cycles should run next
- how Claude Code should be guided
- how stakeholder validation should happen
- when the team can move from concept-shaping to implementation depth

This document is deliberately operational.
It sits after the conceptual documents and before feature-heavy execution.

---

# 2. Current stage summary

VyteMerge already has:

- a clarified product skeleton
- a clearer definition of Home, Explore, Timeline, Studio, and Profile
- a documented timeline grammar
- a temporal entities model
- privacy/visibility rules
- timeline interaction rules
- timeline prototyping rules
- a shell/surface map
- a minimum page map

What is still missing is not more abstract theory.
What is missing is:

- a deliberate execution order
- tighter focus
- clear cycle boundaries
- a way to stop drifting back into backend-first work

---

# 3. Stage goal

## Main goal of this stage

**Make VyteMerge feel like a coherent product before deep backend integration.**

This means:

- the shell should make sense
- the main pages should exist
- the navigation should feel natural
- Home should feel alive
- Explore should feel simple
- Timeline should feel unique
- Studio should feel like a workspace
- Profile should feel identity-rich

The product must become understandable as a system.

---

# 4. What this stage is NOT about

This stage is NOT primarily about:

- real backend completeness
- final API contracts
- feature-depth completion
- design system normalization
- polishing every edge case
- building every subpage
- advanced moderation
- final messaging
- analytics maturity

This stage is about:
- shape
- coherence
- navigation
- page meaning
- visual/product truth

---

# 5. Roadmap phases

The roadmap should progress through 4 practical phases:

## Phase 1 — Shell foundation
Focus:
- navigation
- layout
- top-level coherence
- user entry experience

## Phase 2 — Core surface shaping
Focus:
- Home
- Explore
- Timeline
- Profile
- Studio Home

## Phase 3 — Timeline differentiation
Focus:
- temporal identity
- sparse/dense behavior
- slot vs event
- shared moments
- conflict presentation
- home glimpse

## Phase 4 — Normalization readiness
Focus:
- patterns worth keeping
- what should become reusable
- what should move into toolkit later
- what needs backend next

---

# 6. Immediate execution order

## Step 1
Shell layout and navigation

## Step 2
Home concept page

## Step 3
Explore search-first page

## Step 4
Timeline prototype page

## Step 5
Profile concept page

## Step 6
Studio Home concept page

## Step 7
Notification and support surfaces

This is the recommended order because it follows the user's path of understanding:
- what is this?
- where do I go?
- how do I discover?
- how does time work?
- who are these actors?
- where is my business/work side?

---

# 7. Recommended cycle structure

Each cycle in this stage should follow these rules:

## Rule 1 — One visible UX problem per cycle
Examples:
- "The shell does not communicate the product clearly"
- "Home still feels like a generic feed"
- "Explore is too rigid"
- "Timeline still reads as a planner/calendar"
- "Profile still feels too empty"

## Rule 2 — Maximum scope
Per cycle:
- max 1–2 repos
- max 3 meaningful tasks
- no backend dependency unless absolutely necessary for a visible outcome

## Rule 3 — Mock-first
Every cycle should be executable with believable mocks.

## Rule 4 — Refinement stays inside the same cycle
If the first pass is ugly or conceptually weak:
- refine inside the same cycle
- do not declare it done too early

## Rule 5 — Stakeholder review is mandatory
The cycle is not complete until the visible result is reviewed.

---

# 8. Cycle roadmap — proposed next cycles

## Cycle A — Shell foundation
### Goal
User can understand the main structure of VyteMerge through the shell.

### Includes
- top nav
- bottom tabs
- search entry
- avatar/profile entry
- active states
- 404 branded fallback
- relationship between Home / Timeline / Studio / Profile / Explore

### Does NOT include
- deep Home polish
- timeline complexity
- provider subpages
- backend work

### Success condition
The product no longer feels like disconnected MFEs.

---

## Cycle B — Home as “my line + connected things”
### Goal
Home no longer feels like a generic feed or a random collage.

### Includes
- life/timeline glimpse
- connected opportunities
- public/shared activity fragments
- believable first viewport
- clear entry into Explore and Timeline

### Does NOT include
- final recommendation logic
- full graph complexity
- deep booking flow

### Success condition
The user immediately understands what Home is for.

---

## Cycle C — Explore search-first
### Goal
Explore feels natural and simple, not rigid.

### Includes
- strong search entry
- All results by default
- simple result model
- progressive refinement
- temporal hints only where useful

### Does NOT include
- advanced graph browsing
- too many tabs
- too many filter systems

### Success condition
The user feels they can search naturally without learning a custom UX.

---

## Cycle D — Timeline prototype
### Goal
Timeline clearly feels different from a generic calendar.

### Includes
- sparse timeline
- dense timeline
- slot vs event
- one shared moment
- one conflict
- moving avatar if useful
- one resource-related scenario

### Does NOT include
- final institutional merge views
- backend-complete timeline engine
- final graph view

### Success condition
The user can explain why Timeline is special.

---

## Cycle E — Profile as identity surface
### Goal
Profile clearly communicates who the actor is, what they do, and why they matter.

### Includes
- identity block
- activity fragment
- offering fragment
- follow/reserve relation
- temporal hint if useful

### Success condition
The user feels they understand the actor.

---

## Cycle F — Studio Home
### Goal
Studio clearly feels like a provider workspace.

### Includes
- overview of what I offer
- create/edit/publish entry points
- status/pending request view
- provider-specific empty state

### Success condition
Studio no longer feels like a vague placeholder or admin shell.

---

# 9. What to defer intentionally

The team should intentionally postpone:

- deep moderation flows
- real recommendation engines
- final social feed algorithm depth
- final conflict engine behavior
- full institutional scheduling complexity
- deep profile subpages
- messaging system
- advanced analytics
- toolkit normalization
- backend-perfect booking orchestration

These can return later.
Right now they are distractions from product-shape work.

---

# 10. Prompting strategy for Claude Code

This stage should not rely only on `/start`.

Use a hybrid model:

## Use direct prompting when:
- the team is reshaping a surface
- product direction still needs steering
- visual quality must be corrected before code spreads
- the user is refining concept or navigation

## Use orchestrated execution when:
- the cycle goal is already clear
- docs are already aligned
- the surface direction is accepted
- the work is mainly execution rather than conceptual steering

### Rule
If the product direction is still moving:
- prefer direct prompting

If the direction is stable:
- use `/start` or cycle execution

---

# 11. Suggested prompting mode by phase

## Phase 1 — Shell foundation
Use:
- direct prompts
- design steering
- stakeholder review before merge

## Phase 2 — Core surfaces
Use:
- direct prompts + targeted execution
- possibly cycle-by-cycle execution
- no large autonomous batch runs yet

## Phase 3 — Timeline differentiation
Use:
- very guided prompting
- visual prototypes
- concept review before merge

## Phase 4 — Normalization readiness
Use:
- more structured execution
- start identifying reusable patterns
- begin preparing toolkit normalization

---

# 12. Working agreement for this stage

## Agreement 1
Do not optimize for backend truth before product truth.

## Agreement 2
Do not force toolkit usage if it blocks page meaning.

## Agreement 3
Use the existing stack only.
No unnecessary new technologies.

## Agreement 4
Prefer pages and surfaces over premature abstraction.

## Agreement 5
Refine ugly work before declaring victory.

## Agreement 6
Keep the shell coherent even while pages are still mocked.

## Agreement 7
Use realistic mock data to tell the right story.

---

# 13. Validation framework

Every cycle should be validated on 3 levels:

## 13.1 Conceptual validation
Does this surface express the intended product truth?

## 13.2 Visual validation
Does it look coherent, readable, and product-like?

## 13.3 Behavioral validation
Can the user understand what to do next?

If any of these fail, the cycle is not complete.

---

# 14. Stakeholder review template

For each cycle, stakeholder review should answer:

1. Do I understand what this surface is for?
2. Does this feel like VyteMerge?
3. Does this page help tell the product story?
4. Is the primary CTA correct?
5. Is the page too generic, too rigid, or too confusing?
6. Does it help me see how time, discovery, and life connect?

This keeps review focused on product truth, not random taste.

---

# 15. Exit criteria for this stage

The current UI/UX-only shaping stage can be considered mature enough to move on when:

- shell navigation is coherent
- Home clearly expresses “my line + connected things”
- Explore is simple and search-first
- Timeline clearly feels like a line of life/time
- Profile and Studio are understandable as surfaces
- the minimum page set feels like one product
- mock data is no longer hiding conceptual confusion
- the team can identify which patterns are stable enough to normalize

Only then should the team push strongly into:
- toolkit normalization
- deeper backend integration
- more autonomous execution

---

# 16. Final recommendation

The next best move is not “more theory” and not “more backend.”

The next best move is:

1. shell foundation
2. Home
3. Explore
4. Timeline
5. Profile
6. Studio

one cycle at a time,
with stakeholder review after each one.

That is the cleanest path from concept to a believable product.

---

# 17. Closing principle

> **VyteMerge should now be built surface by surface, starting from shell and moving into Home, Explore, and Timeline, until the user can feel one coherent product before any deep backend truth is required.**
