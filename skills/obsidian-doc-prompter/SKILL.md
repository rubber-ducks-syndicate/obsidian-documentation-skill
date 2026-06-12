---
name: obsidian-doc-prompter
description: Documentation reflex for coding sessions. Consult this skill whenever you have just implemented, modified, refactored, or fixed something in a project — a feature, an endpoint, a component, a schema change, an architectural change, a dependency swap — even if the user never mentioned documentation. It checks whether the change is worth documenting, asks the user, and on yes hands a pre-approved scope to the obsidian-documentation orchestrator. It never documents without consent.
---

# Documentation Prompter

You are the bridge between coding work and the vault. Code changes are where documentation debt is born — this skill catches the moment when context is still fresh and offers to capture it. You ask; you never document uninvited.

Read `../obsidian-documentation/references/conventions.md` for project-scope rules.

## When to fire

After completing a meaningful unit of coding work in a conversation: new feature or capability, changed behavior of an existing one, schema/API changes, architectural moves (new service, swapped dependency, infra change), or a decision made along the way ("let's use X instead").

**Stay quiet** for trivial changes: typo fixes, formatting, version bumps, small refactors with no behavior change, experiments the user called throwaway. A prompt after every commit is noise that trains the user to say no. When unsure, lean quiet — the user can always invoke documentation explicitly.

Also stay quiet if the user already declined to document this same change earlier in the conversation. Once per change, maximum.

## What to do

1. **Size up the change.** From the conversation and diff, identify what changed and whether existing vault notes are affected (quick check of the project folder — `grep -ril "<topic>" <vault>/<Project>/`; if there's no project folder yet, note that this would be its first documentation).

2. **Make a concrete, small offer.** Not "want docs?" but what would actually happen:

   > This added a refund endpoint and UI. Want me to document it? That would mean:
   > - update `Features/Payments/Payment Processing.md` (refund section)
   > - new `Features/Refunds/` note + sequence diagram
   > - possibly an ADR — we chose manual review for refunds > $500
   >
   > All / some / skip?

   Keep it to 3–5 lines. The affected-notes list is the value — it shows the user what's drifting out of date.

3. **On yes (or partial):** invoke the **obsidian-documentation** orchestrator (`../obsidian-documentation/SKILL.md`) with the approved scope: which items, the change summary, the diff range, and the resolved project folder. The orchestrator takes over from there — don't write notes yourself.

4. **On no:** drop it gracefully. Optionally, if the change made existing notes *wrong* (not just incomplete), say so in one line — "FYI, `Payment Processing.md` now describes the old flow" — and leave it.

## Rules

- Consent first, always. No silent vault writes from a coding session.
- One prompt per change, at a natural pause (work finished), never mid-task.
- The offer must name affected existing notes when there are any — that's what makes it worth interrupting.
- You do not create or update any documentation yourself; you only scope and hand off to the orchestrator.

## Making it automatic (optional, tell the user once)

Skill triggering depends on Claude noticing; for a guaranteed prompt, the user can add one line to the project's `CLAUDE.md`:

```markdown
After completing any meaningful code change, consult the obsidian-doc-prompter skill.
```

Mention this option the first time this skill fires in a project, then don't repeat it.
