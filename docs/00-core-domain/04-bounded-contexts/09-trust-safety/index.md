---
title: "09. Trust & Safety — Reports, Moderation & Reviews"
description: "P0 bounded context: content moderation, user reports, blocking, review system, and platform safety enforcement."
---

# 09. Trust & Safety — Reports, Moderation & Reviews

> **Status: P0 — Required before general availability of social and marketplace features.**

---

## Responsibility

The Trust & Safety bounded context defines how VyteMerge maintains a safe and trustworthy environment.

It answers:
> **Is this content or behavior acceptable? What action should the platform take?**

Trust & Safety does NOT:
- Manage social graph (that is `Social`)
- Manage account deactivation at the platform level (that is platform ops)
- Define commercial terms (that is `Policies`)

---

## Subdomains

### 1. Reports

Users report content or behavior that violates platform rules.

#### Report

| Field | Description |
|---|---|
| `reportId` | Unique identifier |
| `reporterProfileId` | Who is reporting |
| `targetType` | `profile \| post \| listing \| booking \| review` |
| `targetId` | The reported entity |
| `reason` | `spam \| harassment \| inappropriate_content \| fraud \| fake_account \| other` |
| `details` | Optional free text |
| `status` | `open \| under_review \| resolved \| dismissed` |
| `resolution` | `no_action \| content_removed \| account_warned \| account_suspended \| account_banned` |
| `createdAt` | ISO8601 |

---

### 2. Blocking

Users can block other Profiles to prevent all interaction.

#### Block

| Field | Description |
|---|---|
| `blockId` | Unique identifier |
| `blockerProfileId` | Who initiated the block |
| `blockedProfileId` | Who is blocked |
| `reason` | Optional private note |
| `createdAt` | ISO8601 |

**Effects of a block:**
- Blocked Profile cannot follow the blocker.
- Blocked Profile cannot see the blocker's Listings, Posts, or calendar.
- Blocked Profile cannot create a Booking with the blocker.
- Existing messages are hidden.
- Block is not visible to the blocked party (they see `NotFound` responses).

---

### 3. Moderation

Moderation actions taken by platform staff or automated systems.

#### ModerationAction

| Field | Description |
|---|---|
| `actionId` | Unique identifier |
| `targetType` | `profile \| post \| listing \| review` |
| `targetId` | The moderated entity |
| `action` | `hide \| remove \| warn \| suspend \| ban` |
| `reason` | Internal reason |
| `initiatedBy` | `report \| proactive \| automated` |
| `reviewedBy` | Platform staff ProfileId (if human review) |
| `appealStatus` | `not_appealed \| under_appeal \| appeal_upheld \| appeal_dismissed` |
| `createdAt` | ISO8601 |

---

### 4. Reviews

Verified reviews from booking attendees about providers.

#### Review

| Field | Description |
|---|---|
| `reviewId` | Unique identifier |
| `bookingId` | The Booking this review is for (must be `Completed`) |
| `reviewerProfileId` | Attendee who left the review |
| `providerId` | Provider being reviewed |
| `listingId` | The Listing being reviewed |
| `rating` | Integer 1–5 |
| `text` | Optional review text |
| `status` | `visible \| hidden \| removed` |
| `createdAt` | ISO8601 |

**Rules:**
- Only attendees of a `Completed` Booking can leave a review.
- One review per `Booking`.
- Review window: 14 days after `Booking.Completed` (configurable).
- Reviews are publicly visible by default.
- Providers can respond to reviews (future).

---

## Commands

| Command | Description |
|---|---|
| `ReportContent` | User submits a report on a Profile, Post, Listing, etc. |
| `ReviewReport` | Moderator reviews a report and records a resolution. |
| `BlockProfile` | User blocks another Profile. |
| `UnblockProfile` | User removes a block. |
| `TakeModerationAction` | Platform takes a moderation action on an entity. |
| `AppealModerationAction` | User appeals a moderation decision. |
| `ResolveAppeal` | Moderator resolves an appeal. |
| `SubmitReview` | Verified attendee submits a review after a Completed Booking. |
| `HideReview` | Moderator hides a review (abuse, spam). |

## Events

| Event | Description |
|---|---|
| `ReportSubmitted` | A report was filed. |
| `ReportResolved` | A report was resolved by a moderator. |
| `ProfileBlocked` | A Profile was blocked. |
| `ProfileUnblocked` | A block was removed. |
| `ModerationActionTaken` | Platform acted on an entity. |
| `ReviewSubmitted` | A review was published. |
| `ReviewHidden` | A review was hidden by moderation. |

---

## Integration with Access

The `Access` module queries Trust & Safety data when evaluating visibility:

```
AccessRequest(actorId: A, resource: ProfileB, action: View)
  → Access checks: has A blocked B? has B blocked A?
  → if blocked: returns Denied (regardless of other rules)
```

Blocks are enforced at the Access layer so they propagate consistently across all features.

---

## Automated Safety Signals

Future: automated detection for spam, hate speech, and fraudulent activity can feed into the `ReportContent` command with `initiatedBy = automated`. Human review is required before any action beyond `hide`.

---

## Open Questions (P0 items to resolve)

- [ ] Who are the human moderators? (platform staff? community moderators? provider self-moderation?)
- [ ] What is the appeals process SLA? (24h? 72h?)
- [ ] Are review responses by providers in scope for MVP?
- [ ] What is the trust score model? (aggregate rating? badges? verification?)
- [ ] How are reports prioritized? (severity levels? queue management?)
- [ ] What constitutes grounds for account suspension vs. permanent ban?

---

## References

- Social privacy/moderation: `04.Social & Discovery/01.social/06.privacy-moderation.md`
- Access module: `backend/vt-core/src/Modules/Access/readme.md`
- Block command: `Social module readme`
