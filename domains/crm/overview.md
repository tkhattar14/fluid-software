---
domain: crm
title: "CRM — Customer Relationship Management"
---

# CRM Domain

CRM is the first domain in Fluid Software because everyone understands what a CRM is, and the variant space is wide enough to prove the pattern.

A CRM for a VC firm tracking deal flow has almost nothing in common with a CRM for a D2C brand managing customer orders. Different entities, different workflows, different integrations, different architecture at every stage. Yet both are called "CRM."

## Variant Landscape

| Variant | Key Entity | Core Workflow | Scale Driver |
|---------|-----------|---------------|-------------|
| **D2C Brand** | Customer (buyer + recipient) | Purchase → fulfillment → retention | Order volume, channels |
| **VC Deal Flow** | Company (prospect) | Sourcing → diligence → close | Deals evaluated per year |
| **Recruiting** | Candidate | Apply → screen → interview → offer | Open positions, applicant volume |
| Real Estate | Property + Lead | Inquiry → showing → offer → close | Listings, lead sources |
| Nonprofit | Donor | Outreach → pledge → gift → steward | Donor count, campaign complexity |
| Agency/Consulting | Client + Project | Pitch → proposal → delivery → renewal | Active accounts, project complexity |

**Bold** variants have specs in this repo. Others are open for contribution.

## Cross-Domain Entities

CRM variants share entities with other domains:
- `customer` → also used in POS, accounting, shipping (see `domains/shared/customer.md`)
- `payment` → also used in POS, accounting, invoicing (see `domains/shared/payment.md`)

## What a CRM Spec Should Capture

1. **Entities:** What objects does this business track? (contacts, deals, orders, campaigns)
2. **Workflows:** What happens from first contact to ongoing relationship?
3. **Channels:** How do customers arrive? (web, WhatsApp, events, referrals, cold outreach)
4. **Integrations:** What external systems does this connect to?
5. **Stage-aware architecture:** How does the system change as the business scales?
