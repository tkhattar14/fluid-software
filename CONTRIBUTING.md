# Contributing to Fluid Software

## The Big Idea

This repo is domain knowledge, not code. The most valuable contributions come from people who deeply understand how a business works, not people who know how to write software.

## How to Contribute

### If you're a domain expert (no technical skills required)

You know how your business actually operates. That knowledge is the most valuable thing in this repo.

**Option 1: Open an issue.** Describe your business type and what your software should do. We'll work with you to turn it into a spec.

**Option 2: Talk to an AI agent.** Open Claude, ChatGPT, or any AI assistant and say:

> "I run a [type of business]. I want to describe how my [CRM/POS/accounting/etc] should work so it can be turned into a Fluid Software spec. Ask me questions about my entities, workflows, integrations, and constraints. Structure your output using YAML frontmatter + markdown body."

Paste the output into a GitHub issue or PR. We'll review and refine it.

**Option 3: Just write it.** If you're comfortable with markdown, read [META-SCHEMA.md](META-SCHEMA.md) and write a spec file directly. It doesn't have to be perfect. Incomplete specs get improved over time.

### If you're a technical contributor

You understand architecture, system design, and infrastructure trade-offs.

- **Add stage-aware architecture recommendations** to existing specs. When should a business migrate from SQLite to Postgres? When do they need a message queue? When does a monolith need to split?
- **Add integration details** to existing specs. API patterns, webhook flows, data sync strategies.
- **Improve the meta-schema** if you see gaps in what it can express.
- **Write shared entity definitions** in `domains/shared/`.

### If you want to add a new domain

1. Create `domains/<domain>/overview.md` describing the domain and its variant landscape
2. Write at least one variant spec
3. Submit a PR

Good candidate domains: POS, accounting, inventory, helpdesk, project management, HR, booking/scheduling.

## Spec Quality

### What makes a good spec

- **Specific.** "The renewal check happens 90 days before expiry" beats "renewals should be tracked."
- **Opinionated.** "Use SQLite at seed stage" beats "choose an appropriate database."
- **From experience.** "We tried X and it didn't work because Y" is more valuable than "X is generally recommended."
- **Stage-aware.** What changes as the business grows? Different architecture, different workflows, different constraints.

### Status labels

- **`real`** — Written from direct experience. Authoritative.
- **`illustrative`** — Demonstrates the format. Domain experts: please upgrade these to `real`.
- **`draft`** — Incomplete. Work in progress.

## Review Process

1. A contributor submits a spec (via issue, PR, or conversation)
2. Maintainers check: does it follow the meta-schema? Is it specific enough?
3. Domain experts (if available) check: is this accurate?
4. Merge. Iterate. Specs improve over time.

## Code of Conduct

Be respectful. Domain expertise comes in many forms. A 20-year insurance broker who's never used Git has as much to contribute here as a senior engineer. Treat every contribution as valuable.
