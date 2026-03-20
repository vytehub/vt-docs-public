---
title: Flow 18 — Home/Feed Experience (Discover, Search, Book)
description: >
  El usuario accede al tab Home y scrollea un feed vertical que mezcla posts sociales,
  listing cards con CTA de booking, suggest cards contextuales, y contenido promocionado.
  Puede buscar con la search bar progresiva (texto → tags → filtros), hacer booking
  directamente desde el feed via bottom sheet, y descubrir contenido nuevo via carruseles
  temáticos. Para usuarios nuevos, un cold-start feed muestra contenido popular y cercano.
status: draft
version: 1
last-reviewed: 2026-03-20
---

# Flow 18 — Home/Feed Experience (Discover, Search, Book)

## 1. Summary

| | |
|---|---|
| **Goal** | Que el usuario descubra contenido, servicios y providers en un feed social unificado con capacidad de booking inline, search progresivo, y recomendaciones contextuales. |
| **Actors** | Usuario autenticado (consumer, provider, o ambos); Usuario no autenticado (lectura limitada — NEEDS-CLARIFICATION) |
| **Out of scope** | Booking flow completo (Flow 02), creación de posts (Flow 03), listing creation (Flow 17), Timeline management (Flow 16), moderación (Flow 14) |
| **Surfaces** | `vt-discover-mfe`: `/home` (feed), `/home/search` (search active), `/home/search/results` (results) |
| **Decision ref** | ADR-0005 (Navigation Architecture — Home es tab 1) |

---

## 2. Domain Context

### El Home no es un marketplace — es un feed social con booking integrado

El Home de VyteMerge combina dos mundos que normalmente están separados:

| Feed social puro (Instagram) | Marketplace puro (Booksy) | Home VyteMerge |
|---|---|---|
| Posts, fotos, stories | Búsqueda → proveedor → servicio → reservar | Posts **y** listings en un solo scroll |
| Engagement: likes, comments | Engagement: book, review | Engagement: like, comment **y** book |
| Descubrimiento por follow graph | Descubrimiento por búsqueda | Descubrimiento por follow graph **+** suggest **+** search |
| Sin acciones comerciales inline | Todo es comercial | Acciones comerciales **opcionales** e inline |

> **Principio de diseño (Xiaohongshu pattern):** El feed debe sentirse como una experiencia social que *incidentalmente* tiene servicios reservables — no como un catálogo con features sociales.

### Tipos de contenido en el feed

| Tipo | Source | CTA | Card type |
|---|---|---|---|
| Social post (`TEXT`, `MEDIA`, `LINK`) | Social module — seguidos | Like, Comment, Share | Social Post Card |
| Listing share (`LISTING_SHARE`) | Social module — seguidos | Book, Save | Listing Card with booking CTA |
| Listing card (discovery) | Suggest/Discovery | Book, Save | Listing Card with booking CTA |
| Suggest card | Suggest module | Action varies | Suggest Card |
| Promoted listing | Campaign module | Book, Save | Listing Card with "Promoted" label |
| Discovery carousel | Discovery module | Browse more | Carousel (horizontal scroll) |

### Content mixing ratio (target)

Basado en research de TikTok, LinkedIn, Pinterest:

| Segment | Followed content | Suggested/Discovery | Promoted |
|---|---|---|---|
| Usuario con follows | 60% | 25% | 15% |
| Usuario nuevo (cold start) | 10% (trending) | 75% (location + category) | 15% |

El ratio se ajusta dinámicamente (PID controller pattern de Pinterest) basado en engagement del usuario.

---

## 3. Preconditions

| # | Condición |
|---|---|
| PRE-01 | Usuario autenticado (o modo lectura limitada si aplica) |
| PRE-02 | Ubicación disponible (GPS o configurada) para sugerencias geo-basadas |
| PRE-03 | Feed algorithm tiene al menos contenido trending/popular (cold start no depende de follows) |

---

