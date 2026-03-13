---
title: Flow 03 — Social: Follow, Feed, Posts y Reactions
description: >
  Cualquier usuario puede seguir perfiles (con o sin aprobación), publicar Posts
  (con cross-posting opcional a redes externas), reaccionar a contenido y consumir
  un feed algorítmico personalizado. El feed combina Posts y Listings de perfiles
  seguidos con sugerencias contextuales (Suggest). Explore muestra contenido público
  sin necesidad de seguir a nadie.
status: draft
version: 2
---

# Flow 03 — Social: Follow, Feed, Posts y Reactions

## 1. Resumen
- **Goal:** que cualquier usuario pueda construir su grafo social (follow/unfollow), publicar
  contenido (Posts de distintos tipos con cross-posting a redes externas), reaccionar a Posts
  y Listings, y consumir un feed personalizado y una pantalla de Explore.
- **Actores:**
  - **Primary:** cualquier usuario autenticado (follower y/o autor de contenido).
  - **Secondary:** Sistema VyteMerge — proyecta el feed, evalúa elegibilidad, encola cross-posts.
- **Surfaces:** `vt-social-mfe` (nuevo MFE).
  - `/feed` — home feed algorítmico.
  - `/explore` — contenido público (requiere sesión).
  - `/profile/:id` — perfil + posts + listings.
  - `/profile/:id/followers` y `/profile/:id/following`.

---

## 2. Domain Context

### Social loop
```
Profile publica Post (y/o Listing)
  → followers lo ven en /feed
  → cualquiera lo ve en /explore (si PUBLIC)
  → reaccionan con Reactions
  → el engagement alimenta el algoritmo de feed
```

### Post vs Listing
Un Post es social (contenido editorial). Un Listing es comercial (reservable).
Pueden vincularse: un `LISTING_SHARE` post referencia un Listing con CTA "Reservar".
Un Listing puede tener su propia pantalla de reacciones; los Post siempre las tienen.

### Feed vs Explore vs Suggest
| Superficie | Contenido | Algoritmo | Sesión |
|-----------|-----------|-----------|--------|
| `/feed` | Posts + Listings de seguidos + Suggest | Algorítmico personalizado | Requerida |
| `/explore` | Posts PUBLIC + Listings Published de toda la plataforma | Semi-algorítmico (recencia + engagement) | Requerida |
| Suggest inline | Cards contextuales basadas en agenda + lugar | Contextual (Timeline + Place) | Requerida |

### Grafo social (estados)
```
NONE → FOLLOWING          (follow en perfil público)
NONE → REQUESTED          (follow en perfil privado)
REQUESTED → FOLLOWING     (owner aprueba)
REQUESTED → NONE          (owner rechaza o solicitante cancela)
FOLLOWING → NONE          (unfollow)
FOLLOWING → MUTED         (mute: permanece FOLLOWING pero no aparece en feed)
MUTED → FOLLOWING         (unmute)
```
> Block y Mute avanzado → Flow 14 (Moderación).

### Cross-posting externo
Al publicar un Post, el usuario puede seleccionar redes externas conectadas (Instagram, Facebook).
El sistema encola un job que publica de forma asíncrona tras guardar el Post en VyteMerge.
El resultado (éxito/fallo) se registra en `ExternalPostRecord`.

---

## 3. Preconditions
- Usuario autenticado con Profile activo.
- Para follow de perfil privado: el perfil objetivo debe existir y ser accesible (no bloqueado).
- Para crear Post: Profile activo (cualquier tipo: Personal o Business).
- Para cross-posting: cuenta externa conectada mediante OAuth en configuración de perfil.

---

## 4. Trigger

**Follow:** usuario visita `/profile/:id` y pulsa "Seguir".
**Post:** usuario pulsa el botón flotante en `/feed`, desde su Profile, o pulsa "Compartir" en un Listing.
**Feed:** usuario navega a `/feed` (entrada principal de la app).
**Explore:** usuario navega a `/explore`.
**Reaction:** usuario pulsa el ícono de reacción en un Post o Listing.

---

## 5. Main Flow

### Capacidad A — Follow / Unfollow

#### A.1 — Follow (perfil público)
1. Usuario A visita `/profile/:id` (perfil público de B).
2. Usuario A pulsa **"Seguir"**.
3. Sistema crea `FollowRelation(follower=A, followee=B, status=FOLLOWING)`.
4. Sistema emite `FollowCreated` → Feed proyección: Posts y Listings elegibles de B aparecen en el feed de A.
5. B recibe notificación in-app: "[A] empezó a seguirte".

