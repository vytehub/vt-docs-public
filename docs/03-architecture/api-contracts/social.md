# API Contracts — Social

Base path: `/api/v1`
Module: `Vt.Modules.Social`
Schema: `social`

> Cubre el Follow graph y Block. Profiles, Posts, Feed y Reactions son endpoints planificados (no implementados todavia).
> Ver flows `03-follow-and-feed.md` y spec pack `DRAFT-social-feed`.
> Visibilidad de Profile controlada por `ProfileVisibility`: `Public | Private` (modulo Social).

---

## Implementation status

| Area | Status |
|------|--------|
| Follow graph | Implementado |
| Block/Unblock | Implementado |
| Follow status query | Implementado |
| Profiles (GET/PATCH) | Planificado |
| Mute/Unmute | Planificado |
| Posts & Feed | Planificado |
| Reactions | Planificado |

---

## Follow Graph

### `POST /social/follows/{targetUserId}`

Envia un Follow Request al usuario target.

**Auth:** `Social.Follow`

- Si el target tiene `Visibility = Public` → acepta automaticamente → emite `FollowRequestedDomainEvent` + `FollowAcceptedDomainEvent`
- Si el target tiene `Visibility = Private` → queda en `Requested` → emite solo `FollowRequestedDomainEvent`

**Command dispatched:** `FollowUserCommand(ActorId, TargetUserId)`
**Events emitted:** `FollowRequestedDomainEvent` | `FollowRequestedDomainEvent` + `FollowAcceptedDomainEvent`

**Response:** `200 OK` (sin body)

**Errors:**
- `400` — `follow.follower_required` | `follow.followee_required` | `follow.cannot_follow_self`
- `403` — token sin permiso `Social.Follow`

---

### `DELETE /social/follows/{otherUserId}`

Deja de seguir a un usuario o cancela un Follow Request pendiente. Puede ser ejecutado por el follower o el followee (remove en ambas direcciones).

**Auth:** `Social.ManageFollowers`

**Command dispatched:** `RemoveFollowCommand(ActorId, OtherUserId)`
**Event emitted:** `FollowRemovedDomainEvent` — incluye campo `RemovedBy`

**Response:** `200 OK` (sin body)

**Errors:**
- `400` — `follow.cannot_remove_while_blocked` | `follow.only_participants_can_modify`

---

### `POST /social/follows/{followerUserId}/approve`

El followee aprueba un Follow Request pendiente del usuario `followerUserId`.

**Auth:** `Social.ManageFollowers`

**Command dispatched:** `ApproveFollowCommand(ActorId, FollowerUserId)`
**Event emitted:** `FollowAcceptedDomainEvent`

**Response:** `200 OK` (sin body)

**Errors:**
- `400` — `follow.only_followee_can_approve_or_reject` | `follow.follow_is_blocked` | `follow.not_requested`

---

### `POST /social/follows/{followerUserId}/reject`

El followee rechaza un Follow Request pendiente del usuario `followerUserId`.

**Auth:** `Social.ManageFollowers`

**Command dispatched:** `RejectFollowCommand(ActorId, FollowerUserId)`
**Event emitted:** `FollowRejectedDomainEvent`

**Response:** `200 OK` (sin body)

**Errors:**
- `400` — `follow.only_followee_can_approve_or_reject` | `follow.follow_is_blocked`

---

### `GET /social/follows/{targetUserId}/status`

Retorna el estado de la relacion de Follow entre el usuario autenticado y `targetUserId`.

**Auth:** `Social.View`

**Query dispatched:** `GetFollowStatusQuery(FollowerId, FolloweeId)`

**Response:** `200 OK`
```json
{
  "followerId": "uuid",
  "followeeId": "uuid",
  "status": 1,
  "blockedBy": "uuid | null"
}
```

> `status` es un entero: `1 = Requested`, `2 = Accepted`, `3 = Blocked`.
> `blockedBy` es el `userId` que ejecuto el bloqueo (presente solo cuando `status = 3`).

---

### `GET /social/followers/{userId}`

Lista los seguidores del usuario `userId`. Solo retorna follows en estado `Accepted`.

**Auth:** `Social.View`

**Query params:** `?limit=50&cursor=ISO8601_datetime`

> Paginacion por cursor: `cursor` es el valor `UpdatedAtUtc` del ultimo item de la pagina anterior.

**Query dispatched:** `GetFollowersQuery(UserId, Limit, CursorUpdatedAtUtc)`

**Response:** `200 OK`
```json
[
  {
    "userId": "uuid",
    "updatedAtUtc": "ISO8601"
  }
]
```

> Nota: la respuesta actual es una lista plana sin wrapper de paginacion. El nombre enriquecido del usuario (displayName, avatarUrl) debe resolverse via `GET /api/v1/social/profiles/{userId}` una vez implementado.

---

### `GET /social/following/{userId}`

Lista los usuarios que sigue `userId`. Mismo shape que `/social/followers/{userId}`.

**Auth:** `Social.View`

**Query params:** `?limit=50&cursor=ISO8601_datetime`

**Query dispatched:** `GetFollowingQuery(UserId, Limit, CursorUpdatedAtUtc)`

**Response:** `200 OK` — mismo shape que `GET /social/followers/{userId}`

---

## Block / Unblock

### `POST /social/blocks/{otherUserId}`

Bloquea al usuario `otherUserId`. Transiciona el FollowEdge existente (en cualquier estado) a `Blocked`, registrando el actor como `BlockedBy`. Si no existe FollowEdge previo, la restriccion se aplica en futuras solicitudes via logica de aplicacion.

**Auth:** `Social.Block`

**Command dispatched:** `BlockUserCommand(ActorId, OtherUserId)`
**Event emitted:** `FollowBlockedDomainEvent` — incluye `BlockedBy`

