# Fluid Software

**Open-source domain knowledge, not code.**

Software should adapt to your business, not the other way around. A CRM for a VC firm looks nothing like a CRM for a D2C brand. A CRM for 50 customers needs fundamentally different architecture than one for 100,000. Yet SaaS forces everyone onto the same tool.

Fluid Software is a shared, open-source repository of domain-specific specifications. Not code. Domain knowledge, structured so AI agents can generate personalized software from it.

## How It Works

1. **Domain experts describe their workflows** through conversation with AI agents
2. **Agents structure conversations into specs** using the [meta-schema](META-SCHEMA.md)
3. **Specs live in this repo** as markdown files, organized by domain and variant
4. **Any code generation tool** (Claude Code, Codex, Lovable, Cursor) can read a spec and generate software from it

The specs are the durable asset. The generation tools are the variable.

## Repo Structure

```
fluid-software/
├── META-SCHEMA.md               # The spec format (start here)
├── CONTRIBUTING.md               # How to contribute
├── domains/
│   └── crm/                      # First domain
│       ├── overview.md           # CRM variant landscape
│       └── variants/
│           ├── d2c-brand.md      # D2C e-commerce (real spec)
│           ├── vc-deal-flow.md   # Venture capital (illustrative)
│           └── recruiting.md     # Recruiting pipeline (illustrative)
└── domains/shared/               # Cross-domain entities
    ├── customer.md
    └── payment.md
```

## Using a Spec

Point your AI agent at a spec file and ask it to generate software:

```
Read fluid-software/domains/crm/variants/d2c-brand.md and generate
a working CRM application for a D2C brand at seed stage.
Use the architecture recommendations in the spec.
```

The spec gives the agent domain knowledge it wouldn't otherwise have: the actual workflows, entities, constraints, and architecture decisions that matter for that specific type of business.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The short version:

- **Domain experts:** Describe your workflows. You don't need to write markdown or know Git. Talk to an AI agent about how your business works, and it structures your knowledge into a spec.
- **Technical contributors:** Add architecture patterns, system design decisions, and stage-aware infrastructure recommendations to existing specs.
- **Everyone:** Open an issue describing a domain or variant that should exist. Someone will build the spec.

## Why This Exists

Three ideas converged:

- **Andrej Karpathy's "Idea Files"** — share concepts, not code. The idea is the primitive.
- **Peter Steinberger's "Prompt Requests"** — review intent, not artifacts.
- **Garry Tan's GBrain** — markdown at scale, managed by AI agents, works.
- **Spec-Driven Development** — specifications, not code, as the primary artifact.

All powerful. All limited to a single person or a single project. Nobody proposed the commons.

Fluid Software is the commons. Open-source domain knowledge that any agent can consume to generate personalized software for any business.

## Status

This is early. CRM is the first domain. The meta-schema will evolve. The specs will get better as domain experts contribute.

If you've run a business and know what your software should actually look like, we want your knowledge in this repo.

## License

[CC-BY-SA 4.0](LICENSE) — This is a knowledge commons, not a code project. Share and adapt with attribution.
