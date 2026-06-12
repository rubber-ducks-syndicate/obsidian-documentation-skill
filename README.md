# Obsidian Documentation Skill System

A modular orchestrator + specialists ecosystem for Claude Code that turns code changes and conversations into structured, linked, project-scoped Obsidian documentation — and keeps it healthy over time.

## Installation

These are **global (user-level) skills** — install them once in your home `.claude/` directory and they work across every repository on your machine:

```bash
cp -r skills/* ~/.claude/skills/
cp -r agents/* ~/.claude/agents/
```

Global installation matters for this system: `config.md` (vault path + repo→project mappings) lives inside the orchestrator skill, and there must be exactly **one** copy of it. Installing per-repo would duplicate config.md into every repository and the copies would drift. Per-repo installation (`<repo>/.claude/`) is only sensible if a single repo needs a deliberately different setup — and then it shadows the global one for that repo.

Result:

```
~/.claude/agents/
├── vault-scanner.md               # read-only vault sweeps (tags, orphans,
│                                  #   duplicates, drift, staleness, concept gaps)
└── code-context-collector.md      # read-only repo fact sheets for grounded docs

~/.claude/skills/
├── obsidian-documentation/        # ← the ONLY entrypoint you invoke directly
│   ├── SKILL.md                   #   router, context gathering, quality gate
│   └── references/
│       ├── conventions.md         #   single source of truth for ALL skills
│       └── config.md              #   vault path + repo→project mappings
├── obsidian-feature/SKILL.md      # feature docs (backend + frontend)
├── obsidian-architecture/SKILL.md # system design, infra, integrations
├── obsidian-adr/SKILL.md          # decision records (numbered, immutable)
├── obsidian-excalidraw/SKILL.md   # diagram generation (.excalidraw.md)
├── obsidian-tagging/SKILL.md      # tag taxonomy + enforcement
├── obsidian-linking/SKILL.md      # backlinks, MOCs, semantic links, orphans
├── obsidian-maintenance/SKILL.md  # knowledge hygiene engine
├── obsidian-doc-prompter/SKILL.md # after coding work: offers to document
└── obsidian-query/SKILL.md        # answers questions FROM the vault (index-based)
```

One-time setup: open `obsidian-documentation/references/config.md` and set `vault_path`. That's it — no per-project CLAUDE.md entries needed. Repo→project mappings accumulate in the same file automatically as you document repos (skills append entries after confirming names with you). If `vault_path` is empty, skills ask once and write the answer back themselves.

Optional, for guaranteed documentation prompts after coding work (skill triggering otherwise depends on Claude noticing): add to any project's `CLAUDE.md`: *"After completing any meaningful code change, consult the obsidian-doc-prompter skill."*

## Core concepts

### Project scope — projects contain repositories

The vault is organized per **business project** (e.g., "Atlas"), and each project contains its **repositories** as subfolders. Repo-specific docs live under the repo folder; cross-repo content (system architecture, decisions) lives at project level.

```
<vault>/
├── Home MOC.md                    # links every project's MOC
├── .claude-docs/                  # hidden machine layer — Obsidian never shows it
│   └── <Project>/
│       ├── index.md               # auto-generated note catalog (retrieval layer)
│       └── log.md                 # append-only run history
└── Atlas/                      # business project
    ├── Atlas MOC.md            # project entry point
    ├── Architecture/              # cross-repo system design
    ├── ADRs/                      # project-level decisions (often span repos)
    ├── MOCs/                      # domain MOCs spanning all repos
    ├── Web App/                   # ← repository (e.g. acme/atlas-web)
    │   ├── Web App MOC.md
    │   ├── Backend/  Frontend/  Features/  Architecture/
    │   └── Diagrams/ Integrations/ Infrastructure/
    └── API/                       # ← repository (e.g. acme/atlas-api)
        └── …
```

Skills resolve project + repo folder automatically from the repository they run in, via the nested `projects:`/`repos:` mappings in `config.md` (asking once on first contact, then appending the mapping). Feature-specific diagrams live next to their feature note, not in `Diagrams/`. The human-facing vault contains only real notes — all machine bookkeeping is hidden in `.claude-docs/`.

