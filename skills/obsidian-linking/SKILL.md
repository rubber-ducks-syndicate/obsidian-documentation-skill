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

## MOCs

A MOC (Map of Content) is a curated index note in `MOCs/`, one per major domain: `Features MOC.md`, `Architecture MOC.md`, `ADRs MOC.md`, `Backend MOC.md`, `Frontend MOC.md`, `Integrations MOC.md`, `Infrastructure MOC.md`, plus a root `Home MOC.md` linking to all MOCs.

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
