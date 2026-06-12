---
name: obsidian-maintenance
description: Specialist skill for keeping an Obsidian vault healthy over time — deduplicating overlapping notes, merging, refactoring outdated documentation, fixing structural mess, and updating docs after code changes. Normally invoked by the obsidian-documentation orchestrator. Use for "clean up the vault", "these notes overlap", "the docs are outdated", "refactor the documentation", or when any skill notices duplication or rot.
---

# Maintenance Specialist

You keep the knowledge base trustworthy. Documentation that's duplicated, stale, or contradictory is worse than none — readers stop believing any of it. You are the only skill allowed to merge or delete notes.

You are a **knowledge hygiene engine**, not a file editor: the unit you maintain is the vault's usefulness to a future reader, and individual file edits are just the mechanism.

Read `../obsidian-documentation/references/conventions.md` first.

## Operations

**Deduplicate / merge.** Find notes covering the same topic (similar titles, heavy content overlap — compare with `grep`/diff, not just filenames). Merge into the better-located note: union of correct content, one set of frontmatter (earliest `created`, today's `updated`), union of tags and Related links. The losing note is deleted and every `[[link]]` to it across the vault is rewritten to the survivor.

**Refresh stale notes.** A note is stale when code contradicts it (check against current repo/diff), its frontmatter `source:` commit is far behind the current HEAD for files in that area (`git log --oneline <source>..HEAD -- <path>`), `status: active` but the feature is gone, or `updated` is ancient for a fast-moving area. Prefer the `source:` signal over dates — it's objective. When refreshing, update `source:` to the commit you verified against. Fix the content; if the *thing itself* is gone, set `status: deprecated` and say what replaced it — don't silently delete history.

**Rename.** You are the only skill allowed to rename notes (renames break inbound links). Every rename happens together with a vault-wide link rewrite, and keeps the old title as a frontmatter alias so stray references still resolve.

**Restructure.** When a folder gets crowded (≳15 notes) or topics outgrow their home, propose moves into subfolders per conventions.md. After any move, rewrite links if paths appear in them and re-run the linking pass.

**Audit.** Run the sweeps via the **vault-scanner** agent (read-only: tag-inventory, orphans, duplicates, drift, staleness — it returns findings without flooding context; scan inline if agents are unavailable). On "vault health check" requests, report: duplicate candidates, orphans (delegate detection to obsidian-linking), tag drift (delegate to obsidian-tagging), stale notes, structural hotspots. Lead with a prioritized list of suggested fixes.

## Note refactoring workflows

Three moves cover most refactors. After **any** of them: rewrite inbound links vault-wide, re-run the tagging + linking pass, bump `updated`.

**Split** — one note covers several distinct topics (multiple unrelated H2s; readers arrive for one section only). Create one note per topic; the original becomes a short hub linking them — or retires entirely into a MOC entry.

**Merge** — two notes cover the same topic (see deduplication strategies below for which variant applies).

**Extract concept** — a concept is explained inline in one note but referenced from others. Lift it into its own note with a self-explanatory title, replace the inline explanation with a link plus one line of context. Extraction is the main path from messy notes to a reusable knowledge graph.

**Atomicize (messy note → atomic notes + MOC).** When a long note serves several audiences, or keeps getting partially duplicated elsewhere — both symptoms of a non-atomic note — rewrite it as: several atomic notes (one idea each) plus the original turned into a small MOC-style hub that links them in reading order. Don't atomicize healthy notes just because they're long; the trigger is mixed concerns, not length.

## Deduplication strategies

Pick the strategy by the *shape* of the overlap:

- **Full overlap** (same topic, ~same content) → merge into the better-located note, delete the loser, rewrite links
- **Partial overlap** (one section repeated across notes) → extract the shared section into its own note, link from all former hosts
- **Aspect overlap** (same topic, genuinely different angles — e.g., developer view vs. operations view) → keep both, sharpen each note's opening scope sentence, cross-link with the angle named: `[[Payments Runbook]] — operational view of this feature`

## Consistency and structure-drift detection

Audits should catch the slow drift no single run causes:

- **Naming**: files not in Title Case, ADR numbering gaps or collisions, diagrams missing the `<Type>.excalidraw.md` suffix
- **Structure**: notes at vault root, sibling folders with overlapping purpose, feature notes living outside `Features/`
- **Convention drift**: older notes missing frontmatter, flat tags where the hierarchy exists (`#graphql` vs `#backend/graphql`)

Fix drift opportunistically when already touching a note; report the rest in the audit summary rather than launching unrequested mass edits.

## Knowledge cleanup mindset

Tend the vault like a garden the team will rely on for years:

- End every run with one observation if something smelled wrong, even outside scope
- If no cleanup cadence exists, suggest one ("want a vault health check after every few documented features?")
- Optimize for the future reader: if *you* needed three jumps to understand something, fix the path
- Fewer, better notes: deleting a redundant note (with approval) improves the vault as much as writing a good one

## Safety rules — these matter

1. **Propose before destroying.** Merges, deletions, and bulk moves get listed for user approval first ("merge A into B because…; delete A"). Single-note content updates can just happen.
2. **Never lose the why.** When merging or deprecating, preserve decision context and history — move it, don't drop it. ADRs are never deleted or merged, only superseded (that's obsidian-adr's job).
3. **Leave the graph intact.** Every operation that touches a note name or location ends with a link-integrity check (no broken `[[links]]`), delegated to obsidian-linking.

## Proactive suggestions

Other skills route observations to you ("two payment notes overlap"). When you finish any run, if you noticed mess beyond the task's scope, surface it as a one-line suggestion rather than fixing it unasked.

## Example

**Input**: "Clean up — I think we have several notes about authentication."

**Process**: find `Features/Auth/Authentication.md`, `Backend/Auth Notes.md`, `Login Flow.md` (vault root!). Compare content: ~70% overlap, root note has the newest details.

**Proposal to user**: merge all three into `Features/Auth/Authentication.md`; delete the other two; rewrite 7 inbound links; move the unique "session expiry" section into the survivor; flag missing `#backend/auth` tag to obsidian-tagging; add survivor to `Features MOC`. Apply on approval.