## 4. Trigger

El usuario abre la app (Home es tab default) o toca el tab **Home** en la barra de navegación inferior (1er tab, ADR-0005).

---

## 5. Main Flow

### Capability A — Feed Principal (Scroll Vertical)

El feed es un **scroll vertical infinito** con cards de diferentes tipos mezcladas algorítmicamente.

#### Layout del feed

```
┌─────────────────────────────────────────┐
│  [Logo]         [🔍]    [🔔]  [Avatar]  │
├─────────────────────────────────────────┤
│                                         │
│  ┌─ Discovery Carousel ──────────────┐  │
│  │ [Near Me] [This Week] [Popular] → │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌─ Social Post Card ────────────────┐  │
│  │ [Avatar] @dra.garcia    [Follow]  │  │
│  │ [Foto: consultorio nuevo]         │  │
│  │ ♡ 42  💬 5  ↗ Share    🔖 Save   │  │
│  │ "¡Inauguramos el consultorio..."  │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌─ Listing Card (bookable) ─────────┐  │
│  │ [Foto: masaje]          [🔖 Save] │  │
│  │ Masaje Relajante 60 min           │  │
│  │ [Ava] Studio Zen  ★ 4.8 (56)     │  │
│  │ 📍 1.2 km  ⏱ 60 min              │  │
│  │         [Desde $8000 · Reservar]  │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌─ Suggest Card ────────────────────┐  │
│  │ ✦ Suggested for you              │  │
│  │ "Tenés 45 min libres mañana.     │  │
│  │  Hay disponibilidad cerca:"       │  │
│  │ [Mini card: Yoga · $3000 · Book]  │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌─ Listing Card (promoted) ─────────┐  │
│  │ Promoted                          │  │
│  │ [Foto: spa]             [🔖 Save] │  │
│  │ Spa Day Completo                  │  │
│  │ [Ava] Harmony Spa  ★ 4.6 (120)   │  │
│  │ 📍 3.5 km  ⏱ 120 min             │  │
│  │         [Desde $15000 · Reservar] │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ... (infinite scroll) ...              │
│                                         │
└─────────────────────────────────────────┘
```

#### Card types — anatomía detallada

**Social Post Card** (posts de seguidos: TEXT, MEDIA, LINK)

```
┌───────────────────────────────────────────┐
│  [Avatar] @username              [Follow]  │
│  [···] (overflow menu)                     │
├───────────────────────────────────────────┤
│  [Full-width image / carousel]             │
├───────────────────────────────────────────┤
│  [♡ Like] [💬 Comment] [↗ Share]  [🔖 Save]│
│  "42 likes"                                │
│  "Caption text preview..."      [more]     │
│  "2 hours ago"                             │
└───────────────────────────────────────────┘
```

**Listing Card** (servicio reservable — orgánico o promoted)

```
┌───────────────────────────────────────────┐
│  [Cover photo / carousel]                  │
│                               [🔖 Save]    │
│  [Promoted] (only if paid)                 │
├───────────────────────────────────────────┤
│  Service Name (bold)                       │
│  [Avatar] Provider Name  ★ 4.8 (120)      │
│  📍 2.3 km  │  ⏱ 60 min                   │
│  [Tag: Wellness]                           │
│                                            │
│  Next: Tue 15:00    [Desde $8000 · Book]   │
└───────────────────────────────────────────┘
```

Elementos clave del Listing Card:
- **"Next: Tue 15:00"** — próximo slot disponible (patrón OpenTable time chips). Si no hay slot en 7 días: "Ver disponibilidad".
- **"Book" CTA** — pill button, primary color, siempre visible (patrón ClassPass). Dos tap targets: card body (navega a detalle) + button (abre booking bottom sheet).
- **"Promoted"** label — small gray text, solo si es contenido pagado (transparency per TikTok policy).
- **Rating** — estrellas + count. Si provider nuevo: "Nuevo" badge en vez de rating.
- **Distance** — solo si ubicación disponible.

