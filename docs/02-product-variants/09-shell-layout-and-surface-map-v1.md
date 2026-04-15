# Shell Layout and Surface Map v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **top-level shell experience** of VyteMerge.

It explains:

- what the main surfaces are
- how they relate to each other
- what belongs to the shell vs individual MFEs
- how users should move through the system
- what the main navigation should communicate
- how the product should feel at first entry
- what pages are minimally required to make the concept understandable

This is a product and UX document.
It is not a backend or implementation spec.

Its goal is to prevent the shell from becoming:
- a neutral frame around disconnected MFEs
- a random set of routes
- a technically valid but conceptually weak entrance to the product

---

# 2. Shell principle

## The shell owns the product story

The shell is not a passive container.

The shell must decide:
- what the top-level surfaces are
- how they are named
- how they are entered
- how they relate to one another
- what a first-time user should understand
- where the user goes next

### Rule
**The shell tells the story.  
MFEs implement the surfaces.**

This means:
- the shell is responsible for coherence
- top-level navigation is a shell decision
- Home composition is a shell concern
- route semantics are a shell concern
- empty-state meaning at the product level is a shell concern

---

# 3. Core top-level surfaces

VyteMerge should currently be organized around these top-level surfaces:

1. **Home**
2. **Timeline**
3. **Studio**
4. **Profile** (via avatar)
5. **Explore** (sub-surface / search-first entry, not a competing top-level concept)

---

# 4. Surface map

## 4.1 Home

### Purpose
My line + the things that connect to it.

### What it should communicate
- the product is alive
- there are things happening
- some of those things matter to me
- I already have a line of life/time
- I can discover, reserve, save, or add things to my timeline

### Role in the system
Home is:
- a top-level destination
- the entry surface for most users
- the bridge between discovery and life/time
- not a pure feed
- not a pure dashboard
- not the full Timeline

---

## 4.2 Explore

### Purpose
Search-first explicit discovery.

### What it should communicate
- I can look for people, businesses, services, or opportunities
- I can refine if I need to
- this is the place for intentional discovery

### Role in the system
Explore is:
- not a competing top-level concept with Home
- not a giant discovery dashboard
- a search-first sub-surface with its own route
- a place the shell should make reachable, but not over-emphasize as a separate product pillar

---

## 4.3 Timeline

### Purpose
My line of life/time, shared moments, and temporal decisions.

### What it should communicate
- this is my temporal reality
- this is where things converge
- this is where I notice and resolve conflicts
- this is not a calendar clone

### Role in the system
Timeline is:
- a top-level destination
- a differentiator of the product
- a place for understanding time deeply
- not just an operational list
- not a generic agenda

---

## 4.4 Studio

### Purpose
Provider/commercial workspace.

### What it should communicate
- this is where I build and manage what I offer
- this is my commercial/work side
- this is not a generic admin panel

### Role in the system
Studio is:
- a top-level destination
- provider/business-oriented
- clearly separated from Home and Timeline
- the place where offerings, rules, and provider workflow live

---

## 4.5 Profile

### Purpose
Identity + activity + offer + reputation.

### What it should communicate
- who this person/business is
- what they do
- what they offer
- why I should care

### Role in the system
Profile is:
- entered through avatar or direct URLs
- not a tab competing with Home/Timeline/Studio
- a deep identity surface
- both personal and social/commercial

---

# 5. Navigation model

## 5.1 Main navigation model

### Top-level shell navigation should be:

- **Home**
- **Timeline**
- **Studio**
- **Profile via avatar**
- **Search / Explore entry** in top nav

This matches the mental model:

- Home = my connected world
- Timeline = my time
- Studio = my offering/workspace
- Profile = who I am / who others are
- Explore = search-first discovery layer

---

## 5.2 Why not separate "Discover" as a main concept

The shell should avoid exposing three competing discovery ideas:
- Home
- Discover
- Explore

That creates conceptual noise.

### Decision
- Keep **Home** as main surface
- Keep **Explore** as explicit search-first sub-surface
- "Discover" can remain an internal/domain naming if useful, but should not dominate user-facing information architecture

---

## 5.3 Primary navigation hierarchy

### Level 1 — Product pillars
- Home
- Timeline
- Studio

### Level 2 — Identity / utility
- Profile
- Notifications
- Search / Explore

### Level 3 — Internal feature routes
- search results
- listing detail
- profile tabs
- timeline expanded views
- studio subpages

This hierarchy prevents:
- top nav overload
- too many competing meanings
- disconnected product feeling

---

# 6. Global shell layout

## 6.1 Mobile shell

### Structure
- top nav
- main content area
- bottom tab bar

### Top nav
Should contain:
- product mark/logo
- search entry
- notifications
- avatar/profile entry

### Bottom tabs
Should contain:
- Home
- Timeline
- Studio

### Why
On mobile, the product should feel:
- focused
- thumb-friendly
- simple
- obvious within seconds

---

## 6.2 Desktop shell

### Structure
The desktop shell may evolve into one of these patterns:

#### Option A — top nav + left rail + content
Good if:
- you want more persistent navigation
- you want timeline/glimpse modules
- you want more desktop structure

#### Option B — top nav + centered content + contextual side rail
Good if:
- Home remains content-first
- Timeline glimpse or contextual modules live on the side
- desktop needs more breathing room without becoming a dashboard

### Recommended direction
For now:
- keep desktop simple
- do not overcommit to a heavy app-shell dashboard
- use side content only when it supports the current surface meaning

---

# 7. Shell-owned elements

The shell should own:

- top nav
- bottom nav / tab bar
- global search entry
- notification entry
- profile entry
- route semantics
- active tab logic
- branded 404
- product-level empty state framing
- Home top-level composition
- user-state entry logic (new vs active user)
- shell-level mock narrative

