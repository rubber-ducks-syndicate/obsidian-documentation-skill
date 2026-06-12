# Shared Vault Conventions

Single source of truth for every skill in the obsidian-documentation ecosystem. If a specialist skill contradicts this file, this file wins.

## Folder structure

```
<vault>/
├── Backend/           # Backend domain notes (services, APIs, data models)
├── Frontend/          # Frontend domain notes (apps, components, state)
├── Features/          # One subfolder per feature: Features/<Feature Name>/
├── Architecture/      # System design, component overviews, data flow
├── ADRs/              # Architecture Decision Records (flat, numbered)
├── Diagrams/          # Shared/cross-cutting diagrams only — feature-specific
│                      #   diagrams live NEXT TO their feature note
├── Integrations/      # Third-party services, external APIs
├── Infrastructure/    # AWS, CI/CD, deployment, networking
└── MOCs/              # Maps of Content (one per major domain)
```

Create subfolders automatically when context warrants it (e.g., `Features/Payments/`, `Backend/GraphQL/`). Never dump notes at vault root.

## Note naming

- Human-readable Title Case: `Payment Processing.md`, not `payment-processing.md`
- ADRs: `ADR-NNNN - Short Decision Title.md` (zero-padded, sequential)
- Diagrams: `<Topic> <Type>.excalidraw.md`, e.g. `Auth Flow.excalidraw.md`
- MOCs: `<Domain> MOC.md`, e.g. `Features MOC.md`

## Frontmatter template

Include frontmatter on every generated note:

```yaml
---
created: 2026-06-12
updated: 2026-06-12
type: feature | architecture | adr | diagram | moc | integration | infrastructure
status: draft | active | deprecated
tags: [feature, backend/graphql]
related: ["[[Other Note]]"]
---
```

Use real dates. Update `updated` whenever a note changes. Keep `tags` in frontmatter as the canonical list; inline tags in body text are optional reinforcement.

## Tags (summary — obsidian-tagging owns the full taxonomy)

Hierarchical, lowercase, singular: `#backend`, `#backend/graphql`, `#frontend/react`, `#feature`, `#architecture`, `#adr`, `#aws`, `#aws/lambda`, `#integration`, `#infrastructure`, `#moc`. Reuse existing tags before inventing new ones.

## Links (summary — obsidian-linking owns the rules)

- Wiki links only: `[[Note Name]]`, with aliases when needed: `[[Payment Processing|payments]]`
- Every link must be bidirectional — if A links to B, ensure B has a "Related" section linking back to A
- Every note must be reachable from at least one MOC. No orphans.
- Embed diagrams: `![[Auth Flow.excalidraw]]`

## Writing style

The audience includes non-experts. Documentation explains the **why**, not just the how.

- Simple language; expand or avoid jargon ("the queue retries failed jobs" beats "idempotent DLQ redrive")
- Lead every note with 2–3 sentences of plain-language summary: what this is and why it exists
- Business context before technical detail
- Concrete examples where they help understanding
- Short sections, descriptive headings
- Prefer prose; use tables/lists only where they genuinely clarify

## Update-first principle

Before creating any note, search the vault for existing coverage of the topic. If a related note exists: update or extend it. Only create a new note when the topic is genuinely new. Never produce two notes covering the same thing — that is a maintenance failure.
