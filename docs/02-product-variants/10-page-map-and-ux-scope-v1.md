# Page Map and UX Scope v1 — VyteMerge

## Estado

Draft v1

---

# 1. Purpose

This document defines the **minimum page set** needed for VyteMerge to become understandable as a product.

Its purpose is to answer:

- what pages must exist now
- what each page is for
- what each page should show
- what each page should NOT try to solve yet
- what mock data is needed
- what user action each page should enable
- what belongs in this stage and what should be deferred

This is a product and UX planning document.
It is not a backend or implementation spec.

Its role is to prevent the team from:
- building too many pages too early
- skipping critical surfaces
- solving the wrong problems first
- losing the product story in isolated MFE work

---

# 2. Core rule of this phase

## Build the minimum page set required to make the concept understandable

This phase is NOT about:
- finishing all features
- wiring backend end-to-end
- normalizing components
- polishing everything to final quality

This phase IS about:
- making the product legible
- making navigation coherent
- making each surface understandable
- making the product feel alive
- validating the concept through believable pages

---

# 3. Product page map — current minimum set

The minimum page set for this phase is:

1. **Home**
2. **Explore**
3. **Timeline**
4. **Studio Home**
5. **Profile**
6. **Notifications / lightweight inbox**
7. **404 / broken route fallback**

Optional but useful after the minimum is working:
8. **Search results detail / dedicated search result state**
9. **Shared event / moment detail**
10. **Basic listing detail**
11. **Basic resource/place detail**

---

# 4. Page 1 — Home

## Purpose

Home is:
# **my line + the things that connect to it**

It is the most important first-entry page of the product.

Home should answer:
- what is happening
- what matters to me
- what is coming next
- what can I add to my life
- what can I reserve, save, or follow

## What Home should show

Minimum:
- timeline/life glimpse
- 2–4 upcoming meaningful moments
- connected opportunities
- public/shared activity fragments
- suggested profiles/businesses/services
- one clear entry to Explore
- one clear entry to full Timeline

## What Home should NOT try to solve yet

- full timeline management
- complete graph browsing
- deep provider management
- perfect personalization
- real backend scoring
- complete recommendation engine

## Primary CTA
- Reservar
- Agregar a mi timeline
- Ver Timeline
- Explorar más
- Seguir

## Mock data needed
- upcoming nodes
- connected opportunities
- public activity fragments
- suggested profiles/businesses
- at least one promoted/commercial item that still feels relevant

## Success condition
The user opens the product and immediately feels:
- there is life here
- this is connected to time
- this is not a generic feed
- I know where to go next

---

# 5. Page 2 — Explore

## Purpose

Explore is the explicit discovery/search surface.

It should answer:
- how do I look for something?
- how do I discover intentionally?
- how do I move from curiosity to action?

## What Explore should show

Minimum:
- strong search entry
- results in “All” by default
- results for people/businesses/services/opportunities
- lightweight temporal hints
- progressive refinement (tags/filters later, not heavy upfront)

## What Explore should NOT try to solve yet

- advanced graph browsing
- many tabs with rigid segmentation
- heavy filtering system
- institutional search complexity
- backend-perfect indexing logic

## Primary CTA
- Buscar
- Refinar
- Abrir resultado
- Reservar
- Agregar a mi timeline
- Guardar

## Mock data needed
- mixed realistic search results
- temporal hints where useful
- believable entities across categories

## Success condition
The user feels:
- this is simple
- I can search naturally
- I don't need to learn a custom system before using it

---

# 6. Page 3 — Timeline

## Purpose

Timeline is the product’s deepest and most differentiating surface.

It should answer:
- what is my life/time line?
- what is next?
- what is shared?
- what conflicts?
- what is just a slot and what is a real event?

## What Timeline should show

Minimum:
- personal line
- sparse and dense examples
- slots vs events
- at least one shared event
- at least one conflict example
- moving avatar / current position if it helps
- one related line or resource glimpse (lightly, not overwhelming)

## What Timeline should NOT try to solve yet

- full institutional merged view
- full 3D/graph exploration
- final advanced gestures
- all privacy permutations in UI richness
- complete conflict engine

## Primary CTA
- Ver detalle
- Resolver conflicto
- Crear evento
- Abrir origen del momento
- Ver línea relacionada
- Expandir

## Mock data needed
- sparse life example
- dense life example
- slot
- event
- shared event
- resource-related event
- conflict scenario

## Success condition
The user feels:
- this is not a calendar clone
- this is my line of life/time
- I understand what makes this different

---

# 7. Page 4 — Studio Home

## Purpose

Studio is the provider/commercial workspace.

It should answer:
- what can I create?
- what am I offering?
- what is pending?
- how do I manage my side of the business?

## What Studio Home should show

Minimum:
- overview of what I offer
- create/edit/publish entry points
- pending requests or pending work
- basic operational summary
- status of active/inactive offerings

## What Studio should NOT try to solve yet

- full business ERP
- full analytics dashboards
- all settings and policy screens
- deep admin complexity

## Primary CTA
- Crear
- Editar
- Publicar
- Ver solicitudes
- Configurar disponibilidad

## Mock data needed
- one or more offerings
- one pending request
- one draft/published state
- one empty-state variant for new providers

