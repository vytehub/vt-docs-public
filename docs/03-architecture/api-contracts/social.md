# API Contracts — Social

Base path: `/api/v1`
Module: `Vt.Modules.Social` *(nuevo — a implementar)*
Schema: `social`

> Cubre Profiles, Follow graph, Posts y Reactions.
> Ver flows `03-follow-and-feed.md` y spec pack `DRAFT-social-feed`.
> Visibilidad controlada por `VisibilityMode`: `None | BusyOnly | BusyOnlyDetails` (módulo Access).

---

## Profiles

### `GET /profiles/:profileId` 🚧

Retorna un Profile. Respuesta varía según `VisibilityMode` del actor autenticado.

**[public — visibility-aware]**

**Response (full — BusyOnlyDetails o seguidor):**
```json
{
  "id": "uuid",
  "userId": "uuid",
  "name": "string",
  "bio": "string | null",
  "avatarUrl": "string | null",
  "privacy": "Public | Private",
  "followerCount": 0,
  "followingCount": 0,
  "isFollowing": false,
  "isFollowedBy": false,
  "followStatus": "none | pending | accepted",
  "isBlocked": false,
  "createdAt": "ISO8601"
}
```

**Response (redactado — BusyOnly):**
```json
{
  "id": "uuid",
  "name": "string",
  "avatarUrl": "string | null",
  "privacy": "Public | Private"
}
```

**Response (sin acceso — None):** `404 Not Found`

---

### `PATCH /profiles/me` 🚧

Actualiza el Profile del usuario autenticado.

**Request (partial):**
```json
{
  "name": "string",
  "bio": "string | null",
  "avatarUrl": "string | null",
  "privacy": "Public | Private"
}
```

**Command dispatched:** `UpdateProfileCommand`
**Event emitted:** `ProfileUpdated`

---

## Follow Graph

### `POST /profiles/:profileId/follow` 🚧

Envía un Follow Request al Profile target.
- Si target `privacy = Public` → acepta automáticamente → `FollowAccepted`
- Si target `privacy = Private` → queda en `Pending` → `FollowRequested`

**Command dispatched:** `FollowProfileCommand`
**Events emitted:** `FollowRequested` | `FollowAccepted`

**Response:** `200 OK`
```json
{ "followStatus": "pending | accepted" }
```

---

### `DELETE /profiles/:profileId/follow` 🚧

Deja de seguir a un Profile (o cancela un Follow Request pendiente).

**Command dispatched:** `UnfollowProfileCommand`
**Event emitted:** `FollowRemoved`

---

### `POST /follow-requests/:requestId/accept` 🚧

El target acepta un Follow Request pendiente.

**Command dispatched:** `AcceptFollowRequestCommand`
**Event emitted:** `FollowAccepted`

---

### `POST /follow-requests/:requestId/reject` 🚧

El target rechaza un Follow Request pendiente.

**Command dispatched:** `RejectFollowRequestCommand`
**Event emitted:** `FollowRejected`

---

### `GET /follow-requests` 🚧

Lista Follow Requests pendientes recibidos por el usuario autenticado.

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "fromProfileId": "uuid",
      "fromProfileName": "string",
      "fromAvatarUrl": "string | null",
      "requestedAt": "ISO8601"
    }
  ],
  "total": 0
}
```

---

### `GET /profiles/:profileId/followers` 🚧

Lista los seguidores de un Profile.

**Query params:** `?page=1&pageSize=20`

**Response:**
```json
{
  "items": [
    { "id": "uuid", "name": "string", "avatarUrl": "string | null" }
  ],
  "total": 0,
  "page": 1,
  "pageSize": 20
}
```

---

### `GET /profiles/:profileId/following` 🚧

Lista los Profiles que sigue un Profile. Mismo shape que `/followers`.

---

## Block / Mute

### `POST /profiles/:profileId/block` 🚧

Bloquea un Profile. Elimina el Follow en ambas direcciones.
El bloqueado no puede ver el Profile del bloqueador ni enviarle Follow Requests.

**Command dispatched:** `BlockProfileCommand`
**Event emitted:** `ProfileBlocked`

---

### `DELETE /profiles/:profileId/block` 🚧

Desbloquea un Profile.

**Command dispatched:** `UnblockProfileCommand`
**Event emitted:** `ProfileUnblocked`

---

### `POST /profiles/:profileId/mute` 🚧

Silencia el Feed del Profile target (posts no aparecen en el feed del actor).
No afecta el Follow graph; el target no se entera.

**Command dispatched:** `MuteProfileCommand`

---

### `DELETE /profiles/:profileId/mute` 🚧

Quita el mute.

---

## Posts & Feed

### `GET /feed` 🚧

Retorna el Feed personalizado del usuario autenticado (posts de Profiles seguidos).

**Query params:** `?page=1&pageSize=20&cursor=string`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "profileId": "uuid",
      "profileName": "string",
      "profileAvatarUrl": "string | null",
      "type": "text | image | video | listing_share",
      "content": "string | null",
      "media": [{ "url": "string", "type": "image | video" }],
      "listingId": "uuid | null",
      "reactionCounts": { "like": 0, "love": 0 },
      "myReaction": "string | null",
      "commentCount": 0,
      "createdAt": "ISO8601"
    }
  ],
  "nextCursor": "string | null"
}
```

---

### `GET /profiles/:profileId/posts` 🚧

Lista los Posts públicos de un Profile (respeta visibilidad).

**Query params:** `?page=1&pageSize=20`

**Response:** mismo shape que `GET /feed` sin personalización.

---

### `POST /posts` 🚧

Crea un Post en el Profile del usuario autenticado.

**Request:**
```json
{
  "profileId": "uuid",
  "type": "text | image | video | listing_share",
  "content": "string | null",
  "media": [{ "url": "string", "type": "image | video" }],
  "listingId": "uuid | null",
  "visibility": "Public | FollowersOnly | Private"
}
```

**Command dispatched:** `CreatePostCommand`
**Event emitted:** `PostPublished`

**Response:** `201 Created` con el Post creado.

---

### `DELETE /posts/:postId` 🚧

Elimina un Post propio.

**Command dispatched:** `DeletePostCommand`
**Event emitted:** `PostDeleted`

---

## Reactions

### `POST /posts/:postId/reactions` 🚧

Agrega una Reaction a un Post.

**Request:**
```json
{ "type": "like | love | wow | haha | sad | angry" }
```

**Command dispatched:** `AddReactionCommand`
**Event emitted:** `ReactionAdded`

> Un usuario solo puede tener una Reaction activa por Post. Volver a enviar reemplaza la anterior.

---

### `DELETE /posts/:postId/reactions` 🚧

Elimina la Reaction del usuario autenticado en un Post.

**Command dispatched:** `RemoveReactionCommand`
**Event emitted:** `ReactionRemoved`