### Repository awareness

Every note carries provenance back to the code:

```yaml
project: "Atlas"         # business project (top-level vault folder)
repo: "acme/atlas-web"   # repository this note documents
source: "a1b2c3d"           # commit/PR the note was verified against
tags: [feature, backend/api, project/atlas, repo/atlas-web]
```

The `#project/…` and `#repo/…` tags make a project's or repo's notes filterable by tag and graph view, not just folder. The `source:` commit gives maintenance an objective staleness signal: `git log <source>..HEAD -- <path>`.

### Grounding (anti-hallucination)

Defined once in `conventions.md`, binding for all skills: every factual claim must trace to the diff, code, conversation, or an existing note; exact names are copied from code, never reconstructed; unknowns become `> [!question] TODO: confirm` callouts instead of confident guesses; the business "why" is never reverse-engineered from code — it comes from the conversation or the user. Conflicts (code vs. conversation, code vs. existing note) are surfaced, not silently resolved.

### Safety guards

No secrets, credentials, or personal data in the vault (reference by location: "key lives in `STRIPE_SECRET`"). Idempotent re-runs — documenting the same topic twice converges on one updated note, never a duplicate. Runs touching more than ~20 existing notes pause for approval. Renames are maintenance-only operations (they break inbound links) and always include a vault-wide link rewrite.

## How it works

`obsidian-documentation` is the router and quality gate. It resolves project scope, gathers context (conversation, git diff, repo structure, existing vault notes, commit history), classifies intent, delegates to specialists, and verifies the combined output. It never writes specialist content itself; if context is missing it asks instead of guessing.

Every run ends with a cross-cutting pass — **obsidian-tagging** then **obsidian-linking** — so each note lands consistently in the knowledge graph.

### Delegation

| Intent | Primary skill | Usually also runs |
|---|---|---|
| Document a feature / PR / endpoint | obsidian-feature | excalidraw, tagging, linking |
| System design / infra / integration | obsidian-architecture | adr, excalidraw, tagging, linking |
| A decision was made | obsidian-adr | architecture, linking |
| Diagram request | obsidian-excalidraw | linking |
| Tag concerns | obsidian-tagging | — |
| MOCs / backlinks / orphans / concept gaps | obsidian-linking | — |
| Cleanup / dedupe / stale docs | obsidian-maintenance | tagging, linking |
| Just finished coding something | obsidian-doc-prompter | → orchestrator on yes |
| Question answered from the vault / find docs | obsidian-query | — |

`obsidian-doc-prompter` sits *upstream* of the orchestrator: after meaningful coding work it names the affected notes ("update Payment Processing.md, new Refunds note + diagram, possible ADR — all/some/skip?"), asks for consent, and hands the approved scope over. It stays quiet on trivial changes, prompts at most once per change, and never writes to the vault itself.

### Scanner agents

Two read-only subagents keep noisy file-sweeps out of the main context; all *writing* stays in the main conversation, where the business "why" lives. Skills fall back to inline scanning when agents are unavailable.

- **code-context-collector** — produces the grounding fact sheet before feature/architecture docs are written: exact endpoints, tables, components with file paths, HEAD sha for `source:`, env var names (never values), and open questions for the user. Quotes commit messages for intent but never invents it.
- **vault-scanner** — runs the audits for maintenance/tagging/linking: tag inventory, orphan detection, duplicate candidates (with overlap shape: full/partial/aspect), naming/structure drift, stale `source:` commits, and concept gaps. Returns findings with evidence, including explicit "clean" results.

## Specialist highlights