#### A.2 — Follow (perfil privado)
1. Usuario A pulsa "Seguir" en el perfil privado de B.
2. Sistema crea `FollowRelation(status=REQUESTED)` y emite `FollowRequested`.
3. B recibe notificación in-app: "[A] quiere seguirte" con CTAs "Aceptar / Rechazar".
4. B abre `/profile/:id/followers` y ve la solicitud pendiente.
5. **Si acepta:** `status → FOLLOWING`; emite `FollowApproved`; A recibe notificación; contenido de B entra al feed de A.
6. **Si rechaza:** `status → NONE`; emite `FollowRejected`; A no recibe notificación (silencioso).
7. **Si A cancela antes de que B decida:** `status → NONE`; emite `FollowCancelled`.

#### A.3 — Unfollow
1. A pulsa "Dejar de seguir" en el perfil de B o desde su lista de Following.
2. Sistema elimina/inactiva `FollowRelation`; emite `FollowRemoved`.
3. Feed de A deja de incluir contenido de B (eventual: en el próximo refresh del feed).

#### A.4 — Mute
1. A pulsa "Silenciar" en el perfil de B o desde el feed.
2. `FollowRelation.status → MUTED`; emite `ProfileMuted`.
3. B no aparece en el feed de A, pero la relación de follow se mantiene activa.
4. B no recibe notificación.

---

### Capacidad B — Crear Post

5. Usuario pulsa el FAB (+) en `/feed`, accede a su Profile y pulsa "Nuevo post", o pulsa "Compartir" en un Listing.
6. Sistema muestra el composer según el tipo de Post:
   - **TEXT:** campo de texto (hasta N caracteres; NEEDS-CLARIFICATION: límite).
   - **MEDIA:** upload de imagen/video + caption.
   - **LINK:** campo de URL → sistema genera link preview automáticamente.
   - **LISTING_SHARE:** el Listing ya está pre-cargado; usuario puede agregar texto + caption.
7. Usuario selecciona **visibilidad**: PUBLIC | FOLLOWERS | UNLISTED | PRIVATE.
8. (Opcional) Usuario selecciona **redes externas** para cross-posting (Instagram, Facebook, etc.) si tiene cuentas conectadas.
9. Usuario pulsa **"Publicar"**.
10. Sistema:
    - Guarda el Post en estado `PUBLISHED`.
    - Emite `PostPublished` → Feed projection actualiza elegibilidad.
    - Si hay cross-posting: encola `ExternalPostJob` por cada red seleccionada (asíncrono).
11. Post aparece en el Profile del autor y en el feed de sus followers según visibilidad.
12. Para cada cross-posting encolado: sistema intenta publicar vía API externa; registra `ExternalPostRecord` con resultado (posted/failed).

#### B.1 — Editar Post
- Usuario puede editar texto/visibilidad de un Post `PUBLISHED`.
- No se puede editar el tipo de post ni el `listingId`.
- Si se cambia la visibilidad a `PRIVATE`: el Post desaparece del feed de followers (eventual).

#### B.2 — Archivar Post
- Usuario pulsa "Archivar" → `status → ARCHIVED`.
- Post desaparece del Profile y del Feed; se conserva en historial privado del autor.

---

### Capacidad C — Feed algorítmico (`/feed`)

13. Usuario navega a `/feed`.
14. Sistema compone el feed del usuario mezclando tres fuentes:
    - **Posts y Listings de seguidos** (read model proyectado por `PostPublished` / `ListingPublished` / `FollowApproved`).
    - **Suggest contextual** (cards basadas en Timeline y Place: "Tenés disponibilidad cerca").
    - **Contenido público de alta relevancia** (fallback si el usuario sigue a pocos perfiles).
15. Sistema aplica **algoritmo de ranking v1**:
    ```
    score(item) =
      recency_score(item.publishedAt)        -- más reciente = mayor score
      × relationship_multiplier(relation)    -- mutual follow > following > sugerido
      × engagement_boost(item.reactionsCount)
      × availability_boost(item)             -- Listings con slots próximos (<48h): +boost
      × geo_boost(item, user.lastKnownPlace) -- Listings cercanos geográficamente: +boost
    ```
    Donde:
    - `recency_score = 1 / (1 + horas_desde_publicación)^1.5`
    - `relationship_multiplier`: mutual_follow=3.0, following=2.0, suggested=1.0
    - `engagement_boost = log(1 + reactionsCount) × 0.3`
