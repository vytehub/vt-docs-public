# API Contracts

This directory contains endpoint skeletons for each bounded context.

Each file lists the main REST routes: HTTP method, path, the command or query it dispatches, and the expected response shape.

These are **design-level contracts** — the canonical source of truth for what the API should look like. They are not auto-generated; they are curated alongside the domain docs.

## Files

| File | Bounded Context |
|---|---|
| `users.md` | Foundation — Users & Onboarding |
| `catalog.md` | Catalog — Services & Products |
| `listing.md` | Listing — Offer configuration & publication |
| `places.md` | Supply — Places (Physical, Online, Community) |
| `timelines.md` | Supply — Timelines & Slots |
| `agreements.md` | Foundation — Agreements (Sharing, Delegation, Partner) |
| `booking.md` | Booking — Lifecycle (cross-context) |
| `policies.md` | Policies — Cancellation, Reschedule, NoShow |
| `social.md` | Social — Profiles, Follow, Feed, Posts |
| `search.md` | Search & Discovery — Listings, Autocomplete, Discover |
| `notifications.md` | Communication — Notifications & Preferences |

## Conventions

- Base path: `/api/v1/`
- Authentication: all endpoints require a valid Keycloak Bearer token unless marked `[public]`.
- Actor resolution: `sub` claim in the JWT resolves to `UserId` and then to `ProfileId`.
- Error format:
  ```json
  { "type": "string", "title": "string", "status": 400, "detail": "string", "traceId": "string" }
  ```
- Pagination: `?page=1&pageSize=20` for list endpoints; response includes `{ items: [], total, page, pageSize }`.

## Status legend

| Symbol | Meaning |
|---|---|
| ✅ | Implemented |
| 🚧 | Planned / in progress |
| ❌ | Not started |