**Response:** `200 OK` (sin body)

**Errors:**
- `400` — `follow.only_participants_can_modify`

---

### `DELETE /social/blocks/{otherUserId}`

Desbloquea al usuario `otherUserId`. Solo puede ejecutarlo quien realizo el bloqueo (`BlockedBy == actorId`).

**Auth:** `Social.Block`

**Command dispatched:** `UnblockUserCommand(ActorId, OtherUserId)`
**Event emitted:** `FollowUnblockedDomainEvent` — incluye `UnblockedBy`

**Response:** `200 OK` (sin body)

**Errors:**
- `400` — `follow.only_blocker_can_unblock`

---

## Profiles (planificado)

> Los siguientes endpoints **no estan implementados todavia**. Estan documentados aqui para guiar el diseño futuro.

### `GET /social/profiles/{userId}` (planificado)

Retorna el Profile social del usuario. La respuesta varia segun la visibilidad y relacion entre actor y target.

**Auth:** `Social.View`

**Response (full — Public o Accepted follower):**
```json
{
  "id": "uuid",
  "userId": "uuid",
  "displayName": "string",
  "bio": "string | null",
  "avatarUrl": "string | null",
  "visibility": "Public | Private",
  "createdAtUtc": "ISO8601"
}
```

**Response (restringido — Private sin follow aceptado):**
```json
{
  "id": "uuid",
  "userId": "uuid",
  "displayName": "string",
  "avatarUrl": "string | null",
  "visibility": "Private"
}
```

> `Profile.DisplayName` es el campo canónico (no `name`).

---

### `PATCH /social/profiles/me` (planificado)

Actualiza el Profile del usuario autenticado.

**Request (partial):**
```json
{
  "displayName": "string",
  "bio": "string | null",
  "avatarUrl": "string | null",
  "visibility": "Public | Private"
}
```

**Command dispatched (planificado):** `UpdateProfileCommand`
**Event emitted (planificado):** `ProfileUpdatedDomainEvent`

---

## Mute (planificado)

> No implementado. El estado `Muted` suprime `PostPublished` del feed del follower sin afectar el Follow graph. El target no se entera.

### `POST /social/mutes/{otherUserId}` (planificado)

**Command dispatched (planificado):** `MuteUserCommand`

### `DELETE /social/mutes/{otherUserId}` (planificado)

**Command dispatched (planificado):** `UnmuteUserCommand`

---

## Posts & Feed (planificado)

> No implementados. Ver spec pack `DRAFT-social-feed`.

### `GET /social/feed` (planificado)

Feed del usuario autenticado (posts de usuarios seguidos con `status = Accepted` y sin Mute activo).

**Query params:** `?limit=20&cursor=string`

### `GET /social/profiles/{userId}/posts` (planificado)

Posts publicos del usuario `userId`.

### `POST /social/posts` (planificado)

**Request:**
```json
{
  "type": "text | image | video | listing_share",
  "content": "string | null",
  "media": [{ "url": "string", "type": "image | video" }],
  "listingId": "uuid | null",
  "visibility": "Public | FollowersOnly | Private"
}
```

**Command dispatched (planificado):** `CreatePostCommand`
**Event emitted (planificado):** `PostPublishedDomainEvent`

### `DELETE /social/posts/{postId}` (planificado)

**Command dispatched (planificado):** `DeletePostCommand`
**Event emitted (planificado):** `PostDeletedDomainEvent`

---

## Reactions (planificado)

> No implementados.

### `POST /social/posts/{postId}/reactions` (planificado)

**Request:** `{ "type": "like | love | wow | haha | sad | angry" }`

> Un usuario solo puede tener una Reaction activa por Post. Volver a enviar reemplaza la anterior.

**Command dispatched (planificado):** `AddReactionCommand`
**Event emitted (planificado):** `ReactionAddedDomainEvent`

### `DELETE /social/posts/{postId}/reactions` (planificado)

**Command dispatched (planificado):** `RemoveReactionCommand`
**Event emitted (planificado):** `ReactionRemovedDomainEvent`

---

## Domain Events Summary

| Event | Payload fields | Trigger |
|-------|---------------|---------|
| `FollowRequestedDomainEvent` | `FollowId`, `FollowerId`, `FolloweeId` | Follow creado (siempre) |
| `FollowAcceptedDomainEvent` | `FollowId`, `FollowerId`, `FolloweeId` | Follow auto-aceptado (Public) o aprobado manualmente |
| `FollowRejectedDomainEvent` | `FollowId`, `FollowerId`, `FolloweeId` | Followee rechaza request |
| `FollowRemovedDomainEvent` | `FollowId`, `FollowerId`, `FolloweeId`, `RemovedBy` | Follow o request eliminado |
| `FollowBlockedDomainEvent` | `FollowId`, `FollowerId`, `FolloweeId`, `BlockedBy` | Bloqueo ejecutado |
| `FollowUnblockedDomainEvent` | `FollowId`, `FollowerId`, `FolloweeId`, `UnblockedBy` | Desbloqueo ejecutado |
| `ProfileCreatedDomainEvent` | `ProfileId` | Profile creado al registrarse |

---

## Permissions

| Permission constant | Value | Operations |
|--------------------|-------|-----------|
| `Permissions.FollowUser` | `Social.Follow` | `POST /social/follows/{targetUserId}` |
| `Permissions.ManageFollowers` | `Social.ManageFollowers` | Approve, Reject, RemoveFollow |
| `Permissions.BlockUser` | `Social.Block` | Block, Unblock |
| `Permissions.ViewSocial` | `Social.View` | GetFollowers, GetFollowing, GetFollowStatus |