**Suggest Card** (recomendación contextual)

```
┌───────────────────────────────────────────┐
│  ✦ Suggested for you                      │
│  "Tenés 45 min libres mañana a las 14h.  │
│   Hay disponibilidad cerca:"              │
│  ┌──────────────────────────────────────┐ │
│  │ [Mini listing card with Book CTA]    │ │
│  └──────────────────────────────────────┘ │
│  [Dismiss] [Not interested in this type]  │
└───────────────────────────────────────────┘
```

- Generada por el módulo Suggest basada en agenda + ubicación + preferencias.
- Dismissible — "Not interested" entrena el algoritmo.
- Max 1 suggest card cada 5 cards de feed (evitar spam).

#### Interacciones del feed

| Acción | Gesto | Resultado |
|---|---|---|
| Scroll | Swipe vertical | Infinite scroll, carga más contenido |
| Pull to refresh | Pull down | Refresh del feed con nuevo contenido |
| Like | Tap ♡ | Optimistic UI, emite `ReactionAdded` |
| Save/Bookmark | Tap 🔖 | Guarda en colección privada |
| Tap card body | Tap | Navega a detalle (post detail / listing detail) |
| Tap "Book" button | Tap | Abre Booking Bottom Sheet (Capability C) |
| Tap "Follow" | Tap | Follow request / instant follow |
| Long press card | Long press | Share sheet (copy link, share to...) |
| Dismiss suggest | Tap dismiss | Remueve suggest card, feedback al algoritmo |

---

### Capability B — Search Progresivo

La search bar está en la navbar. Al tocar 🔍, se expande a una pantalla de búsqueda con evolución progresiva: texto → sugerencias → tags → resultados filtrados.

#### Estado 1 — Search idle (en navbar)

```
┌─────────────────────────────────────────┐
│  [Logo]         [🔍]    [🔔]  [Avatar]  │
└─────────────────────────────────────────┘
```

#### Estado 2 — Search active (tap en 🔍)

```
┌─────────────────────────────────────────┐
│  [←] [🔍 Buscar servicios, personas...] │
├─────────────────────────────────────────┤
│                                         │
│  Recientes                              │
│  [×] masaje relajante                   │
│  [×] dra. garcía                        │
│                                         │
│  Trending                               │
│  🔥 yoga  🔥 peluquería  🔥 nutrición  │
│                                         │
│  Categorías                             │
│  [🧘 Wellness] [💇 Belleza] [🏥 Salud]  │
│  [🎵 Música]   [🏋 Fitness] [🎨 Arte]   │
│                                         │
└─────────────────────────────────────────┘
```

#### Estado 3 — Typing (autocomplete)

```
┌─────────────────────────────────────────┐
│  [←] [🔍 masa|                        ] │
├─────────────────────────────────────────┤
│                                         │
│  Sugerencias                            │
│  📋 masaje relajante     (servicio)     │
│  📋 masaje deportivo     (servicio)     │
│  👤 Masajes Zen          (provider)     │
│  📍 Masajes Club Norte   (lugar)        │
│                                         │
│  Tags relacionados                      │
│  [Wellness] [Relax] [Spa] [Deportivo]   │
│                                         │
└─────────────────────────────────────────┘
```

Autocomplete muestra resultados tipados: servicios, providers, lugares. Tap en sugerencia ejecuta la búsqueda. Tags se acumulan como filtros.

#### Estado 4 — Results con tag chips activos

