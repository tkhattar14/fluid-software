---
entity: payment
type: shared
used_by: [crm, pos, accounting, invoicing]
version: 1.0
---

# Payment (Shared Entity)

Payment processing appears across CRM (order payments), POS (in-store transactions), accounting (revenue tracking), and invoicing (B2B payments).

## Base Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | yes | Unique identifier |
| gateway_id | string | yes | ID from the payment provider (Razorpay payment_id, Stripe charge_id) |
| amount | decimal | yes | Payment amount |
| currency | string | yes | ISO 4217 currency code |
| status | enum | yes | pending, authorized, captured, failed, refunded |
| method | string | no | card, upi, netbanking, wallet, bank_transfer |
| customer_id | string | yes | Reference to customer |
| created_at | datetime | yes | When the payment was initiated |
| captured_at | datetime | no | When the payment was captured (may differ from created) |

## Payment Flow

```
INITIATED → AUTHORIZED → CAPTURED → (optional: REFUNDED)
                ↓
             FAILED
```

**Authorized vs. Captured:** Some gateways separate authorization (funds reserved) from capture (funds transferred). At seed stage, auto-capture is fine. At scale, manual capture allows for fraud review before finalizing.

## Webhook Pattern

All payment gateways communicate status changes via webhooks. The CRM/POS must:
1. Receive the webhook
2. Verify the signature (gateway-specific)
3. Update the payment record
4. Trigger downstream workflows (order creation, receipt, fulfillment)

**Idempotency is critical.** Gateways may send the same webhook multiple times. The handler must be idempotent — processing the same event twice should not create duplicate orders or send duplicate notifications.

## Gateway Abstraction

At seed stage, you pick one gateway and hardcode it. At growth, abstract behind an interface so you can:
- Switch gateways without rewriting business logic
- Support multiple gateways for different markets (Razorpay for India, Stripe for international)
- Handle gateway-specific quirks in one place

## Compliance

- Never store raw card numbers. Use gateway tokens.
- PCI-DSS applies if handling card data directly (most D2C businesses avoid this by using hosted checkout).
- Retain payment records for tax/audit purposes per local law (typically 7+ years).
- Refund records must be linked to original payment for accounting reconciliation.
