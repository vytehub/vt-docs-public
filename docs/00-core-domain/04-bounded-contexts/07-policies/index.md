---
title: "07. Policies — Cancellation, Reschedule & No-show"
description: "P0 bounded context: rules governing cancellation windows, reschedule conditions, no-show enforcement, and their economic consequences."
---

# 07. Policies — Cancellation, Reschedule & No-show

> **Status: P0 — Required to complete the Booking lifecycle.**

---

## Responsibility

The Policies bounded context defines the **rules** that govern how bookings can be modified, cancelled, and what happens when a party doesn't comply.

Policies does NOT:
- Execute payments or refunds (that is `Commerce`)
- Manage booking state (that is the booking flow)
- Define what a service is (that is `Offering`)

Policies **answers the question**: "Given this booking and this action, what rules apply and what are the consequences?"

---

## Policy Types

### Cancellation Policy

Defines the conditions and consequences of cancelling a Booking.

| Field | Description |
|---|---|
| `policyId` | Unique identifier |
| `listingId` | The Listing this policy applies to |
| `cancellableBy` | `attendee \| provider \| both` |
| `windows` | List of `CancellationWindow` (ordered by priority) |
| `appliesTo` | `all \| first_booking \| repeat_bookings` |

#### CancellationWindow

| Field | Description |
|---|---|
| `hoursBeforeStart` | Cancellations more than N hours before start apply this rule |
| `refundPercent` | Percentage of payment to refund (0–100) |
| `creditOnly` | If true, refund is platform credit, not cash |
| `penaltyAmount` | Fixed penalty charged to the cancelling party (optional) |

**Example:**
```
- More than 48h before: 100% refund
- 24–48h before: 50% refund
- Less than 24h before: no refund
```

---

### Reschedule Policy

Defines if and how a Booking can be moved to a different slot.

| Field | Description |
|---|---|
| `reschedulableBy` | `attendee \| provider \| both` |
| `windowHours` | Minimum hours before start that reschedule is allowed |
| `maxReschedules` | Maximum number of reschedules per booking (0 = not allowed) |
| `samePriceRequired` | If true, the new slot must have the same price |

---

### No-show Policy

Defines what happens when an attendee or provider is a no-show.

| Field | Description |
|---|---|
| `attendeeNoShow.penaltyPercent` | Percentage of payment forfeited (0–100) |
| `attendeeNoShow.creditImpact` | Whether no-show affects platform credit balance |
| `providerNoShow.refundPercent` | Refund to attendee when provider no-shows (typically 100%) |
| `providerNoShow.platformAction` | `none \| warning \| suspension` |

---

## Commands

| Command | Description |
|---|---|
| `AttachPolicyToListing` | Associates a Policy with a Listing. |
| `UpdateCancellationPolicy` | Modifies the cancellation windows for a Listing. |
| `UpdateReschedulePolicy` | Modifies reschedule rules for a Listing. |
| `EvaluateCancellation` | Given a booking and current time, returns the applicable consequences. |
| `EvaluateReschedule` | Returns whether rescheduling is allowed and under what terms. |

## Events

| Event | Description |
|---|---|
| `PolicyAttached` | A Policy was linked to a Listing. |
| `PolicyUpdated` | Policy rules changed. |
| `CancellationEvaluated` | System evaluated and recorded the cancellation consequence. |

---

## Integration with Booking & Commerce

```
CancelBooking command:
  → Policies.EvaluateCancellation(bookingId, cancelledBy, cancelledAt)
  → returns: { refundAmount, penaltyAmount, refundType }
  → Commerce.IssueRefund(paymentId, refundAmount)
  → Booking transitions to Cancelled

MarkNoShow command:
  → Policies.EvaluateNoShow(bookingId, noShowType)
  → returns: { penaltyAmount, refundAmount }
  → Commerce applies consequences
  → Booking transitions to NoShow
```

---

## Default Platform Policies

When a provider has not configured a Cancellation Policy, the platform default applies:

| Window | Refund |
|---|---|
| > 24h before | 100% |
| ≤ 24h before | 0% |

Providers may set stricter or more lenient policies within platform limits (TBD).

---

## Open Questions (P0 items to resolve)

- [ ] What are the platform-level maximum strictness limits for cancellation policies?
- [ ] Can providers override the default no-show penalty?
- [ ] How are reschedules handled when the new slot has different pricing?
- [ ] Who pays the reschedule window enforcement? (system automatic or manual provider action?)
- [ ] How are group booking cancellations handled? (one attendee cancels out of N?)

---

## References

- Booking lifecycle: `01-domain-behavior/02-lifecycles/booking-lifecycle.md`
- Booking rules (Listing): `03.Offer - Catalog & Listings/02.listing/05.booking_rules.md`
- Commerce (refunds): `06-commerce/index.md`
- Incentivos anti-cancelación: `02-product-variants/use-cases/incentivos-anticancelacion-noshow.md`