```
┌─────────────────────────────────────────┐
│  [←] [🔍 masaje                       ] │
│  [Wellness ×] [< 5km ×] [< $10000 ×]  │
├─────────────────────────────────────────┤
│  [Listings (12)]  [Providers (5)]       │
├─────────────────────────────────────────┤
│                                         │
│  ┌─ Listing Card ────────────────────┐  │
│  │ Masaje Relajante · Studio Zen     │  │
│  │ ★ 4.8 · $8000 · 60 min · 1.2 km │  │
│  │         [Reservar]                │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌─ Listing Card ────────────────────┐  │
│  │ Masaje Deportivo · Relax Pro      │  │
│  │ ★ 4.5 · $6000 · 45 min · 2.8 km │  │
│  │         [Reservar]                │  │
│  └───────────────────────────────────┘  │
│                                         │
│  [Providers tab: show profile cards]    │
│                                         │
└─────────────────────────────────────────┘
```

**Tag chips de filtro:**
- Se agregan por tap en tags sugeridos, categorías, o desde el panel de filtros.
- Removibles con ×.
- Filtros disponibles: categoría, distancia, rango de precio, disponibilidad (hoy/esta semana), rating mínimo.
- Los filtros se aplican en tiempo real (no requieren submit).

**Tabs de resultados:**
- **Listings** (default): cards de servicios reservables.
- **Providers**: cards de perfiles (avatar, nombre, rating, servicios principales, distancia).
- Futuro: Places, Posts.

---

### Capability C — Booking desde el Feed (Bottom Sheet)

Cuando el usuario toca "Reservar" en un Listing Card, se abre un **bottom sheet** que permite booking en 3 taps sin salir del feed.

#### Flujo de 3 taps

```
Feed Card [Tap 1: "Reservar"]
    │
    ▼
Bottom Sheet — Half Screen (Slot Quick Pick)
    ┌──────────────────────────────────────┐
    │  ─── (swipe handle)                  │
    │                                      │
    │  Masaje Relajante · 60 min           │
    │  [Ava] Studio Zen  ★ 4.8            │
    │  $8,000                              │
    │                                      │
    │  Próximos horarios disponibles:      │
    │  [Mar 15:00] [Mar 17:00] [Mié 10:00]│
    │  [Mié 14:00] [Jue 9:00]             │
    │                                      │
    │        [Ver todos los horarios →]    │
    │                                      │
    │  [Tap 2: select time chip]           │
    └──────────────────────────────────────┘
    │
    ▼
Bottom Sheet — Expanded (Confirm)
    ┌──────────────────────────────────────┐
    │  ─── (swipe handle)          [✕]     │
    │                                      │
    │  ✓ Martes 24 Mar · 15:00 - 16:00    │
    │  Masaje Relajante · 60 min           │
    │  Studio Zen · Av. Corrientes 1234    │
    │  $8,000                              │
    │                                      │
    │  ┌──────────────────────────────────┐│
    │  │ Juan Pérez (pre-filled)         ││
    │  │ juan@email.com (pre-filled)     ││
    │  └──────────────────────────────────┘│
    │                                      │
    │  [Tap 3: ✦ Confirmar Reserva]        │
    │                                      │
    │  Confirmación automática             │
    │  Cancellation: Flexible              │
    └──────────────────────────────────────┘
    │
    ▼
Toast on Feed (Confirmation — feed position preserved)
    ┌──────────────────────────────────────┐
    │  ✓ ¡Reservado! Masaje · Mar 15:00   │
    │  [Ver detalle]            [Deshacer] │
    └──────────────────────────────────────┘
```

#### Reglas del bottom sheet

| Regla | Detalle |
|---|---|
| **Pre-selected slot** | El primer chip es el "Next available" que ya se mostraba en el feed card |
| **Pre-filled data** | Nombre y email del perfil del usuario. Read-only display, no editable fields |
| **Time chips** | Próximos 5 slots disponibles como chips horizontales. Tap selecciona. |
| **Custom form** | Si `Listing.formRequired = true`, el sheet se expande con campos adicionales (adds 1-2 taps) |
| **"Ver todos"** | Navega a full-page calendar/slot picker (fuera del bottom sheet) |
| **Close button** | Visible ✕ en esquina + swipe-down para cerrar (NN/g guideline) |
| **No nesting** | Never nest bottom sheets. Si se necesita más info, navegar a full page |
| **Hold + TTL** | Al seleccionar slot, se hace `POST /bookings/hold` con countdown (5 min). Timer visible |
| **Error recovery** | Si slot taken (409): toast "Este horario se acaba de ocupar" + refresh chips inline |

