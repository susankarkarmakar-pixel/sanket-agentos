# Sanket AgentOS

> An open-source, self-healing AI automation agent for Indian e-governance portals.
> Part of the **Sanket** suite (Sanket Notes · Sanket PDF Studio · Sanket Document Scanner).

---

## The problem

Across India's block, district, and state e-governance portals (Samarth, Seva Setu,
Social Registry, PM-JAY, SIR, recruitment/admit-card systems, and hundreds more),
field officers repeat the same manual chores every day: search a beneficiary,
cross-match a CSV, tick a declaration, upload a PDF, mark a status.

The usual fix is a one-off Chrome extension hardcoded to one portal's HTML. It works
until the portal's UI changes — then every selector breaks and someone has to rewrite
the extension from scratch. There is no shared tooling between districts, even though
every district office is solving the exact same class of problem.

## The idea

**Sanket AgentOS** is a local-first, open-source automation agent that understands a
web page the way a person does — by looking at it — instead of relying on brittle,
hardcoded CSS/XPath selectors.

- **Plain-language recipes.** An officer describes a task once
  ("upload this CSV of EPIC numbers, search each one, mark matched rows as
  verified") and the agent executes it.
- **Vision + DOM understanding.** A vision-language model reads the page
  (screenshot + accessibility tree) to find fields, buttons, and tables — no
  selector-hunting required.
- **Self-healing.** When a portal's UI changes, the agent re-locates the same
  logical elements instead of failing outright.
- **Recipe marketplace.** A recipe built for one block's portal can be shared,
  downloaded, and reused by any other district running the same system —
  turning dozens of one-off extensions into a single, growing library.

This generalizes a pattern already proven in practice across the Sanket extension
family (voter marking, pension re-entry, PM-JAY verification, Kanyashree PVF, etc.) —
Sanket AgentOS is the platform those extensions converge into.

## Monorepo layout

```
sanket-agentos/
├── packages/
│   ├── agent-core/        # Page understanding: screenshot + DOM → structured actions
│   ├── recipe-runtime/     # Parses & executes recipes against agent-core
│   └── extension-host/     # Chrome extension shell that embeds the runtime
├── recipes/                # Shared, reusable automation recipes (YAML)
├── examples/                # Worked example recipes for real portal flows
└── docs/
    └── ARCHITECTURE.md
```

## How a recipe works

A recipe is a small YAML file describing *intent*, not selectors. See
[`recipes/example-seva-setu-marking.yaml`](recipes/example-seva-setu-marking.yaml)
for a full example modeled on a real Samarth-portal marking workflow.

```yaml
name: mark-seva-setu-pending
description: >
  Upload a CSV of Aadhaar numbers, search each on the beneficiary portal,
  and mark matching rows as "Seva Setu".
steps:
  - input: csv
    expect_columns: [aadhaar_no]
  - for_each: row
    do:
      - find: "search field for Aadhaar number"
        action: type
        value: "{{row.aadhaar_no}}"
      - find: "search button"
        action: click
      - find: "Seva Setu checkbox in results row"
        action: click
      - find: "save/submit button"
        action: click
```

No CSS selectors, no XPath — `agent-core` resolves `find:` descriptions against
the live page at run time.

## Roadmap

- [ ] **Phase 1 — Agent core:** screenshot + accessibility-tree capture, VLM
      grounding (BYOK: OpenAI / Anthropic / local model), element resolution API
- [ ] **Phase 2 — Recipe runtime:** YAML recipe parser, step executor, CSV/Excel
      input binding, run logs
- [ ] **Phase 3 — Extension host:** Chrome extension UI wrapping the runtime,
      recipe picker, run history
- [ ] **Phase 4 — Recipe marketplace:** public recipe registry (GitHub-based
      initially — a `recipes/` PR *is* a publish), versioning, portal-change
      alerts
- [ ] **Phase 5 — Self-healing:** automatic re-grounding when a recipe step
      fails to resolve, with a diff report instead of a silent break

## Tech stack (proposed)

- **TypeScript** monorepo, **pnpm workspaces**
- **Playwright / Chrome DevTools Protocol** for page capture and control
- **BYOK LLM/VLM calls** (same pattern as Susan AI) — no vendor lock-in
- **YAML** recipe format, versioned in this repo

## Contributing

This repo starts as a scaffold — architecture and interfaces first, working
code next. Issues and recipe contributions are welcome once Phase 1 lands.

## License

MIT — see [`LICENSE`](LICENSE).