16. Items marcados como `MUTED` se excluyen del feed.
17. Items del mismo autor no se agrupan más de N veces seguidas (anti-spam; NEEDS-CLARIFICATION: N).
18. Feed se entrega paginado (cursor-based). Al hacer scroll, se cargan más items.

---

### Capacidad D — Explore (`/explore`)

19. Usuario navega a `/explore`.
20. Sistema muestra contenido `PUBLIC` de toda la plataforma, no filtrado por follows:
    - Posts PUBLIC ordenados por score (recencia + engagement).
    - Listings Published con mayor actividad reciente.
    - Filtros disponibles: tipo de contenido, categoría/tag, ubicación/distancia.
21. Sin sesión activa → redirect a login (Explore requiere sesión).
22. Contenido de perfiles bloqueados por el usuario se excluye (Access enforcement).

---

### Capacidad E — Reactions

23. Usuario ve un Post o Listing y pulsa el ícono de reacción.
24. Sistema muestra selector de tipos: ❤️ Like / 🔥 Fire / 👏 Clap / 😮 Wow (NEEDS-CLARIFICATION: set final).
25. Usuario selecciona tipo.
26. Sistema registra `Reaction(actorId, targetType, targetId, type)`.
27. El contador de reacciones del item se actualiza (eventual consistency).
28. Si el usuario ya reaccionó al mismo item con el mismo tipo → se elimina la reacción (toggle).
29. Si el usuario ya reaccionó con un tipo diferente → se reemplaza.
30. El autor del Post/Listing recibe notificación in-app: "[A] reaccionó a tu publicación".

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| Perfil privado — A sigue a B y B hace su perfil público después | FollowRelation en FOLLOWING permanece; no hay cambio requerido |
| A cancela solicitud de follow antes de que B decida | `status → NONE`; la notificación de B se marca como expirada |
| Post con `LISTING_SHARE` cuyo Listing fue archivado | El Post se muestra pero la card del Listing indica "no disponible"; CTA deshabilitado |
| Cross-posting a Instagram falla | Post en VyteMerge ya está publicado; `ExternalPostRecord.status = failed`; usuario ve aviso en su Post |
| Usuario intenta seguir a alguien que lo bloqueó | Sistema devuelve 403; no se crea FollowRelation (Access enforcement) |
| Feed vacío (usuario sin follows y sin Suggest) | Sistema muestra estado vacío con CTA "Descubrir perfiles en Explore" |
| Reaction en Listing por usuario que no está autenticado | Redirect a login |
| Post UNLISTED compartido por link a usuario bloqueado | Access rechaza; 403 |
| Mutual unfollow | Cada dirección es una FollowRelation independiente; unfollow en un sentido no afecta el otro |
| Cross-posting a Facebook — post tiene MEDIA > límite de FB | Job falla; `ExternalPostRecord.status = failed`; usuario notificado |
| Feed de usuario con muchos follows (>1000) | Proyección acotada a los N más relevantes; NEEDS-CLARIFICATION: N |

---

## 7. Data Model (v1)

```
-- Módulo: Social

FollowRelation {
  id          UUID
  followerId  UUID          -- Profile que sigue
  followeeId  UUID          -- Profile seguido
  status      REQUESTED | FOLLOWING | MUTED
  createdAt   DateTime
  updatedAt   DateTime
  -- unique constraint: (followerId, followeeId)
}

Post {
  id              UUID
  authorId        UUID          -- Profile
  type            TEXT | MEDIA | LINK | LISTING_SHARE
  body            string?       -- TEXT, MEDIA caption, LISTING_SHARE caption
  mediaUrls       string[]?     -- MEDIA: URLs al storage
  linkUrl         string?       -- LINK
  linkPreview     LinkPreview?  -- { title, description, imageUrl, domain }
  listingId       UUID?         -- LISTING_SHARE: referencia (solo ID; no copy)
  visibility      PUBLIC | FOLLOWERS | UNLISTED | PRIVATE
  status          PUBLISHED | ARCHIVED
  reactionsCount  int           -- denormalizado para ranking
  crossPosts      ExternalPostRecord[]
  publishedAt     DateTime
  archivedAt      DateTime?
  createdAt       DateTime
  updatedAt       DateTime
}

ExternalPostRecord {
  id          UUID
  postId      UUID
  platform    instagram | facebook | twitter | linkedin | tiktok
  status      pending | posted | failed
  externalId  string?       -- ID del post en la red externa
  errorMsg    string?
  postedAt    DateTime?
  createdAt   DateTime
}

Reaction {
  id          UUID
  actorId     UUID          -- Profile
  targetType  post | listing
  targetId    UUID
  type        like | fire | clap | wow
  createdAt   DateTime
  -- unique constraint: (actorId, targetType, targetId)
}

-- Read model: Feed projection

FeedItem {
  id           UUID
  feedOwnerId  UUID          -- Profile cuyo feed es
  itemType     post | listing | suggest_card
  itemId       UUID
  authorId     UUID
  score        float
  seenAt       DateTime?
  insertedAt   DateTime
  -- índice por (feedOwnerId, score DESC, insertedAt DESC)
}
```