#### Confirmation pattern

- **Toast/snackbar** en el feed: "¡Reservado! [Servicio] · [Fecha Hora]" con "Ver detalle" y "Deshacer".
- **Feed position preserved** — el usuario sigue scrolleando. El feed card se actualiza con badge "Reservado".
- **"Deshacer"** (10 segundos) — cancela el booking inmediatamente (release hold).
- **Push notification + email** enviados asincrónicamente.

---

### Capability D — Discovery Carousels

En la parte superior del feed (debajo de navbar) y cada ~15-20 cards, aparecen **carruseles horizontales** con contenido agrupado temáticamente.

#### Tipos de carousels

| Carousel | Contenido | Trigger |
|---|---|---|
| **Near Me** | Listings con disponibilidad cercana (geo) | Siempre (si ubicación disponible) |
| **This Week** | Listings con slots esta semana | Siempre |
| **Because You Follow @provider** | Offerings de providers que sigue | Si sigue providers |
| **Popular in [Category]** | Top listings por engagement/bookings | Basado en historial |
| **New Providers** | Providers recién registrados con offerings | Cold start boost |
| **Your Recent** | Listings visitados recientemente / rebooking | Si tiene historial |

#### Layout del carousel

```
┌─────────────────────────────────────────────────────┐
│  Near Me                              [See all →]    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────     │
│  │ [foto]   │ │ [foto]   │ │ [foto]   │ │          │
│  │ Masaje   │ │ Yoga     │ │ Corte    │ │  ...     │
│  │ ★4.8 1km │ │ ★4.6 2km │ │ ★4.9 0.5 │ │          │
│  │ $8000    │ │ $3000    │ │ $2500    │ │          │
│  └──────────┘ └──────────┘ └──────────┘ └─────     │
└─────────────────────────────────────────────────────┘
```

- Cards compactas con: thumbnail, nombre, rating, distancia, precio.
- Horizontal scroll con peek (siguiente card parcialmente visible).
- "See all →" navega a grid completa filtrada.
- Tap en card: navega a listing detail o abre booking bottom sheet.

---

### Capability E — Cold Start (Usuario Nuevo)

Para usuarios sin follows ni historial, el feed usa una estrategia de bootstrapping.

#### Onboarding interests (primer uso)

```
┌─────────────────────────────────────────┐
│  ¿Qué te interesa?                      │
│                                         │
│  [🧘 Wellness]  [💇 Belleza]  [🏥 Salud]│
│  [🎵 Música]    [🏋 Fitness]  [🎨 Arte] │
│  [🐕 Mascotas]  [📚 Educación] [🍽 Food]│
│                                         │
│  Elegí al menos 3              [Listo]  │
│                                         │
└─────────────────────────────────────────┘
```

Después de seleccionar intereses, el cold start feed muestra:
1. **Carousels por categoría elegida** (top of feed).
2. **Listings populares near me** por cada categoría.
3. **Trending providers** en la zona.
4. **Suggest cards** tipo "Descubrí [categoría] cerca tuyo".

#### Cold start fallback (sin ubicación)

Si el usuario no comparte ubicación:
- Feed basado solo en categorías seleccionadas + trending global.
- Banner suave: "Compartí tu ubicación para ver servicios cerca tuyo" (no bloqueante).

#### Graduation (salida del cold start)

El usuario "gradúa" del cold start cuando:
- Sigue al menos 5 providers, **o**
- Tiene al menos 3 interacciones (likes, saves, bookings).

Después, el feed transiciona al ratio normal (60/25/15).

---

## 6. Alternate & Edge Cases

