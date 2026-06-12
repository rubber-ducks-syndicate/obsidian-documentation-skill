---
name: obsidian-architecture
description: Specialist skill for documenting system design in an Obsidian vault — component overviews, data flows, infrastructure layout, integration landscapes, and architecture notes. Normally invoked by the obsidian-documentation orchestrator. Use when the user discusses how the system is shaped, asks "document our architecture", describes services and how they connect, or after architecture discussions/refactors. Produces notes under Architecture/, Infrastructure/, and Integrations/.
---

# Architecture Documentation Specialist

You document **how the system is shaped**: components, boundaries, data flow, infrastructure, and external integrations. Output goes to `Architecture/` (and `Infrastructure/` / `Integrations/` for those domains).

Read `../obsidian-documentation/references/conventions.md` first.

## When to use / not use

- **Use**: "document our service architecture", "write up how data flows from the app to the warehouse", "document the AWS setup", post-refactor system overviews
- **Don't use**: one capability in detail (→ obsidian-feature), recording why a design was chosen (→ obsidian-adr — but you often run together), drawing itself (→ obsidian-excalidraw)

## Where does this content belong?

Quick test before writing a single line:

| Content | Home |
|---|---|
| How the system (or one area) is shaped **today** | Architecture note (this skill) |
| One capability and its end-to-end behavior | Feature note (→ obsidian-feature) |
| Why we chose X over Y, with alternatives | ADR (→ obsidian-adr) |

Rules of thumb:

- Architecture notes describe the **current state**; ADRs preserve the **fork in the road**. If you catch yourself writing "we considered…" inside an architecture note, stop — that paragraph is an ADR. The architecture note keeps a one-line summary of the outcome plus a link to the ADR.
- If a section only matters to one feature, it belongs in that feature's notes; the architecture note mentions the feature and links to it. Symmetrically, if a feature note starts explaining system-wide structure, that content moves here.

## Levels of architecture notes

Pick the level the request needs; don't write all three by default:

1. **System Overview** (`Architecture/System Overview.md`) — the whole system on one page; every vault should eventually have exactly one
2. **Domain/component notes** (`Architecture/API Layer.md`, `Infrastructure/AWS Setup.md`) — one bounded area each
3. **Integration notes** (`Integrations/Stripe.md`) — one external service each: what we use it for, how we talk to it, failure behavior

## Process

1. Search the vault for existing architecture notes — update before creating. The System Overview in particular should be **maintained**, never forked.
2. Establish facts from repo structure, configs (docker-compose, terraform, package.json), and the conversation. If the conversation describes a *target* architecture vs. *current* code, ask which one to document.
3. Write the note(s): plain language, why-first. A reader should understand what each component is **for** before how it works.
4. Flag diagram candidates to the orchestrator (component diagram for overviews, flow diagram for data paths) — obsidian-excalidraw draws them; you embed the result.
5. If the discussion included a *decision* (chose X over Y), tell the orchestrator an ADR is warranted.

## Vault design awareness

conventions.md defines a category-based layout, but users may already organize their vault differently. Before placing notes, check the vault's top level:

- **PARA** (Projects / Areas / Resources / Archive): place architecture docs under the relevant Area and apply the conventions.md structure *inside* it rather than fighting it.
- **Johnny Decimal** (numbered folders like `20-29 Architecture`): respect the existing numbering and slot notes into the matching range.

When such a system exists, adapt to it and report the mapping to the orchestrator. Never impose a second, competing hierarchy — two filing systems in one vault is worse than either alone.

## Scalability and long-term maintainability

Minimal rules that keep architecture docs useful for years:

- **One System Overview, always current.** Depth lives in domain notes; the overview stays around one page. If it grows past that, push detail down and link.
- **Explain once, link everywhere.** If two notes explain the same component, the explanation lives in the component's own note and the others link to it.
- **Phrase structurally, not numerically.** "Horizontally scaled workers" survives; "currently 3 instances" is stale next month. Where numbers matter, rely on the `updated` field to signal freshness.
- **Flag growth early.** When a domain note exceeds roughly a screenful of headings, propose a split and route it to obsidian-maintenance rather than letting it sprawl.

## Architecture note template

```markdown
---
created: <date>
updated: <date>
type: architecture
status: active
tags: [architecture, <domain tags>]
related: ["[[System Overview]]"]
---

# <Area Name>

<2–3 sentences: what this part of the system does and why it exists.>

## Components

<Each major component: name, one-line purpose, link to its note if one
exists. [[Backend/Payment Service]] — charges cards and records payments.>

## How data flows

<Plain-language walk through the main path(s). Embed diagram when
available: ![[<Area> Component Diagram.excalidraw]]>

## Key constraints and decisions

<Why it's shaped this way. Link ADRs: [[ADR-0007 - Adopt GraphQL]].>

## Related

- [[Architecture MOC]]
- <neighboring areas, features built on this>
```

## Example

**Input**: conversation about splitting a monolith — API stays, workers move to SQS-driven Lambdas.

**Output**:
- Update `Architecture/System Overview.md` (new shape)
- `Architecture/Async Workers.md` — why workers were split out, how jobs flow API → SQS → Lambda
- `Infrastructure/AWS Setup.md` updated with SQS/Lambda (tags: `infrastructure`, `aws/sqs`, `aws/lambda`)
- Flags: component diagram candidate; ADR candidate ("split workers from monolith")
