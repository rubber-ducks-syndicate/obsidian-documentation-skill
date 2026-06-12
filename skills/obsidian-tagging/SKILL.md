---
name: obsidian-tagging
description: Specialist skill that owns the tag taxonomy of an Obsidian vault — assigning tags to new notes, keeping tags consistent, extending the hierarchy, and auditing/fixing tag drift. Normally invoked by the obsidian-documentation orchestrator as the final pass of every documentation run, and directly for "fix my tags", "what tags should this have", "tags are a mess" requests.
---

# Tagging Specialist

You own the tag taxonomy. Every documentation run ends with your pass, because tags created ad-hoc by different skills (or humans) drift, and drifted tags make search and graph views useless.

Read `../obsidian-documentation/references/conventions.md` first.

## Core taxonomy

```
#backend            #frontend            #feature        #architecture
  #backend/api        #frontend/react    #adr            #moc
  #backend/graphql    #frontend/state    #integration    #diagram
  #backend/db         #frontend/ui       #infrastructure
#aws                                       #aws/lambda  #aws/sqs  #aws/s3
```

Format rules: lowercase, singular, `/` for hierarchy, no spaces or camelCase. A note gets 2–5 tags: one *type* tag (`feature`, `architecture`, `adr`…) plus *domain* tags (`backend/graphql`, `aws/lambda`…).

## Reuse before invent

Before assigning any tag, list what already exists in the vault:

```bash
grep -rhoE '#[a-z][a-z0-9/_-]+' <vault> --include='*.md' | sort | uniq -c | sort -rn
```

(Also check frontmatter `tags:` lists, which don't carry `#`.) If an existing tag covers the concept, use it. Extend the hierarchy only when a real cluster of notes needs it — 3+ notes on GraphQL justifies `#backend/graphql`; one note doesn't. When you add a hierarchy level, the parent tag remains valid (`#backend/graphql` notes are still found via `#backend`).

## Two modes

**Assign mode** (every orchestrator run): take the run's new/updated notes, assign tags from the taxonomy, ensure all notes in the run use identical spelling, write tags into frontmatter.

**Audit mode** (tag cleanup requests, or when you spot drift): inventory all tags; find near-duplicates (`#front-end` vs `#frontend`), wrong format (`#Backend`, plurals), single-use orphan tags, and untagged notes. Propose a rename map to the user **before** mass-editing, then apply it across the vault.

## Example

**Assign**: new notes `Features/Refunds/Refunds.md`, `Backend.md`, `Frontend.md` →
`Refunds.md`: `[feature, backend, frontend]` · `Backend.md`: `[feature, backend/api]` · `Frontend.md`: `[feature, frontend/react]`

**Audit**: vault contains `#GraphQL` (3 notes), `#backend/graphql` (5 notes), `#payments` (1 note) → propose: merge `#GraphQL` → `#backend/graphql`; keep `#payments` only if more payment notes are expected, otherwise drop and rely on the `Features/Payments/` folder + links.
