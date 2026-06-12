---
name: obsidian-feature
description: Specialist skill for documenting features (backend + frontend) in an Obsidian vault. Normally invoked by the obsidian-documentation orchestrator, but also use it directly when the user asks to document a feature, a new capability, an endpoint, a UI flow, or "what I just built" — including from a git diff or PR. Produces feature notes under Features/ with backend/frontend sub-notes, ready for the tagging and linking pass.
---

# Feature Documentation Specialist

You document **one capability at a time**: what it does, why it exists, and how its backend and frontend parts work together. Output goes to `Features/<Feature Name>/`.

Read `../obsidian-documentation/references/conventions.md` first — folder structure, frontmatter, naming, and writing style all come from there.

## When to use / not use

- **Use**: "document the payments feature", "write up what this PR adds", "document this endpoint and the screen that uses it"
- **Don't use**: whole-system design (→ obsidian-architecture), recording a decision (→ obsidian-adr), pure diagram requests (→ obsidian-excalidraw)

## Inputs you should work from

Git diff and touched files (what actually changed), the conversation (intent and business reason), existing vault notes about this feature or its neighbors, and related code (handlers, components, schemas). If you can't tell what the feature **does for the user/business**, ask — never reverse-engineer a guess about purpose from code alone.

## Process

1. **Check for an existing note.** Search `Features/` (and Backend/, Frontend/) for this feature. If found, update it instead of creating a new one.
2. **Determine scope.** Small feature → one note. Larger feature with distinct backend and frontend work → main note + `Backend.md` + `Frontend.md` sub-notes in the feature folder.
3. **Write the main note** using the template below.
4. **Flag diagram opportunities** to the orchestrator (e.g., a request flow worth drawing) — do NOT create diagrams yourself; that is obsidian-excalidraw's job.
5. **Propose tags and links** in frontmatter/Related sections, but final consistency is enforced by obsidian-tagging and obsidian-linking.

## Main feature note template

```markdown
---
created: <date>
updated: <date>
type: feature
status: active
tags: [feature, <domain tags>]
related: ["[[<related notes>]]"]
---

# <Feature Name>

<2–3 plain-language sentences: what this feature does and why it exists. A non-developer should understand this paragraph.>

## Why we built it

<Business/user problem it solves. The "why".>

## How it works

<Plain-language walkthrough of the flow, end to end. Mention key components
by name and link them: [[Backend/Payment Service]].>

## Backend

<Summary + link to [[Backend]] sub-note if one exists. Endpoints, data
model changes, jobs — explained simply.>

## Frontend

<Summary + link to [[Frontend]] sub-note if one exists. Screens,
components, state — explained simply.>

## Example

<A concrete scenario: "When a customer clicks Pay, …">

## Related

- [[Features MOC]]
- <other related notes>
```

Sub-notes (`Backend.md`, `Frontend.md`) follow the same shape, scoped down, and always link back to the main note.

## Example

**Input**: git diff adds `POST /refunds` endpoint, a `RefundButton` React component, and a `refunds` table.

**Output**:
- `Features/Refunds/Refunds.md` — main note: why refunds were needed, how a refund flows from button click to payout
- `Features/Refunds/Backend.md` — endpoint, table, refund job (tags: `feature`, `backend`, `backend/api`)
- `Features/Refunds/Frontend.md` — RefundButton, confirmation modal (tags: `feature`, `frontend/react`)
- Flag to orchestrator: "refund flow is a good sequence-diagram candidate"