---

## 8. Commands

### Módulo Social — Follow

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `RequestFollow` | `FollowRelation` | No existe FollowRelation activa; no hay bloqueo |
| `ApproveFollow` | `FollowRelation` | Status=REQUESTED; caller=followee (Profile owner o Delegado) |
| `RejectFollow` | `FollowRelation` | Status=REQUESTED; caller=followee |
| `CancelFollowRequest` | `FollowRelation` | Status=REQUESTED; caller=follower |
| `Unfollow` | `FollowRelation` | Status=FOLLOWING; caller=follower |
| `MuteProfile` | `FollowRelation` | Status=FOLLOWING; caller=follower |
| `UnmuteProfile` | `FollowRelation` | Status=MUTED; caller=follower |

### Módulo Social — Post

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `CreatePost` | `Post` | Profile activo; tipo válido; listingId existe si LISTING_SHARE |
| `UpdatePost` | `Post` | Status=PUBLISHED; caller=autor |
| `ArchivePost` | `Post` | Status=PUBLISHED; caller=autor |
| `RestorePost` | `Post` | Status=ARCHIVED; caller=autor |

### Módulo Social — Reactions

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `AddReaction` | `Reaction` | Target existe y es accesible; actor autenticado |
| `RemoveReaction` | `Reaction` | Reaction existe; caller=actor |
| `ChangeReaction` | `Reaction` | Reaction existe con tipo diferente; caller=actor |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `FollowCreated` | `RequestFollow` (público) | Feed: inserta FeedItems de followee; Communication: notifica followee |
| `FollowRequested` | `RequestFollow` (privado) | Communication: notifica followee con solicitud |
| `FollowApproved` | `ApproveFollow` | Feed: inserta FeedItems de followee; Communication: notifica follower |
| `FollowRejected` | `RejectFollow` | Communication: silencioso (no notifica follower) |
| `FollowRemoved` | `Unfollow` | Feed: elimina FeedItems del ex-followee (eventual) |
| `ProfileMuted` | `MuteProfile` | Feed: filtra FeedItems del perfil muteado |
| `PostPublished` | `CreatePost` | Feed: inserta FeedItem en feeds de followers; Suggest: actualiza elegibilidad |
| `PostUpdated` | `UpdatePost` | Feed: actualiza FeedItem (visibility change) |
| `PostArchived` | `ArchivePost` | Feed: elimina FeedItem de todos los feeds |
| `ReactionAdded` | `AddReaction` | Social: incrementa reactionsCount en Post/Listing; Feed: recalcula score; Communication: notifica autor |
| `ReactionRemoved` | `RemoveReaction` | Social: decrementa reactionsCount |
| `ExternalPostEnqueued` | `CreatePost` (con cross-posting) | CrossPost worker: publica en plataforma externa asíncrona |

---

## 10. Invariants

1. Un usuario no puede seguirse a sí mismo.
2. Solo puede existir una `FollowRelation` activa por par `(followerId, followeeId)`.
3. `ApproveFollow` y `RejectFollow` solo los puede ejecutar el followee (o su Delegado).
4. Un Post `ARCHIVED` no aparece en Feed, Profile ni Explore.
5. Un Post `LISTING_SHARE` debe referenciar exactamente un Listing existente.
6. Un actor puede tener como máximo una Reaction por `(targetType, targetId)` a la vez.
7. `ExternalPostRecord` se crea solo si el usuario tiene la cuenta externa conectada y activa.
8. Un Post en VyteMerge se publica independientemente del resultado del cross-posting externo.
9. El feed de un usuario no incluye contenido de perfiles que el usuario tiene en MUTED.
10. `FeedItem.score` se recalcula al cambiar `reactionsCount` o al pasar tiempo (recency decay).

---

## 11. Outputs