## Success condition
The user feels:
- this is my workspace
- I know where provider work lives
- this is different from Home and Timeline

---

# 8. Page 5 — Profile

## Purpose

Profile is identity + activity + offer + reputation.

It should answer:
- who is this?
- what do they do?
- what do they offer?
- why should I follow or reserve?

## What Profile should show

Minimum:
- identity block
- activity fragment
- offerings/listings fragment
- follow state
- relationship to this profile
- temporal hint if useful

## What Profile should NOT try to solve yet

- every social subpage
- every moderation setting
- full graph exploration
- fully mature reputation systems

## Primary CTA
- Seguir
- Ver actividad
- Ver offerings
- Reservar
- Explorar más

## Mock data needed
- believable identity
- sample activity
- sample offer
- some social signals

## Success condition
The user feels:
- I understand this actor
- I understand why this profile matters in the ecosystem

---

# 9. Page 6 — Notifications / lightweight inbox

## Purpose

A lightweight place for:
- attention-worthy events
- confirmations
- requests
- reminders
- relevant changes

## What it should show

Minimum:
- booking-related signals
- timeline-relevant signals
- follow/social signals if helpful
- simple read/unread structure

## What it should NOT try to solve yet

- full messaging system
- multi-thread communication
- advanced notification center
- complex filtering

## Primary CTA
- Open related item
- Open Timeline
- Open request
- Mark as read

## Mock data needed
- 4–8 believable notification items
- at least one temporal alert
- at least one social/commercial alert

## Success condition
The user feels:
- the product is alive
- meaningful things come back to me here

---

# 10. Page 7 — 404 / broken route fallback

## Purpose

Prevent the product from feeling broken.

## What it should show

Minimum:
- branded error state
- human copy
- clear way back to Home

## Primary CTA
- Volver al inicio

## Success condition
Even an error route still feels like VyteMerge.

---

# 11. Optional next pages after the minimum set

Once the minimum set feels coherent, the next useful pages would be:

## 11.1 Search result detail
A slightly richer result state after search.

## 11.2 Shared moment detail
A focused page/sheet for:
- event
- participants
- resource
- relation
- conflict context

## 11.3 Listing detail (basic)
Enough to support:
- reserve
- understand offer
- see temporal hints

## 11.4 Resource/place detail
Useful later for:
- field
- room
- plaza
- venue
- clinic room

These should come after the core surfaces are already coherent.

---

# 12. Surface-to-page relationship

## Home → Explore
Home introduces, Explore deepens discovery.

## Home → Timeline
Home previews life/time, Timeline explains and enables action.

## Home → Profile
Home surfaces identity-rich fragments, Profile deepens the actor.

## Home → Studio
Home may contain provider-adjacent signals, Studio is where provider work happens.

## Timeline → Shared moment / listing / profile
Timeline is where moments can branch into richer detail.

## Explore → result → reserve/add to timeline
Explore supports intentional discovery and commitment.

---

# 13. What belongs in this phase

This phase includes:
- page composition
- navigation clarity
- mock-first believable content
- top-level CTA logic
- product vocabulary
- temporal hints
- page hierarchy
- entry flows
- shell-level story

This phase does NOT include:
- final backend integration
- feature-depth completion
- component normalization
- toolkit-first discipline
- production-ready analytics
- full messaging
- full moderation
- complete admin complexity

---

# 14. UX scope by page

## Home
Understand the product emotionally and contextually.

## Explore
Search and intentional discovery.

## Timeline
Understand and act on time.

## Studio
Work and offer management.

## Profile
Understand the actor.

## Notifications
Return to what matters.

## 404
Never feel broken.

---

# 15. Mock data scope by page

## Home
Needs the richest story.

## Explore
Needs believable search result variety.

## Timeline
Needs the most conceptually precise mock set.

## Studio
Needs just enough to feel like a workspace.

## Profile
Needs just enough to feel identity-rich.

## Notifications
Needs just enough to feel alive.

---

# 16. Priority order for page prototyping

If building this in order, the recommended sequence is:

## Step 1
Home

## Step 2
Explore

## Step 3
Timeline

## Step 4
Profile

## Step 5
Studio Home

## Step 6
Notifications

## Step 7
404 polish / global fallback polish

### Why this order
Because it follows the user's first understanding path:
- what is this?
- how do I discover?
- how does time work?
- who are these actors?
- where is my provider side?
- how does the system stay alive?

---

# 17. Design decisions

## Decision 1
**The minimum page set is enough to make VyteMerge understandable without full backend depth.**

## Decision 2
**Home is the emotional/product entry.**

## Decision 3
**Explore is the intentional discovery entry.**

## Decision 4
**Timeline is the conceptual differentiator.**

## Decision 5
**Studio is the provider workspace.**

## Decision 6
**Profile is identity + activity + offer.**

## Decision 7
**Notifications and 404 are supporting pages, but still important for product feel.**

## Decision 8
**Page-level truth matters more than component-level normalization in this phase.**

---

# 18. Closing principle

> **The product becomes believable when the minimum set of pages already tells one coherent story: what matters to me, what I can discover, how my time works, who these actors are, and where my own commercial/work side lives.**
