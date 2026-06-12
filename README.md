# Obsidian Documentation Skill System

A modular orchestrator + specialists ecosystem for Claude Code that turns code changes and conversations into structured, linked Obsidian documentation.

## Installation

Copy the contents of `skills/` into your project's `.claude/skills/` directory:

```bash
cp -r skills/* <your-repo>/.claude/skills/
```

Result:

```
.claude/skills/
├── obsidian-documentation/        # ← the ONLY entrypoint you use
│   ├── SKILL.md
│   └── references/
│       └── conventions.md         # shared: folders, frontmatter, naming, style
├── obsidian-feature/SKILL.md      # feature docs (backend + frontend)
├── obsidian-architecture/SKILL.md # system design, infra, integrations
├── obsidian-adr/SKILL.md          # decision records
├── obsidian-excalidraw/SKILL.md   # diagram generation (.excalidraw.md)
├── obsidian-tagging/SKILL.md      # tag taxonomy + enforcement
├── obsidian-linking/SKILL.md      # backlinks, MOCs, orphan prevention
└── obsidian-maintenance/SKILL.md  # dedupe, merge, refresh, restructure
```

## How it works

`obsidian-documentation` is the router and quality gate. It gathers context (conversation, git diff, repo structure, existing vault notes, commit history), classifies intent, delegates to specialists, and verifies the combined output for consistency. It never writes specialist content itself; if context is missing it asks instead of guessing.

Every run ends with a cross-cutting pass: **obsidian-tagging** then **obsidian-linking**, so every note lands consistently in the knowledge graph.

## Delegation table

| Intent | Primary skill | Usually also runs |
|---|---|---|
| Document a feature / PR / endpoint | obsidian-feature | excalidraw, tagging, linking |
| System design / infra / integration | obsidian-architecture | adr, excalidraw, tagging, linking |
| A decision was made | obsidian-adr | architecture, linking |
| Diagram request | obsidian-excalidraw | linking |
| Tag concerns | obsidian-tagging | — |
| MOCs / backlinks / orphans | obsidian-linking | — |
| Cleanup / dedupe / stale docs | obsidian-maintenance | tagging, linking |

## Example flows

**"Document the refunds feature from this PR"** → orchestrator reads the diff and searches the vault → obsidian-feature writes `Features/Refunds/Refunds.md` (+ Backend/Frontend sub-notes) → obsidian-excalidraw adds `Refunds Sequence.excalidraw.md` next to it → tagging assigns `feature`, `backend/api`, `frontend/react` → linking connects it to `Payment Processing`, `Stripe`, and `Features MOC`.

**"We chose Postgres over DynamoDB for orders"** → obsidian-adr writes `ADRs/ADR-0012 - Use Postgres for Orders Service.md` → obsidian-architecture updates the orders overview → linking wires ADR ↔ architecture both ways.

**"The vault is getting messy"** → obsidian-maintenance audits, proposes merges/deletions for approval, applies them, then tagging+linking repair the graph.

## Vault conventions (enforced everywhere)

Defined once in `obsidian-documentation/references/conventions.md`: folder layout (`Backend/ Frontend/ Features/ Architecture/ ADRs/ Diagrams/ Integrations/ Infrastructure/ MOCs/`), frontmatter template, Title Case naming, hierarchical lowercase tags (`#backend/graphql`), bidirectional `[[wiki links]]`, no orphans, update-before-create, plain-language why-first writing.

## Extending

Add a new specialist (e.g., `obsidian-runbook`) by creating its skill folder, pointing it at `references/conventions.md`, and adding one row to the orchestrator's routing table. Specialists never duplicate each other's responsibilities — diagrams belong to excalidraw, tags to tagging, links to linking, regardless of which skill is primary.
