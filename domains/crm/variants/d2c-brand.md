---
domain: crm
variant: d2c-brand
status: real
contributor: "Tushar Khattar"
version: 1.0

stages:
  seed:
    description: "0-500 customers, 1-2 person team, <50 orders/month"
    architecture:
      database: sqlite
      deployment: single-server
      pattern: monolith
  growth:
    description: "500-5K customers, 3-5 person team, 50-500 orders/month"
    architecture:
      database: postgresql
      deployment: api-server + background-workers
      pattern: modular-monolith
  scale:
    description: "5K+ customers, dedicated ops team, 500+ orders/month"
    architecture:
      database: postgresql-read-replicas
      deployment: api + workers + cdn + cache
      pattern: event-driven

entities:
  - name: customer
    fields: [name, email, phone, channel, source, acquisition_date]
    relationships: [orders, recipients, segments, leads]
    shared: true

  - name: recipient
    fields: [name, age, gender, preferences, relationship_to_customer]
    relationships: [customer, orders]
    note: "D2C brands selling personalized or gifted products often have a buyer (customer) and a separate recipient. A parent buying for multiple children. A corporate buyer ordering for employees."

  - name: order
    fields: [products, quantity, amount, status, source, date, fulfillment_status]
    relationships: [customer, recipient, payment, shipment]

  - name: lead
    fields: [name, contact, source, utm_params, form_step, score, status]
    relationships: [customer]
    note: "Progressive capture — leads are created at first interaction and enriched as they move through the funnel"

  - name: segment
    fields: [name, criteria, customer_count]
    relationships: [customers, campaigns]

  - name: campaign
    fields: [name, channel, segment, status, sent_count, conversion_rate]
    relationships: [segment, orders]

workflows:
  - name: lead-capture-to-conversion
    trigger: visitor lands on site or event booth
    stages: [seed, growth, scale]
  - name: order-fulfillment
    trigger: payment.completed
    stages: [seed, growth, scale]
  - name: post-purchase-communication
    trigger: order.shipped OR order.delivered
    stages: [seed, growth, scale]
  - name: retention-campaigns
    trigger: scheduled OR segment.criteria_met
    stages: [growth, scale]
  - name: abandoned-cart-recovery
    trigger: lead.status == abandoned AND lead.score > threshold
    stages: [growth, scale]

integrations:
  - name: payment-gateway
    examples: [razorpay, stripe, square]
    required: true
  - name: shipping-provider
    examples: [shiprocket, shippo, delhivery]
    required: true
    stages: [seed, growth, scale]
  - name: whatsapp-business
    examples: [whatsapp-cloud-api, wati, interakt]
    required: false
    note: "Critical for D2C in India and emerging markets. Post-purchase updates, order status, re-engagement."
  - name: social-media
    examples: [instagram-api, meta-ads-api]
    required: false
    stages: [growth, scale]
  - name: analytics
    examples: [mixpanel, posthog, google-analytics]
    required: false
  - name: invoicing
    examples: [zoho-books, quickbooks, tally]
    required: true
    stages: [growth, scale]
---

# D2C Brand CRM

A CRM for direct-to-consumer brands selling physical products online, at events, and through messaging channels. This spec is written from real experience running a D2C personalized products business.

## What Makes D2C CRM Different

The customer is not always the end user. A parent buys a product for their child. A corporate buyer orders gifts for employees. This creates a **buyer-recipient split** that most generic CRMs don't handle. You need to track both: the customer (who pays) and the recipient (who the product is for).

D2C also has extreme channel diversity. Orders come from your website, from Instagram DMs, from WhatsApp messages, from physical events and stalls, from referrals. Each channel has different data quality — a website order has clean structured data; a WhatsApp order is a messy conversation that needs to be parsed into an order.

## Entities in Detail

### Customer

The person who pays. Core fields:
- Name, email, phone (at least one contact method required)
- Acquisition channel (website, whatsapp, event, referral, instagram)
- Acquisition source (specific campaign, event name, referral code)
- UTM parameters (for web-originated customers)
- Lifetime value (calculated: sum of all order amounts)
- Order count

A customer can have multiple recipients. A mother buying books for three children. A company ordering gifts for a team. The customer-recipient relationship is one-to-many and is the single most important data model decision in D2C CRM.

### Recipient

The person the product is for. This entity doesn't exist in most CRM tools, which is why D2C brands constantly fight their CRM.

- Name, age/DOB, gender
- Preferences (interests, themes, sizes — varies by product type)
- Relationship to customer (child, spouse, employee, friend)
- Reference media (photo, measurements — for personalized products)

### Lead

A visitor who has shown intent but hasn't purchased. Progressive capture is essential — don't ask for everything upfront.

**Lead lifecycle:**
```
Landing page visit → Lead created (NEW)
    ↓
Step 1: Product preferences → IN_PROGRESS
    ↓
Step 2: Contact details → IN_PROGRESS
    ↓
Step 3: Shipping address → IN_PROGRESS
    ↓
Payment → CONVERTED  |  Leaves → ABANDONED  |  WhatsApp → WHATSAPP_ORDER
```

**Lead scoring (0-100):**
| Signal | Points |
|--------|--------|
| Product preferences completed | 15 |
| Contact details provided | 25 |
| Shipping address provided | 15 |
| Reached payment step | 20 |
| Time on site > 3 minutes | 10 |
| Returned visitor | 15 |

