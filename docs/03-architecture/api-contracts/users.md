# API Contracts — Users & Profiles

Base path: `/api/v1`

---

## Users (account self-service)

### `GET /users/me` ✅
Returns the current authenticated user's account.

**Response:**
```json
{
  "userId": "uuid",
  "keycloakSub": "string",
  "email": "string",
  "displayName": "string",
  "locale": "string",
  "timezoneId": "string",
  "createdAt": "ISO8601"
}
```

---

### `PATCH /users/me` 🚧
Updates account-level preferences (locale, timezone, notification settings).

**Request:**
```json
{
  "locale": "string",
  "timezoneId": "string"
}
```

**Command dispatched:** `UpdateUserPreferences`

---

## Profiles

### `GET /profiles/:profileId` ✅
Returns a Profile. Visibility is enforced by the Access module.

**Response (full):**
```json
{
  "profileId": "uuid",
  "userId": "uuid",
  "displayName": "string",
  "bio": "string",
  "avatarUrl": "string",
  "type": "person | organization",
  "visibility": "public | private",
  "followersCount": 0,
  "followingCount": 0
}
```

**Response (BusyOnly — non-follower):**
```json
{
  "profileId": "uuid",
  "displayName": "string",
  "avatarUrl": "string",
  "type": "person | organization"
}
```

---

### `GET /profiles/me` ✅
Returns the authenticated user's own Profile(s).

---

### `PATCH /profiles/:profileId` 🚧
Updates profile metadata. Only the owner can update.

**Request:**
```json
{
  "displayName": "string",
  "bio": "string",
  "avatarUrl": "string"
}
```

**Command dispatched:** `UpdateProfile`

---

### `POST /profiles/:profileId/follow` 🚧
Follow a Profile.

**Response:**
```json
{ "status": "followed | requested" }
```

**Command dispatched:** `RequestFollow` or `CreateFollow` (depending on target's `follow_requires_approval`).

---

### `DELETE /profiles/:profileId/follow` 🚧
Unfollow a Profile.

**Command dispatched:** `Unfollow`

---

### `POST /profiles/:targetId/follow/approve` 🚧
Approve a pending follow request. Only the target profile owner can call this.

**Command dispatched:** `ApproveFollow`

---

### `POST /profiles/:targetId/follow/reject` 🚧
Reject a pending follow request.

**Command dispatched:** `RejectFollow`

---

### `POST /profiles/:profileId/block` 🚧
Block a Profile.

**Command dispatched:** `BlockProfile`

---

## Posts

### `GET /profiles/:profileId/posts` 🚧
List posts by a Profile. Paginated.

**Query params:** `?page=1&pageSize=20`

---

### `POST /profiles/:profileId/posts` 🚧
Create a Post.

**Request:**
```json
{
  "text": "string",
  "mediaUrls": ["string"],
  "listingId": "uuid | null"
}
```

**Command dispatched:** `CreatePost`
**Event emitted:** `PostCreated`

---

### `POST /posts/:postId/reactions` 🚧
React to a Post.

**Request:**
```json
{ "type": "like | ... " }
```

**Command dispatched:** `ReactToPost`
