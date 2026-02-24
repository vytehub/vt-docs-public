---
title: "06. Commerce — Orders, Payments & Refunds"
description: "P0 bounded context: economic commitments, payment processing, refunds, disputes, and payouts."
---

# 06. Commerce — Orders, Payments & Refunds

> **Status: P0 — Required for production bookings with payments.**

---

## Responsibility

The Commerce bounded context owns the **economic side** of every transaction in VyteMerge:
- Orders (what was bought, at what price, with what discounts)
- Payments (capturing money from a buyer)
- Refunds (returning money, in full or partial)
- Disputes (contested transactions)
- Payouts (sending money to providers)

Commerce does NOT manage:
- Booking lifecycle (that is the booking flow / `Timelines`)
- Cancellation rules that trigger refunds (those are `Policies`; Commerce executes the refund)
- Service definitions or pricing rules (those are `Offering`)

---

## Key Entities

### Order
An Order represents the economic commitment for a purchase.

| Field | Description |
|---|---|
| `orderId` | Unique identifier |
| `profileId` | Buyer's Profile |
| `providerId` | Provider's Profile |
| `lineItems` | List of `OrderLineItem` (listing, qty, unit price, discounts) |
| `subtotal` | Sum before discounts |
| `discountAmount` | Total discount applied |
| `total` | Final amount charged |
| `currency` | ISO 4217 (e.g., ARS, USD) |
| `status` | `Pending \| Paid \| Refunded \| PartiallyRefunded \| Disputed \| Cancelled` |
| `bookingId` | Optional — if the order is for a Service Listing booking |
| `paymentId` | Reference to the Payment |
| `createdAt` | ISO8601 |

### OrderLineItem
A single item in an Order.

| Field | Description |
|---|---|
| `listingId` | The Listing being purchased |
| `description` | Snapshot of listing title at purchase time |
| `unitPrice` | Price at purchase time (snapshot — not live) |
| `quantity` | Number of units |
| `discountAmount` | Item-level discount |
| `total` | `(unitPrice × quantity) - discountAmount` |

### Payment
Represents a payment attempt and its result.

| Field | Description |
|---|---|
| `paymentId` | Unique identifier |
| `orderId` | The Order being paid |
| `method` | `card \| transfer \| credit \| external` |
| `amount` | Amount charged |
| `currency` | ISO 4217 |
| `status` | `Pending \| Captured \| Failed \| Refunded` |
| `gatewayReference` | External payment gateway transaction ID |
| `capturedAt` | ISO8601 |

### Refund
A full or partial return of a Payment.

| Field | Description |
|---|---|
| `refundId` | Unique identifier |
| `paymentId` | The original Payment |
| `amount` | Amount refunded |
| `reason` | `cancellation \| dispute \| goodwill \| other` |
| `initiatedBy` | `provider \| customer \| system \| platform` |
| `status` | `Pending \| Processed \| Failed` |

---

## Commands

| Command | Description |
|---|---|
| `CreateOrder` | Creates an Order from a Listing purchase (or Booking confirmation with payment). |
| `CapturePayment` | Captures payment for an Order (integrates with payment gateway). |
| `IssueRefund` | Issues a full or partial refund for a Payment. |
| `OpenDispute` | Customer contests a transaction. |
| `ResolveDispute` | Platform or provider resolves a dispute. |
| `InitiatePayout` | Transfers provider funds after service completion. |

## Events

| Event | Description |
|---|---|
| `OrderCreated` | Order registered in the system. |
| `PaymentCaptured` | Payment successfully captured. |
| `PaymentFailed` | Payment attempt failed. |
| `RefundIssued` | Refund processed. |
| `DisputeOpened` | Customer contested a transaction. |
| `DisputeResolved` | Dispute outcome recorded. |
| `PayoutInitiated` | Provider payout started. |

---

## Integration with Booking

```
Booking flow:
  [User confirms booking] → CreateOrder → CapturePayment
                         → PaymentCaptured → BookingConfirmed
                         → (if payment fails) → BookingCancelled

Cancellation flow:
  [CancelBooking] → check Policies → IssueRefund (if eligible)
                 → RefundIssued → BookingCancelled
```

See `07-policies/index.md` for the rules that determine refund eligibility.

---

## Relationship to "Devolución por Compromiso"

The "Devolución por Compromiso" anti-no-show mechanism (defined in `core-model.md`) is implemented via Commerce:
- On booking: `CreateOrder` with a `commitment_amount`.
- On attendance confirmed: `IssueRefund` (returns the committed amount as credit or cash).
- On no-show: refund withheld (per policy).

---

## Open Questions (P0 items to resolve before implementation)

- [ ] Which payment gateway? (MercadoPago, Stripe, manual transfer?)
- [ ] Is VyteMerge a marketplace (collects then pays out) or a referral (directs to provider)?
- [ ] How are partial refunds calculated when a group booking is partially cancelled?
- [ ] What is the payout schedule? (T+N days after service completion)
- [ ] How are disputes escalated to platform admins?

---

## References

- Core model (Order entity): `00-core-domain/03-core-model/core-model.md`
- Cancellation policies: `07-policies/index.md`
- Booking lifecycle: `01-domain-behavior/02-lifecycles/booking-lifecycle.md`