- **obsidian-feature** — one capability end to end; main note + Backend/Frontend sub-notes for larger features; flags diagram and ADR candidates instead of creating them.
- **obsidian-architecture** — current system shape at three levels (one System Overview per project, domain notes, integration notes). Includes "where does this belong" rules (architecture vs. feature vs. ADR), PARA / Johnny Decimal awareness (adapts to an existing vault philosophy, never imposes a second hierarchy), and long-term maintainability rules.
- **obsidian-adr** — numbered, immutable records with mandatory alternatives and consequences; changes supersede, never rewrite.
- **obsidian-excalidraw** — valid `.excalidraw.md` for the Excalidraw plugin, 5–12 elements per diagram, consistent palette, placed next to the owning note and embedded via `![[name.excalidraw]]`.
- **obsidian-tagging** — hierarchical lowercase taxonomy (`#backend/graphql`); reuse-before-invent; assign mode every run, audit mode for drift.
- **obsidian-linking** — structural links (note→MOC, ADR→affected) enforced; conceptual links suggested only with a written reason; missing-link/concept-gap detection; cluster → MOC suggestions; MOC chain `Home MOC → Project MOC → domain MOCs`.
- **obsidian-maintenance** — knowledge hygiene engine: split / merge / extract-concept / atomicize workflows, dedup strategies by overlap shape, drift detection, rename ownership, periodic cleanup mindset. Destructive changes always proposed before applied; ADRs are never deleted.
- **obsidian-query** — turns the vault into a queryable knowledge base (after Karpathy's [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern). Owns the hidden per-project `index.md` (one line per note: path, summary, tags, date — updated on every run, rebuildable via vault-scanner) and `log.md` (append-only, grep-parseable run history), both in `<vault>/.claude-docs/<Project>/`. Queries read the index first, drill into candidate notes plus one hop of links, and answer with `[[citations]]`; answers that involved real synthesis are offered back into the vault as new notes so explorations compound. Says "not in the vault" honestly instead of padding with general knowledge. At larger scale it suggests a local search tool (e.g. qmd) instead of pretending the index still suffices.

## Example flows

**"Document the refunds feature from this PR"** → orchestrator resolves the project folder from the repo → code-context-collector returns the fact sheet (endpoint, table, component names + HEAD sha) → obsidian-feature writes `<Project>/Features/Refunds/Refunds.md` (+ sub-notes) → obsidian-excalidraw adds `Refunds Sequence.excalidraw.md` next to it → tagging assigns `feature`, `backend/api`, `frontend/react`, `project/acme-webshop` → linking connects it to `Payment Processing`, `Stripe`, and `Features MOC`.

**"We chose Postgres over DynamoDB for orders"** → obsidian-adr writes `<Project>/ADRs/ADR-0012 - Use Postgres for Orders Service.md` → obsidian-architecture updates the orders overview → linking wires ADR ↔ architecture both ways.

**Claude just finished implementing rate limiting** → obsidian-doc-prompter: "This added rate limiting to the API. Document it? Would update `Architecture/API Layer.md`, add a feature note, possible ADR (token bucket over fixed window). All / some / skip?" → on yes, orchestrator runs the approved scope.

**"The vault is getting messy"** → obsidian-maintenance sends vault-scanner sweeping → proposes merges/renames/deletions for approval → applies them → tagging + linking repair the graph.

**"Why do refunds over $500 need manual review?"** → obsidian-query reads the project's hidden `index.md` → picks `Refunds` + `ADR-0013` → reads them → answers with citations: the fraud pattern from the pilot drove ADR-0013's choice of manual review ([[ADR-0013]], [[Refunds]]). If the answer required synthesizing several notes, it offers to file it back as a new note.

## Writing style (all generated docs)

Plain language for non-experts, the **why** before the how, business context before technical detail, 2–3 sentence plain-language summary at the top of every note, concrete examples, shorter-but-grounded over complete-looking-but-invented.

## Extending

Add a new specialist (e.g., `obsidian-runbook`) by creating its skill folder, pointing it at `references/conventions.md`, and adding one row to the orchestrator's routing table. Specialists never duplicate each other's responsibilities — diagrams belong to excalidraw, tags to tagging, links to linking, merges/renames to maintenance, regardless of which skill is primary.

## Official Obsidian references

Skills consult these instead of guessing syntax: [links](https://help.obsidian.md/links) · [properties](https://help.obsidian.md/properties) · [tags](https://help.obsidian.md/tags) · [callouts](https://help.obsidian.md/callouts) · [aliases](https://help.obsidian.md/aliases) · [Excalidraw plugin](https://github.com/zsviczian/obsidian-excalidraw-plugin)
