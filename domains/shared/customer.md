---
entity: customer
type: shared
used_by: [crm, pos, accounting, shipping]
version: 1.0
---

# Customer (Shared Entity)

The customer entity appears across nearly every domain. This file defines the common base. Domain-specific extensions are defined in each variant spec.

## Base Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | yes | Unique identifier |
| name | string | yes | Full name |
| email | string | no | Primary email |
| phone | string | no | Primary phone (with country code) |
| created_at | datetime | yes | When the customer record was created |
| source | string | yes | How this customer was acquired (web, event, referral, whatsapp, etc.) |
| source_detail | string | no | Specific source (campaign name, event name, referral code) |

At least one of `email` or `phone` is required. Which one is primary depends on the market — email-first in the US/EU, phone-first in India and emerging markets.

## Common Extensions by Domain

| Domain | Additional Fields |
|--------|------------------|
| CRM | lifetime_value, order_count, last_order_date, segment, acquisition_channel |
| POS | loyalty_points, visit_count, preferred_location |
| Accounting | billing_address, tax_id, payment_terms |
| Shipping | shipping_addresses[], default_address |

## Deduplication

Customers acquired through different channels often create duplicate records. A customer who first came through a WhatsApp message and later ordered through the website is one customer with two records.

Dedup strategy:
- **Phone match** (strongest signal, especially in markets where phone is primary ID)
- **Email match** (strong signal)
- **Name + address match** (weak signal, use for flagging not auto-merging)

At seed stage, dedup is manual. At growth, automated matching with manual review for ambiguous cases. At scale, probabilistic matching with confidence scores.