| # | Caso | Comportamiento |
|---|---|---|
| ALT-01 | Feed vacío (error de carga) | Skeleton loading → si falla: "No pudimos cargar el feed. [Reintentar]" |
| ALT-02 | Search sin resultados | "No encontramos resultados para '[query]'. Probá con otros filtros o explorá categorías." + categorías sugeridas |
| ALT-03 | Slot tomado durante booking (409) | Bottom sheet muestra toast "Este horario se acaba de ocupar" + refresh automático de chips |
| ALT-04 | Hold expirado (TTL) | Bottom sheet muestra countdown → al expirar: "Tu reserva expiró. Seleccioná otro horario." Sheet vuelve a slot selection |
| ALT-05 | Listing unpublished mientras se ve en feed | Si el usuario toca "Reservar": toast "Este servicio ya no está disponible". Card se actualiza/oculta |
| ALT-06 | Provider bloqueado por usuario | Posts y listings del provider no aparecen en feed (Access enforcement) |
| ALT-07 | Feed con solo contenido promoted | Si el algoritmo no tiene contenido orgánico suficiente, no llenar con promoted. Mostrar carousels de discovery en su lugar |
| ALT-08 | Booking requiere custom form | Bottom sheet se expande con campos del form. Si el form es complejo (>3 campos), navegar a full page en vez de bottom sheet |
| ALT-09 | Usuario toca "Reservar" sin autenticarse | Redirect a login con `returnUrl` que restaura contexto (listing + slot pre-seleccionado) |
| ALT-10 | Pull to refresh durante booking bottom sheet | Bottom sheet permanece abierto. Feed se refresha debajo. No interrumpir el flow de booking |

---

## 7. Data Model

### FeedItem (Read Model — projection)

```
FeedItem {
  id: UUID
  type: POST | LISTING | SUGGEST | PROMOTED
  sourceProfileId: UUID
  sourceProfileName: string
  sourceProfileAvatar: URL
  createdAt: DateTime

  // For POST type
  postType: TEXT | MEDIA | LINK | LISTING_SHARE
  postContent: string?
  postMedia: MediaItem[]?
  referencedListingId: UUID?  // for LISTING_SHARE

  // For LISTING / PROMOTED type
  listingId: UUID
  listingTitle: string
  listingPrice: decimal
  listingDurationMin: int
  listingRating: decimal?
  listingReviewCount: int
  listingPlaceName: string?
  listingDistanceKm: decimal?  // calculated per-user
  listingCoverImage: URL?
  nextAvailableSlot: { start: DateTime, slotId: UUID }?

  // For SUGGEST type
  suggestType: TIME_GAP | NEARBY | TRENDING | REBOOKING
  suggestMessage: string
  suggestListingId: UUID?

  // Engagement
  likeCount: int
  commentCount: int
  isLikedByViewer: bool
  isSavedByViewer: bool
}
```

### SearchResult (Read Model)

```
SearchResult {
  type: LISTING | PROFILE | PLACE
  id: UUID
  title: string
  subtitle: string?
  imageUrl: URL?
  rating: decimal?
  reviewCount: int?
  distanceKm: decimal?
  price: decimal?           // for LISTING
  durationMin: int?         // for LISTING
  followerCount: int?       // for PROFILE
  matchedTags: Tag[]
}
```

---

## 8. Queries

| Query | Params | Returns | Used in |
|---|---|---|---|
| `GetFeed` | `profileId`, `cursor`, `limit` | Paginated FeedItems (mixed types) | Feed principal |
| `GetFeedColdStart` | `profileId`, `interests[]`, `location?` | Bootstrapped feed for new users | Cold start |
| `SearchGlobal` | `query`, `filters`, `cursor`, `limit` | Typed SearchResults (listings, profiles, places) | Search |
| `SearchAutocomplete` | `partial`, `limit` | Suggested completions with type | Search typing |
| `GetDiscoveryCarousel` | `type`, `location?`, `category?`, `limit` | Compact listing cards for carousel | Carousels |
| `GetListingSlots` | `listingId`, `from`, `to` | Available slots for bottom sheet | Booking from feed |
| `GetSuggestions` | `profileId`, `location?`, `timeWindow` | Contextual suggest cards | Suggest cards |