- `FollowRelation` activa (FOLLOWING) entre dos Profiles.
- `Post` publicado en VyteMerge + (opcionalmente) en redes externas.
- `FeedItem` proyectados en el feed del usuario y sus followers.
- `Reaction` registrada con contador actualizado en el target.
- Notificaciones in-app enviadas a los actores relevantes.

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo existente:** `Social` (extender/refactorizar según patrones del monolito)

```
src/Modules/Social/
├── Social.Application/
│   ├── Commands/
│   │   ├── RequestFollow/
│   │   ├── ApproveFollow/
│   │   ├── RejectFollow/
│   │   ├── CancelFollowRequest/
│   │   ├── Unfollow/
│   │   ├── MuteProfile/
│   │   ├── UnmuteProfile/
│   │   ├── CreatePost/
│   │   ├── UpdatePost/
│   │   ├── ArchivePost/
│   │   ├── AddReaction/
│   │   ├── RemoveReaction/
│   │   └── ChangeReaction/
│   └── Queries/
│       ├── GetFeed/           -- cursor-based; aplica algoritmo de score
│       ├── GetExplore/        -- cursor-based; content PUBLIC
│       ├── GetProfile/        -- Profile + stats (followers, following, posts)
│       ├── GetFollowers/
│       ├── GetFollowing/
│       ├── GetFollowRequests/ -- owner: solicitudes pendientes
│       └── GetPostReactions/
├── Social.Domain/
├── Social.Infrastructure/     -- SocialDbContext (schema: social)
│   └── Projections/
│       └── FeedProjection/    -- handler de PostPublished, FollowApproved, etc.
├── Social.IntegrationEvents/
└── Social.Presentation/
```

**Worker externo (cross-posting):**
```
src/Workers/CrossPostWorker/
  -- Consume ExternalPostEnqueued (outbox)
  -- Publica en Instagram Graph API / Facebook Graph API / etc.
  -- Actualiza ExternalPostRecord
```

**Endpoints:**
```
-- Follow
POST   /profiles/:id/follow               → RequestFollow
POST   /profiles/:id/unfollow             → Unfollow
POST   /profiles/:id/mute                 → MuteProfile
POST   /profiles/:id/unmute               → UnmuteProfile
GET    /profiles/:id/followers            → GetFollowers
GET    /profiles/:id/following            → GetFollowing
GET    /profiles/me/follow-requests       → GetFollowRequests
POST   /profiles/me/follow-requests/:id/approve  → ApproveFollow
POST   /profiles/me/follow-requests/:id/reject   → RejectFollow

-- Posts
POST   /posts                             → CreatePost
PATCH  /posts/:id                         → UpdatePost
POST   /posts/:id/archive                 → ArchivePost
POST   /posts/:id/restore                 → RestorePost
GET    /posts/:id                         → GetPost

-- Reactions
POST   /reactions                         → AddReaction  { targetType, targetId, type }
DELETE /reactions/:id                     → RemoveReaction
PATCH  /reactions/:id                     → ChangeReaction

-- Feed
GET    /feed                              → GetFeed (cursor, limit)
GET    /explore                           → GetExplore (cursor, filters)
```

**Cross-posting — integración externa:**
```
-- Configuración de cuentas externas (OAuth)
GET    /social-accounts                   → lista de cuentas externas conectadas
POST   /social-accounts/connect           → inicia OAuth flow (platform=instagram|facebook|...)
DELETE /social-accounts/:id               → desconecta cuenta externa

-- Estado del cross-post
GET    /posts/:id/cross-posts             → lista de ExternalPostRecords del Post
```

### Frontend — `vt-social-mfe` (nuevo MFE)

```
/feed                         → home feed algorítmico
/explore                      → contenido público con filtros
/profile/:id                  → perfil público: posts + listings + stats
/profile/:id/followers        → lista de seguidores
/profile/:id/following        → lista de seguidos
/profile/me/follow-requests   → solicitudes de follow pendientes (owner)
/post/:id                     → post individual con reacciones
/settings/social-accounts     → conectar/desconectar cuentas externas (cross-posting)
```

**UI States:**

**`/feed`:**
- Estado vacío: CTA "Seguí a alguien o explorá en Explore".
- Feed con mix de Posts, Listing cards y Suggest cards.
- Botón flotante (+) para crear Post.
- Selector de tipo de post en composer.
- Panel de cross-posting al crear Post (plataformas disponibles con toggle).

**`/profile/:id`:**
- Header: avatar, nombre, bio, stats (followers / following / posts).
- CTA "Seguir" / "Siguiendo" / "Solicitud enviada".
- Tabs: Posts | Listings.
- Si perfil privado y no se sigue: solo header visible; contenido oculto.

