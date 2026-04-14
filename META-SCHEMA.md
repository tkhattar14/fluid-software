# Meta-Schema: The Fluid Software Spec Format

Every domain spec in this repo follows this format. The format is designed to be human-readable (markdown), agent-consumable (structured YAML frontmatter), and Git-diffable (plain text).

## Spec File Structure

A spec file has two parts: YAML frontmatter (structured, machine-readable) and markdown body (detailed, human-readable).

### Frontmatter

```yaml
---
domain: crm                          # Software category
variant: d2c-brand                   # Business type
status: real | illustrative | draft   # See below
contributor: "Name or handle"        # Who contributed the domain knowledge
version: 1.0

stages:
  seed:
    description: "0-500 customers, 1-2 person team"
    architecture:
      database: sqlite
      deployment: single-server
      pattern: monolith
  growth:
    description: "500-10K customers, 3-10 person team"
    architecture:
      database: postgresql
      deployment: api-server + workers
      pattern: modular-monolith
  scale:
    description: "10K+ customers, dedicated ops"
    architecture:
      database: postgresql-read-replicas | distributed
      deployment: microservices
      pattern: event-driven

entities:
  - name: customer
    fields: [name, email, phone, channel, source]
    relationships: [orders, segments]
    shared: true                      # References domains/shared/customer.md

  - name: order
    fields: [products, amount, status, source, date]
    relationships: [customer, payment, fulfillment]

workflows:
  - name: post-purchase-followup
    trigger: order.status == completed
    stages: [seed, growth, scale]     # Which stages this workflow applies to

integrations:
  - name: payment-gateway
    examples: [razorpay, stripe]
    required: true
  - name: shipping
    examples: [shiprocket, shippo]
    required: false
    stages: [growth, scale]           # Only needed at these stages

constraints:
  - "Customer data retention: configurable per jurisdiction"
  - "Payment data: PCI-DSS compliant storage required at scale stage"
---
```

### Markdown Body

After the frontmatter, the markdown body provides detailed descriptions that the frontmatter can't capture:

- **Entity details:** Field descriptions, validation rules, edge cases
- **Workflow walkthroughs:** Step-by-step process descriptions with decision points
- **Architecture rationale:** Why SQLite at seed, why Postgres at growth, when to migrate
- **Constraints explained:** Regulatory details, business rules, compliance requirements
- **Integration notes:** API patterns, webhook flows, data sync strategies
- **Real-world context:** How this actually works in practice, common mistakes, things that surprise you

The body is where domain expertise lives. The frontmatter is the index. Both matter.

## Field Reference

### `domain`
The software category. Examples: `crm`, `pos`, `accounting`, `inventory`, `helpdesk`.

### `variant`
The business type within a domain. Examples: `vc-deal-flow`, `d2c-brand`, `recruiting-pipeline`. A variant captures how a specific type of business uses this category of software.

### `status`
- **`real`** — Written from direct experience by someone who operates this workflow. Authoritative.
- **`illustrative`** — Demonstrates the format with reasonable domain knowledge. Not authoritative. Contributions from domain experts welcome to upgrade to `real`.
- **`draft`** — Work in progress. Incomplete.

### `stages`
Architecture tiers keyed by business maturity. Every spec should define at least `seed` and `growth`. Each stage includes:
- `description` — What this stage looks like (team size, customer count, revenue)
- `architecture.database` — Storage recommendation
- `architecture.deployment` — Infrastructure pattern
- `architecture.pattern` — Software architecture pattern

### `entities`
The data objects in this domain variant. Each entity has:
- `name` — Identifier
- `fields` — Key attributes (not exhaustive, but the ones that matter)
- `relationships` — Links to other entities
- `shared` — If `true`, references a cross-domain entity in `domains/shared/`

### `workflows`
The processes that define how this business operates. Each workflow has:
- `name` — Identifier
- `trigger` — What starts this workflow
- `stages` — Which business stages this workflow applies to (some workflows only exist at scale)

Workflow details go in the markdown body.

### `integrations`
External systems this variant connects to. Each integration has:
- `name` — Category (e.g., `payment-gateway`, not `stripe`)
- `examples` — Specific tools (e.g., `[razorpay, stripe]`)
- `required` — Whether the variant needs this to function
- `stages` — If only relevant at certain stages

### `constraints`
Business rules, regulatory requirements, compliance needs. Brief in frontmatter, detailed in markdown body.

## Cross-Domain Entities

Entities that appear across multiple domains live in `domains/shared/`. For example, `customer` appears in CRM, POS, accounting, and shipping. The shared entity file defines the common fields. Domain-specific extensions are defined in each variant spec.

When a variant spec sets `shared: true` on an entity, generation agents should read the corresponding `domains/shared/<entity>.md` file for the base definition.

## Design Principles

1. **Human-readable first.** A business person should be able to read a spec and say "yes, that's how we work" or "no, we do it differently."
2. **Agent-consumable second.** The YAML frontmatter gives agents structured data to work with. The markdown body gives context.
3. **Opinionated about architecture.** Stage-aware recommendations are the point. "It depends" is not useful.
4. **Incomplete is fine.** A spec that covers 70% of a domain is more valuable than no spec. Contributors improve it over time.