The shell should NOT try to own:
- every inner feature interaction
- detailed feature rendering
- local component logic inside each MFE
- domain-specific microflows

---

# 8. MFE-owned responsibilities

MFEs should own:

- detailed rendering of surfaces
- internal routes and tabs
- feature-specific empty states
- feature interactions
- local mock data where appropriate
- surface-specific experimentation

### Examples
- discover/social MFE:
  - Home blocks rendering
  - Explore result rendering
  - graph fragments/cards
- timeline/bookings MFE:
  - timeline line rendering
  - node behaviors
  - related timelines
  - merged view
- provider MFE:
  - Studio pages
  - offering/listing pages
- profile MFE:
  - profile page
  - tabs and identity blocks

---

# 9. First-time user vs active user entry

## 9.1 First-time user

### Shell responsibility
The shell should help the first-time user understand:
- what this product is
- what Home is for
- what can be discovered
- that time matters here
- that there is more than just a feed

### Recommended first-entry behavior
Home should be:
- discovery-first
- alive
- connected
- easy to read
- not empty

It should avoid:
- giant featureless blank states
- pure timeline complexity
- too many controls
- provider-heavy wording

---

## 9.2 Active user

### Shell responsibility
The shell should help the active user:
- re-enter their world quickly
- see their line/context
- continue their next action
- move between Home and Timeline naturally

### Recommended active-state behavior
Home can lean more into:
- my line
- my next moments
- connected opportunities
- relevant activity
- contextual discovery

---

# 10. Page set required to complete the concept

You do NOT need every final production page yet.
You need the **minimum page set** required so the product becomes understandable.

## Required page set

### 10.1 Home page
Must exist and feel real.

Needs:
- life/timeline glimpse
- connected opportunities
- discovery blocks
- activity/content
- clear entry to Timeline and Explore

### 10.2 Explore page
Must exist and feel simple.

Needs:
- search-first entry
- progressive refinement
- believable result rendering
- temporal hints where useful

### 10.3 Timeline page
Must exist and feel differentiated.

Needs:
- line of life/time
- nodes
- sparse/dense examples
- conflict possibility
- at least one shared moment idea

### 10.4 Studio home/workspace page
Must exist and feel provider-oriented.

Needs:
- overview of offering/workspace
- clear create/manage actions
- not a generic admin dump

### 10.5 Profile page
Must exist and feel identity-rich.

Needs:
- who this entity is
- activity
- offer
- follow/reserve relation

### 10.6 Branded 404
Must exist so the system never feels broken.

---

# 11. Surface relationship map

## Home ↔ Timeline
Home previews and contextualizes.
Timeline deepens and operationalizes.

## Home ↔ Explore
Home exposes interesting things.
Explore lets me search intentionally.

## Home ↔ Profile
Home can surface identity-rich moments and profiles.
Profile lets me understand the actor deeply.

## Home ↔ Studio
Home may inspire provider action, but Studio is where provider work lives.

## Timeline ↔ Studio
Timeline may reflect provider commitments and conflicts.
Studio defines what the provider offers.

## Timeline ↔ Profile
Timeline fragments may enrich profile.
Profile may expose temporal hints, but Timeline remains the primary temporal surface.

---

# 12. Navigation copy principles

The shell should use words that feel:
- simple
- direct
- human
- product-coherent

Avoid:
- too much internal domain language
- too many synonyms for similar things
- mixing "discover", "explore", "feed" as if they were equal pillars
- admin/product jargon

### Suggested shell-level vocabulary
- Home
- Timeline
- Studio
- Buscar / Explorar
- Perfil
- Notificaciones

---

# 13. Empty-state principles at shell level

The shell should enforce these truths:

## Home
Never feels empty in a dead way.
If there is no real data, mock/curated content must still tell the right story.

## Timeline
Empty state must reinforce temporal identity, not generic marketplace discovery.

## Studio
Empty state should invite creation and management.

## Profile
Empty state should still feel like an identity surface, not a blank shell.

## 404
Should feel product-native, not broken/dev-like.

---

# 14. Current stage design rule

## Do not prioritize toolkit normalization yet

At this stage:
- page meaning matters more than reusable component purity
- layout and surface truth matter more than design system discipline
- using the existing stack is enough
- normalization comes later

### Practical rule
Use the technologies already in use.
Do not add unnecessary new tech.
Do not let toolkit constraints stop page-level conceptual progress.

---

# 15. Recommended execution order after this document

## Phase 1
Shell/Home/Explore map
- route semantics
- Home/Explore relationship
- global navigation logic

## Phase 2
Home page concept
- my line + connected things
- not pure feed
- believable first entry

## Phase 3
Explore page simplification
- search-first
- progressive refinement
- no rigid structure

## Phase 4
Timeline page concept
- line of life/time
- nodes
- shared moments
- conflict hints

## Phase 5
Studio/Profile concept completion
- provider workspace
- identity-rich profile

This sequence keeps the shell grounded while the Timeline model continues maturing.

---

# 16. Decisions

## Decision 1
**The shell owns the product story.**

## Decision 2
**The main pillars are Home, Timeline, and Studio.**

## Decision 3
**Profile is entered via avatar and direct route, not as a competing main tab.**

## Decision 4
**Explore exists as a search-first sub-surface, not as a competing top-level pillar.**

## Decision 5
**The shell must be meaningful even with mocked data.**

## Decision 6
**The minimum page set should be built to make the concept understandable before deep backend work.**

## Decision 7
**Page-first, toolkit-later is the correct rule for this stage.**

---

# 17. Closing principle

> **VyteMerge Shell should make the product understandable before it makes it complete: the user should quickly feel what Home, Timeline, Studio, Explore, and Profile are for, and how they belong to one connected system.**
