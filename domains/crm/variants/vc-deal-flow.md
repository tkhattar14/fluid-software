---
domain: crm
variant: vc-deal-flow
status: illustrative
contributor: "Fluid Software"
version: 0.1

stages:
  seed:
    description: "Solo GP or emerging manager, <100 deals evaluated/year"
    architecture:
      database: sqlite
      deployment: single-server
      pattern: monolith
  growth:
    description: "Small fund, 2-5 partners, 100-500 deals evaluated/year"
    architecture:
      database: postgresql
      deployment: api-server + workers
      pattern: modular-monolith
  scale:
    description: "Established fund, 5+ partners, 500+ deals/year, multiple funds"
    architecture:
      database: postgresql
      deployment: api + workers + integrations-layer
      pattern: modular-monolith

entities:
  - name: company
    fields: [name, sector, stage, location, website, founding_date, description]
    relationships: [deals, contacts, notes, tags]

  - name: contact
    fields: [name, email, role, company, relationship_strength, last_contact]
    relationships: [company, introductions, meetings]
    shared: true

  - name: deal
    fields: [company, stage, lead_partner, source, amount, valuation, status]
    relationships: [company, contacts, notes, documents]

  - name: fund
    fields: [name, vintage, size, deployment_period, focus]
    relationships: [deals, lps]

  - name: lp
    fields: [name, type, commitment, contact]
    relationships: [fund]
    note: "Limited Partners — the investors in the fund itself"

workflows:
  - name: deal-pipeline
    trigger: company identified (inbound or sourced)
    stages: [seed, growth, scale]
  - name: portfolio-monitoring
    trigger: quarterly OR company.milestone
    stages: [seed, growth, scale]
  - name: fundraising
    trigger: new fund cycle
    stages: [growth, scale]

integrations:
  - name: deal-sourcing
    examples: [pitchbook, crunchbase, affinity]
    required: false
  - name: cap-table
    examples: [carta, pulley, angellist]
    required: false
    stages: [growth, scale]
  - name: email
    examples: [gmail-api, outlook-api]
    required: true
    note: "Email is the primary communication channel in VC. Auto-logging emails to deal records is critical."
  - name: calendar
    examples: [google-calendar, calendly]
    required: true

constraints:
  - "Deal data is highly confidential — access controls per partner/deal"
  - "LP reporting requirements drive data completeness needs"
  - "Relationship attribution matters — who introduced whom, who sourced the deal"
---

# VC Deal Flow CRM

> **Status: Illustrative.** This spec demonstrates the format using reasonable domain knowledge about venture capital workflows. It is not written from direct VC operating experience. If you're a VC partner, fund administrator, or investor relations professional, we'd value your contribution to upgrade this to a `real` spec.

## What Makes VC CRM Different

The primary entity is a **company**, not a person. You're tracking organizations through a pipeline: from first hearing about them, to meeting the founders, to doing diligence, to making an investment decision.

Relationships are everything. Who introduced you to the company. Who co-invested. Who you can call for a reference. The CRM is as much a relationship graph as it is a deal tracker.

VC also has unusually long time horizons. A company you pass on today might be relevant in two years. A founder you met at a conference might start something interesting next year. The CRM needs to be a long-term memory, not just an active pipeline.

## Deal Pipeline

```
SOURCED → FIRST MEETING → DEEP DIVE → PARTNER MEETING → TERM SHEET → CLOSED
                                                              ↓
                                                           PASSED
```

At each stage:
- **Sourced:** Company enters the system. Source tracked (inbound, referral, event, cold outreach).
- **First meeting:** Initial conversation with founders. Notes captured. Decision: pursue or pass.
- **Deep dive:** Extended diligence. Market analysis, reference calls, financial review.
- **Partner meeting:** Company presented to full partnership. Decision: term sheet or pass.
- **Term sheet:** Negotiation. Legal review.
- **Closed:** Investment made. Company moves to portfolio.
- **Passed:** Archived with notes on why. Searchable for future reference.

## Portfolio Monitoring (Post-Investment)

Once invested, the relationship shifts from evaluation to support:
- Quarterly updates from portfolio companies (revenue, burn, milestones)
- Board meeting tracking and preparation
- Follow-on investment decisions
- Founder support requests

## Common Mistakes

1. **Not logging relationship context.** Six months later, "how did we meet this founder?" is unanswerable if you didn't capture it.
2. **Treating passed deals as deleted.** They're not — they're deferred. The company you passed on at Series A might be right at Series B.
3. **No multi-fund support from the start.** Even a first-time fund manager will eventually raise a second fund. The data model should support it.
