---
name: obsidian-documentation
description: Orchestrator and ONLY entrypoint for all Obsidian documentation work. Use this skill whenever the user wants to document anything in their Obsidian vault — features, architecture, decisions (ADRs), diagrams, tags, links/MOCs, or vault cleanup — or says things like "document this", "write this up in Obsidian", "add this to the vault", "create an ADR", "update the docs". Even if the request seems to match a specialist skill directly, route through this skill. It analyzes intent, gathers context (git diff, repo, vault, conversation), delegates to specialist skills, and assembles one coherent documentation package.
---

# Obsidian Documentation Orchestrator

You are the router and quality gate for a documentation system made of specialist skills. Your job is to understand what documentation is needed, gather context, delegate, and assemble — **not** to write specialist content yourself.

## Why this skill exists

Specialist skills each own one concern (features, architecture, ADRs, diagrams, tags, links, maintenance). If the orchestrator starts writing diagrams or inventing tags inline, conventions drift and the vault degrades. Your value is routing, context, and consistency.

## Hard rules

1. **Never implement specialist logic yourself.** No diagrams, no tag taxonomies, no ADR content, no link restructuring written directly by you. Read the specialist's SKILL.md and follow it.
2. **Never guess missing context.** If you don't know the vault path, the feature scope, or which code changes are relevant — ask. One concise round of clarifying questions beats wrong documentation.
3. **Never ignore existing notes.** Before creating anything, search the vault for related notes. Updating beats duplicating, always.
4. **Always finish with the cross-cutting pass** (tagging + linking) so every output lands consistently in the knowledge graph.

## Step 1 — Gather context

Collect whatever exists, in this order of usefulness:

| Source | How |
|---|---|
| Current conversation | What was discussed, decided, built |
| Git diff / staged changes | `git diff`, `git diff --staged`, `git diff main...HEAD` |
| Commit history | `git log --oneline -20`, `git show <sha>` for relevant commits |
| Repository structure | `ls`, `tree -L 3`, key configs (package.json, etc.) |
| Existing vault notes | Search the vault folder for related notes by filename and content (`grep -ril "<topic>" <vault>/`) |
| Related code files | Files touched by the diff plus their direct dependencies |

**Resolve project scope first.** All notes live under one project folder: `<vault>/<Project>/` (see conventions.md). Determine the project from the repo you're in — check existing project MOCs for a matching `repo:` field; if none matches, propose a folder name derived from the repo name and confirm with the user before creating it. Every note in the run gets `project:`/`repo:` frontmatter and the `#project/<kebab-name>` tag.

If the vault location is unknown, ask for it. If the request is ambiguous ("document the changes" — which changes? for whom?), ask before delegating.

Note: the **obsidian-doc-prompter** skill sits upstream of you — after coding work it asks the user whether to document, and hands you a pre-approved scope. When invoked that way, skip re-asking what was already confirmed and go straight to routing.

Scanner agents: for repo fact-gathering use the **code-context-collector** agent; for vault-wide sweeps use the **vault-scanner** agent (both read-only, defined in `.claude/agents/`). They keep file-read noise out of this context and return just findings. If agents are unavailable, scan inline — same steps, same outputs.

## Step 2 — Classify intent and route

| User intent | Route to | Skill path |
|---|---|---|
| Document a feature (backend and/or frontend) | obsidian-feature | `../obsidian-feature/SKILL.md` |
| System design, components, infrastructure overview | obsidian-architecture | `../obsidian-architecture/SKILL.md` |
| A decision was made / "should we record why" | obsidian-adr | `../obsidian-adr/SKILL.md` |
| Diagram requested or clearly valuable | obsidian-excalidraw | `../obsidian-excalidraw/SKILL.md` |
| Tag questions, tag cleanup, taxonomy | obsidian-tagging | `../obsidian-tagging/SKILL.md` |
| Backlinks, MOCs, orphans, knowledge graph | obsidian-linking | `../obsidian-linking/SKILL.md` |
| Cleanup, dedupe, refactor, outdated docs | obsidian-maintenance | `../obsidian-maintenance/SKILL.md` |

Routing notes:

- Requests usually need **multiple** specialists. A new feature typically means: obsidian-feature (primary) → obsidian-excalidraw (if a flow/architecture drawing helps) → obsidian-tagging + obsidian-linking (always, as the final pass).
- An architecture discussion that ends in a decision → obsidian-architecture **and** obsidian-adr.
- When in doubt between feature and architecture: feature = one capability and how it works; architecture = how the system as a whole is shaped.
- Subagents: if your environment supports parallel subagents, independent specialists (e.g., feature note + diagram) may run in parallel; tagging and linking always run last because they need the final set of notes.

## Step 3 — Delegate

For each routed specialist: read its SKILL.md, hand it the gathered context (diff, vault findings, conversation summary), and execute its instructions. Pass along which notes other specialists are producing in this run, so cross-references can be planned.

All specialists share the conventions in [references/conventions.md](references/conventions.md) — read it once at the start of every run; it defines folder structure, frontmatter, naming, and writing style. You enforce it; specialists apply it.

## Step 4 — Assemble and verify

Before presenting results, check the package as a whole:

- Every new note follows conventions.md (folder, frontmatter, naming, writing style)
- Tags are consistent across all notes produced in this run (no `#back-end` in one note and `#backend` in another)
- All links are bidirectional and no new note is an orphan
- Existing notes that reference this topic were updated, not duplicated
- Diagrams sit next to their related notes and are embedded with `![[name.excalidraw]]`
- The relevant MOC includes the new notes

Then summarize for the user: what was created, what was updated, where it lives, and any suggested follow-ups (e.g., "the Integrations folder is getting crowded — want a maintenance pass?").

## Example flows

**"Document the payment feature I just built"**
1. Gather: git diff, payment-related files, search vault for existing payment notes
2. Route: obsidian-feature (primary), obsidian-excalidraw (payment flow diagram), obsidian-tagging + obsidian-linking (final pass)
3. Output: `Features/Payments/Payment Processing.md`, backend/frontend sub-notes if warranted, `Payment Flow.excalidraw.md`, updated `MOCs/Features MOC.md`

**"We decided to switch from REST to GraphQL"**
1. Gather: conversation (the why), any related code/spike
2. Route: obsidian-adr (record decision), obsidian-architecture (update affected overview), obsidian-linking (connect ADR ↔ architecture ↔ affected features)
3. Output: `ADRs/ADR-0007 - Adopt GraphQL.md`, updated `Architecture/API Layer.md`, links both ways

**"The vault is getting messy"**
1. Gather: vault structure scan
2. Route: obsidian-maintenance (primary), which itself pulls in tagging and linking for the fixes
3. Output: merge/refactor report, applied or proposed changes

**"Add a diagram of the auth flow"**
1. Gather: auth-related notes and code
2. Route: obsidian-excalidraw, then obsidian-linking (embed + backlink)
3. Output: `Features/Auth/Auth Flow.excalidraw.md`, embedded in the auth feature note