---

## 9. Commands

| Command | Params | Effect | Events emitted |
|---|---|---|---|
| `AddReaction` | feedItemId, reactionType | Adds like/reaction | `ReactionAdded` |
| `RemoveReaction` | feedItemId, reactionType | Removes like/reaction | `ReactionRemoved` |
| `SaveToCollection` | listingId / postId | Saves to user's private collection | `ItemSaved` |
| `DismissSuggestion` | suggestId, reason? | Removes suggest card, trains algorithm | `SuggestionDismissed` |
| `HoldSlot` | listingId, slotId | Reserves slot with TTL (5 min) | `SlotHeld` |
| `CreateBooking` | listingId, slotId, holdId, intake? | Confirms the booking | `BookingCreated` |
| `RecordSearch` | query, filters, resultCount | Analytics tracking | `SearchPerformed` |
| `SelectInterests` | profileId, categoryIds[] | Sets cold start preferences | `InterestsUpdated` |

---

## 10. Events (Domain Events)

| Event | Raised by | Consumed by |
|---|---|---|
| `ReactionAdded` / `ReactionRemoved` | Social | Feed (update count), Notifications (if post owner) |
| `ItemSaved` | Social | User collections, Feed (update saved state) |
| `SuggestionDismissed` | Suggest | Suggest algorithm (negative signal) |
| `SlotHeld` | Booking | Timeline (temporary block), Listing (decrement available) |
| `BookingCreated` | Booking | Timeline (permanent block), Notifications (both parties), Feed (badge update) |
| `SearchPerformed` | Search | Analytics, Suggest (search as signal) |
| `InterestsUpdated` | Social | Feed algorithm (cold start preferences) |
| `ListingPublished` | Listing | Feed (add LISTING_SHARE if opted), Search (index), Discovery (candidate) |
| `PostPublished` | Social | Feed (add to followers' feeds) |

---

## 11. Invariants

1. Feed nunca muestra contenido bloqueado por Access (blocked users, private listings sin permiso).
2. Promoted content siempre lleva label "Promoted" visible — nunca se disfraza de orgánico.
3. Suggest cards máximo 1 cada 5 feed items — evitar spam de sugerencias.
4. Booking bottom sheet no nesta otros sheets — si se necesita más, navegar a full page.
5. Slot hold tiene TTL estricto — no se puede extender. Al expirar, slot se libera.
6. Search results respetan Access — listings privados no aparecen a usuarios sin permiso.
7. Cold start feed no depende de follows — funciona con solo categorías + ubicación.
8. Pull to refresh no interrumpe bottom sheet abierto.
9. Feed position se preserva después de booking confirmation — el usuario no pierde su lugar.
10. "Deshacer" booking solo disponible dentro de 10 segundos post-confirmación.

---

## 12. Technical Mapping

### MFE

| MFE | Routes | Purpose |
|---|---|---|
| `vt-discover-mfe` | `/home` | Feed principal (default route, tab 1) |
| `vt-discover-mfe` | `/home/search` | Search active state |
| `vt-discover-mfe` | `/home/search/results` | Search results with filters |

### Toolkit Components Needed

| Component | Description | Priority |
|---|---|---|
| `vt-post-card` | Social post card (TEXT, MEDIA, LINK, LISTING_SHARE types) | High |
| `vt-listing-card` | Bookable listing card with "Book" CTA + next available | High |
| `vt-suggest-card` | Contextual suggestion card with dismiss action | High |
| `vt-promoted-card` | Listing card variant with "Promoted" label | High |
| `vt-booking-bottom-sheet` | Multi-state bottom sheet for inline booking (slot chips + confirm) | High |
| `vt-feed-search-bar` | Search input with autocomplete + tag chips | High |
| `vt-discovery-carousel` | Horizontal scrollable carousel of compact listing cards | Medium |
| `vt-reaction-bar` | Like, comment, share, save action bar (exists — verify/extend) | Medium |
| `vt-tag-chip` | Removable filter tag chip | Low |
| `vt-empty-search` | No results state with category suggestions | Low |

### Backend Dependencies

| Endpoint / Module | Status | Notes |
|---|---|---|
| Feed algorithm (GetFeed) | NEEDS-CLARIFICATION | Read model with content mixing logic |
| Search (SearchGlobal) | Domain docs exist | Elasticsearch / Meilisearch implementation TBD |
| Suggest (GetSuggestions) | Domain docs exist | Requires Timeline + Location integration |
| Booking hold (HoldSlot) | Spec pack exists (DRAFT-open-and-book) | 5-min TTL with slot locking |
| Available slots (GetListingSlots) | Spec pack exists | Already defined in DRAFT-open-and-book |
| Discovery carousels | NEEDS-CLARIFICATION | Caching strategy for popular/nearby content |

---

## 13. Acceptance Criteria

| # | Criterio | Verifiable by |
|---|---|---|
| AC-01 | Feed muestra mezcla de posts, listings, suggests en un solo scroll | Visual inspection |
| AC-02 | Pull to refresh carga nuevo contenido | E2E test |
| AC-03 | Listing card muestra "Next available" slot + Book CTA siempre visible | Visual inspection |
| AC-04 | Tap en "Book" abre bottom sheet con 5 slots como chips | E2E test |
| AC-05 | Booking desde feed completa en ≤ 3 taps (happy path sin custom form) | E2E test |
| AC-06 | Feed position preserved after booking confirmation (toast, no redirect) | E2E test |
| AC-07 | Search autocomplete muestra resultados tipados (listing, provider, place) | E2E test |
| AC-08 | Tag chips filtran resultados en tiempo real | E2E test |
| AC-09 | Cold start feed muestra contenido sin follows (categorías + ubicación) | E2E test |
| AC-10 | Onboarding interests permite seleccionar ≥ 3 categorías | E2E test |
| AC-11 | Promoted content lleva label "Promoted" siempre visible | Visual inspection |
| AC-12 | Suggest cards aparecen max 1 cada 5 feed items | Algorithm test |
| AC-13 | Slot hold expira después de TTL con feedback al usuario | Integration test |
| AC-14 | "Deshacer" booking funciona dentro de 10 segundos | E2E test |
| AC-15 | Blocked users' content never appears in feed | Security test |
| AC-16 | Discovery carousels scroll horizontally with peek | Visual inspection |

---

## 14. NEEDS-CLARIFICATION

| # | Pregunta | Impacto |
|---|---|---|
| NC-01 | ¿Feed algorithm es server-side (pre-mixed) o client-side (merge de streams)? | Backend architecture, performance |
| NC-02 | ¿Read model de FeedItem es event-sourced projection o view materializada? | Data pipeline, latency |
| NC-03 | ¿Usuarios no autenticados pueden ver el feed? ¿Con qué limitaciones? | Auth flow, cold start |
| NC-04 | ¿El "Deshacer" booking (10s) es un soft-delete del booking o un cancel del hold? | Booking module logic |
| NC-05 | ¿Search engine: Elasticsearch, Meilisearch, u otro? | Backend implementation |
| NC-06 | ¿Discovery carousels se cachean? ¿TTL de cache? | Performance, freshness |
| NC-07 | ¿El ratio 60/25/15 es configurable por Feature Flag? | Feed tuning |
| NC-08 | ¿Bottom sheet booking: si hay custom form > 3 campos, cuál es el threshold para navegar a full page? | UX decision |
| NC-09 | ¿`nextAvailableSlot` en FeedItem es denormalized (projection) o batch-queried client-side? | Performance tradeoff |
