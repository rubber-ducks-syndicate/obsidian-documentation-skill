---
name: obsidian-maintenance
description: Specialist skill for keeping an Obsidian vault healthy over time — deduplicating overlapping notes, merging, refactoring outdated documentation, fixing structural mess, and updating docs after code changes. Normally invoked by the obsidian-documentation orchestrator. Use for "clean up the vault", "these notes overlap", "the docs are outdated", "refactor the documentation", or when any skill notices duplication or rot.
---

# Maintenance Specialist

You keep the knowledge base trustworthy. Documentation that's duplicated, stale, or contradictory is worse than none — readers stop believing any of it. You are the only skill allowed to merge or delete notes.

Read `../obsidian-documentation/references/conventions.md` first.

## Operations

**Deduplicate / merge.** Find notes covering the same topic (similar titles, heavy content overlap — compare with `grep`/diff, not just filenames). Merge into the better-located note: union of correct content, one set of frontmatter (earliest `created`, today's `updated`), union of tags and Related links. The losing note is deleted and every `[[link]]` to it across the vault is rewritten to the survivor.

**Refresh stale notes.** A note is stale when code contradicts it (check against current repo/diff), `status: active` but the feature is gone, or `updated` is ancient for a fast-moving area. Fix the content; if the *thing itself* is gone, set `status: deprecated` and say what replaced it — don't silently delete history.

**Restructure.** When a folder gets crowded (≳15 notes) or topics outgrow their home, propose moves into subfolders per conventions.md. After any move, rewrite links if paths appear in them and re-run the linking pass.

**Audit.** On "vault health check" requests, report: duplicate candidates, orphans (delegate detection to obsidian-linking), tag drift (delegate to obsidian-tagging), stale notes, structural hotspots. Lead with a prioritized list of suggested fixes.

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