**`/explore`:**
- Grid de contenido PUBLIC: Posts + Listing cards.
- Filtros: tipo, categoría/tag, distancia.
- Sin filtros aplicados: contenido trending (recencia + engagement).

---

## 13. Acceptance Criteria

- [ ] Usuario puede seguir un perfil público; follow se activa inmediatamente.
- [ ] Usuario puede seguir un perfil privado; queda en estado REQUESTED hasta aprobación.
- [ ] Owner de perfil privado puede aprobar o rechazar solicitudes de follow.
- [ ] Al aprobar, el follower ve el contenido del perfil en su feed.
- [ ] Al rechazar, el follower no recibe notificación.
- [ ] Usuario puede dejar de seguir; el contenido del ex-followee desaparece del feed (eventual).
- [ ] Usuario puede silenciar un perfil; el contenido muteado no aparece en feed pero la relación de follow se mantiene.
- [ ] Usuario puede crear Posts de tipo TEXT, MEDIA, LINK y LISTING_SHARE.
- [ ] Un Post LISTING_SHARE muestra la card del Listing con CTA "Reservar" si el Listing está Published.
- [ ] Si el Listing fue archivado, la card indica "no disponible" y el CTA está deshabilitado.
- [ ] Usuario puede seleccionar visibilidad del Post (PUBLIC / FOLLOWERS / UNLISTED / PRIVATE).
- [ ] Post PUBLIC aparece en `/explore` y en el feed de followers.
- [ ] Post FOLLOWERS aparece solo en el feed de followers aprobados.
- [ ] Post UNLISTED no aparece en feed ni en listas; solo accesible por link directo.
- [ ] Usuario puede archivar un Post; desaparece del feed y del Profile.
- [ ] Al crear un Post con cross-posting seleccionado, el sistema encola la publicación externa.
- [ ] El resultado del cross-posting (éxito o fallo) es visible en el Post del usuario.
- [ ] Un fallo de cross-posting no afecta la publicación en VyteMerge.
- [ ] `/feed` muestra Posts y Listings de perfiles seguidos mezclados con Suggest cards.
- [ ] El feed aplica el algoritmo de ranking (recencia × relación × engagement × proximidad).
- [ ] `/explore` muestra contenido PUBLIC ordenado por score; requiere sesión.
- [ ] Usuario puede reaccionar a un Post o Listing con múltiples tipos (like, fire, clap, wow).
- [ ] Reaccionar al mismo item con el mismo tipo elimina la reacción (toggle).
- [ ] El autor recibe notificación in-app al recibir una reacción.
- [ ] Lista de followers y following visible en `/profile/:id/followers` y `/profile/:id/following`.

---

## 14. NEEDS-CLARIFICATION

- **Límite de caracteres de Post TEXT:** ¿cuántos caracteres máximo? (ej: 500 / 1000 / sin límite).
- **Anti-spam de feed:** ¿cuántos items del mismo autor pueden aparecer seguidos (N)? ¿Y en la misma hora?
- **Feed de alta escala:** ¿cuál es el límite de follows antes de acotar la proyección? (ej: top 500 por score).
- **Set final de tipos de Reaction:** ¿like / fire / clap / wow es el set definitivo, o se ajusta con diseño?
- **Explore sin seguir:** si el usuario tiene 0 follows, ¿el `/feed` muestra contenido de Explore como fallback, o queda vacío con el CTA?
- **Cross-posting — plataformas v1:** ¿cuáles se implementan en v1? Recomiendo arrancar con Instagram + Facebook (mismo Meta API). Twitter/X, LinkedIn y TikTok tienen APIs más restrictivas.
- **Cross-posting — imagen requerida en Instagram:** Instagram Graph API requiere media para publicar. ¿Qué hacemos con Posts TEXT sin imagen al cross-postear a Instagram? ¿Bloqueamos la opción o generamos una imagen automáticamente?
- **Visibilidad del Profile privado sin follow:** ¿qué metadata mínima se muestra a quien no sigue? (nombre, avatar, bio truncada, contador de posts oculto?).
- **Reactions en Listings:** ¿las reacciones a Listings se muestran en el Listing Detail dentro de `vt-listings-mfe`, o en el Post que comparte el Listing?
- **Notificación de follow en perfiles públicos:** ¿el followee recibe notificación en tiempo real, o solo en batch (ej: "5 personas nuevas te siguen")?
