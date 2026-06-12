---
name: obsidian-linking
description: Specialist skill that owns the knowledge graph of an Obsidian vault — bidirectional wiki links, Maps of Content (MOCs), backlink maintenance, and orphan prevention. Normally invoked by the obsidian-documentation orchestrator as the final pass of every documentation run, and directly for "connect these notes", "build a MOC", "find orphan notes", "the graph view is a mess" requests.
---

# Linking Specialist

You own the connections. A vault's value is its graph: notes that can't be reached won't be read, and one-way links rot. Every documentation run ends with your pass.

Read `../obsidian-documentation/references/conventions.md` first.

## Rules

1. **Bidirectional always.** If note A links to B, B's `## Related` section must link back to A. After any run, verify both directions exist — Obsidian shows backlinks automatically, but explicit return links survive exports and make relationships intentional.
2. **No orphans.** Every note must be reachable from at least one MOC. New note → add it to the relevant MOC in the same run.
3. **Update neighbors.** When a new note mentions an existing topic, add the link in the new note **and** consider whether the existing note should mention the new one ("Refunds" should appear in "Payments"' Related section).
4. **Link meaningfully.** Link the first mention of a concept in a note; don't link every repetition. Use aliases for readable prose: `[[Payment Processing|payments]]`.

## Structural vs conceptual links

Treat the two kinds of links differently:

- **Structural links** mirror how the system and vault are organized: note → its MOC, feature → its architecture area, ADR → affected notes. Mandatory, checked every run (rules above).
- **Conceptual links** connect ideas that belong together in a reader's head even though no hierarchy joins them: `[[Refunds]] ↔ [[Fraud Detection]]`, because both reason about reversed payments. Suggested, never forced.

A vault needs both: structural links make it navigable, conceptual links make it intelligent.

## Suggesting meaningful links

A conceptual link needs a reason you can write down in one line: shared domain concept, same external dependency, one consumes the other's output, common failure mode. "Both mention Stripe once" is not a reason; "both implement Stripe webhook handling" is. Put the reason in the Related entry so future readers know why the link exists:

```markdown
- [[Fraud Detection]] — also consumes payment-reversal events
```

If you can't name the reason, don't suggest the link — noisy links bury the meaningful ones.

## Missing links (concept gaps)

Orphan detection finds unlinked *files*; also look for unlinked *concepts*:

- A term appearing in 3+ notes that has no note of its own → suggest a stub note (or a home in an existing note) and link the mentions to it
- Two notes sharing several distinctive terms that never link to each other → suggest the pair, naming the shared concept
- A MOC section whose theme clearly has matching notes scattered elsewhere that aren't listed

Present these as suggestions for the user to approve — concept gaps need human judgment about whether the concept deserves a note.

## Clustering → MOC suggestions

While linking, watch for clusters: 4+ notes that keep linking to each other, or share a tag/domain, but have no shared MOC section. Suggest either a new section in an existing MOC, or — at 5+ notes in a distinct domain — a new MOC (threshold below). Name the cluster after the concept, not the folder: "Payment lifecycle", not "misc backend notes".

## MOCs

A MOC (Map of Content) is a curated index note. The hierarchy is project-scoped: vault-root `Home MOC.md` → `<Project>/<Project> MOC.md` (the project's entry point; its frontmatter `repo:` field maps the repository to this folder) → domain MOCs in `<Project>/MOCs/`, one per major domain: `Features MOC.md`, `Architecture MOC.md`, `ADRs MOC.md`, `Backend MOC.md`, `Frontend MOC.md`, `Integrations MOC.md`, `Infrastructure MOC.md`.

When documenting a project for the first time, create the project MOC and add it to `Home MOC.md` in the same run — a project folder unreachable from Home is a project-sized orphan.

Create a MOC when a domain has 5+ notes, or when the orchestrator's run introduces a new domain. Don't create empty placeholder MOCs.

```markdown
---
created: <date>
updated: <date>
type: moc
tags: [moc]
---

# <Domain> MOC

<One sentence: what this map covers.>

## <Logical grouping>

- [[Note]] — one-line description
- …

## Related maps

- [[Home MOC]]
```

Group by theme, not alphabetically — a MOC is a guided tour, not a file listing.

## Orphan detection

On vaults beyond a handful of folders, delegate orphan + concept-gap detection to the **vault-scanner** agent (read-only; returns findings only). Inline fallback:

```bash
# Notes never referenced by any wiki link (candidate orphans):
for f in $(find <vault> -name '*.md' ! -path '*MOCs*'); do
  n=$(basename "$f" .md)
  grep -rql "\[\[$n" <vault> --include='*.md' --exclude="$(basename "$f")" >/dev/null || echo "$f"
done
```

For each orphan: add it to the right MOC and link it from at least one topically related note — or flag it to obsidian-maintenance if it looks obsolete.

## Process per run

1. Receive the run's new/updated notes from the orchestrator
2. Add forward links (first mentions), then return links in every linked note
3. Insert new notes into the relevant MOC(s); create a MOC if a domain crossed the threshold
4. Verify: no orphan among the run's notes, every link resolves to a real file (watch for typos in note names)

## Example

Run produced `Features/Refunds/Refunds.md` →
add `[[Payment Processing]]` and `[[Stripe]]` links inside Refunds; add `[[Refunds]]` to Related in `Payment Processing.md` and `Integrations/Stripe.md`; add `- [[Refunds]] — customer-initiated refunds via Stripe` to `Features MOC.md`.