Leads scoring above 50 that go ABANDONED should trigger follow-up (WhatsApp message at seed stage, automated sequence at growth stage).

### Order

The core transaction. One payment → one order → one or more products.

**Order sources:**
- Website checkout (cleanest data)
- WhatsApp order (requires manual or AI-assisted parsing)
- Event/stall order (captured on tablet or phone, often with spotty connectivity)
- Admin-created (phone orders, special requests)

**Order statuses:**
`pending` → `confirmed` → `processing` → `shipped` → `delivered`

With branches: `cancelled`, `refunded`, `returned`

**Multi-product orders:** A single order can contain multiple products for multiple recipients. Example: a parent orders a personalized book for each of their three children. That's one order, one payment, three products, three recipients.

### Segment

A reusable filter over the customer base. Examples:
- "Repeat buyers" (order_count > 1)
- "Event customers" (acquisition_channel == event)
- "High value" (lifetime_value > threshold)
- "Dormant" (last_order_date > 90 days ago)

At seed stage, segments are manual. At growth stage, they're rule-based. At scale, they're ML-driven with lookalike audiences.

## Workflows in Detail

### 1. Lead Capture to Conversion

**Seed stage:** Simple landing page with form. Leads captured in a spreadsheet or basic database. Follow-up via personal WhatsApp message. Conversion is manual — you're talking to every lead yourself.

**Growth stage:** Multi-step form with progressive capture. Leads stored in the CRM with scoring. Abandoned leads get automated WhatsApp/email follow-up after 1 hour, 24 hours, 72 hours. A/B testing on follow-up messages.

**Scale stage:** Full funnel automation. Lead scoring drives prioritization. High-intent leads get immediate outreach. Segment-based nurture sequences. Attribution tracking across channels (UTM → segment → campaign → conversion).

### 2. Order Fulfillment

**Seed stage:** Payment webhook triggers order creation. Founder manually reviews, creates shipping label, packs and ships. Tracking number sent to customer via WhatsApp.

**Growth stage:** Automated pipeline — payment confirmed → order queued → production triggered → shipping label auto-generated → tracking notification sent. Exception handling for failed payments, address issues, out-of-stock.

**Scale stage:** Batch processing. Daily fulfillment runs. Multi-warehouse routing. Automated carrier selection based on destination, weight, and cost. Real-time inventory sync.

### 3. Post-Purchase Communication

This is where D2C brands build loyalty. The sequence matters:

1. **Order confirmed** (immediate) — "Thank you, we've received your order"
2. **Production update** (if applicable) — "Your product is being made"
3. **Shipped** (when tracking available) — "Your order is on its way" + tracking link
4. **Delivered** (carrier confirms) — "Your order has been delivered. How did it go?"
5. **Review request** (3-7 days post-delivery) — "Share your experience"
6. **Re-engagement** (30-60 days post-delivery) — "We have something new for you"

At seed stage, this is manual WhatsApp messages. At growth, it's automated sequences triggered by order status changes. At scale, it's personalized based on customer segment and purchase history.

### 4. Retention Campaigns (Growth+ Only)

Segments drive campaigns:
- **Win-back:** Customers with no order in 90+ days → special offer
- **Cross-sell:** Customers who bought product A → recommend product B
- **Referral:** High-satisfaction customers → referral code with incentive
- **Seasonal:** Birthday-based (if recipient DOB captured) → personalized offer

## Architecture Rationale

### Seed → Growth Migration

The trigger is usually one of:
- Order volume exceeds what one person can manually process (~50/month)
- Multiple channels are generating orders and data is fragmented
- You need to run campaigns and can't segment customers manually

**What changes:** SQLite → PostgreSQL. Single app → API server + background worker for async tasks (email, WhatsApp messages, shipping label generation). Add a simple queue (Redis-based) for reliable job processing.

### Growth → Scale Migration

The trigger:
- Multiple people processing orders simultaneously (need proper concurrency)
- Read-heavy analytics queries slowing down the transactional database
- Campaign volume requires bulk operations that can't run on the primary DB

**What changes:** Add read replicas for analytics queries. Move to event-driven architecture — order events trigger downstream systems (fulfillment, notifications, analytics) independently. Add CDN for media assets. Cache hot data (product catalog, customer profiles) in Redis.

## Constraints

- **Data retention:** Customer data should be retained as long as the business relationship exists. Reference media (photos for personalized products) should be auto-deleted after production + a configurable buffer (default: 90 days).
- **Payment compliance:** Never store raw card details. Use payment gateway tokens. At scale, PCI-DSS compliance required.
- **Communication consent:** WhatsApp Business API requires opt-in. Track consent status per customer per channel.
- **Multi-currency:** Not needed at seed. Important at growth if selling internationally. Critical at scale.

## Common Mistakes

1. **Treating buyer and recipient as the same person.** They're not. Build the data model right from day one — retrofitting is painful.
2. **Not capturing acquisition source.** At seed it seems unnecessary. At growth, when you're spending money on ads and events, you desperately need to know where customers come from. Capture UTM params and channel from the start.
3. **Manual fulfillment without a queue.** Even at seed stage, use a simple status-based queue (spreadsheet is fine). "Which orders need to ship today?" should be answerable in one click, not by scrolling through messages.
4. **Ignoring WhatsApp as a channel.** In India and emerging markets, WhatsApp IS the primary channel. Customers will message you to place orders, ask about status, request changes. If your CRM can't handle unstructured WhatsApp conversations, you'll have a parallel system in your phone's chat history.
