# API Contracts

This directory contains endpoint skeletons for each bounded context.

Each file lists the main REST routes: HTTP method, path, the command or query it dispatches, and the expected response shape.

These are **design-level contracts** — the canonical source of truth for what the API should look like. They are not auto-generated; they are curated alongside the domain docs.

## Files

| File | Bounded Context |
|---|---|
| `users.md` | Foundation — Users |
| `offering.md` | Offer — Catalog & Listings |
| `timelines.md` | Supply — Timelines & Slots |
| `social.md` | Social & Discovery |
| `booking.md` | Booking flow (cross-context) |
| `notifications.md` | Communication |

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
