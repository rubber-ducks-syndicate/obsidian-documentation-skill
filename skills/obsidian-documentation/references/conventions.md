# Shared Vault Conventions

Single source of truth for every skill in the obsidian-documentation ecosystem. If a specialist skill contradicts this file, this file wins.

## Vault location & repo mapping — [config.md](config.md)

All configuration lives in **`references/config.md` next to this file** — no per-project CLAUDE.md entries required. It holds `vault_path` and the repo→project mappings. Resolution order:

1. `config.md` (`vault_path` + matching `projects:` entry for the current repo)
2. The user stated it earlier in this conversation
3. Otherwise: **ask** — then write the answer back into `config.md` so it's never asked again

Never guess a vault path or write notes to an assumed location.

## Project scope — the root concept

The vault is organized **per business project**, and projects contain **repositories**. A project (e.g., "Atlas") is one top-level vault folder; each of its repos gets a subfolder inside it. Everything about the project lives under its folder:

```
<vault>/
├── Home MOC.md            # vault root: links every project's MOC
├── Atlas/              # business project
└── <Project B>/
```

- Project and repo-folder names are human-readable Title Case (`atlas-web` → repo folder `Web App` or `Atlas Web` — confirm with the user on first contact, then reuse).
- The repo ↔ project mapping is recorded in [config.md](config.md) (`projects:` list with nested `repos:`) — the source of truth every run resolves against. MOC frontmatter may mirror it, but config.md wins on conflict.
- Never write project-specific notes outside the project folder. Cross-project comparisons or company-wide notes are the only thing allowed beside project folders (and they link into the projects).

## Folder structure (inside each project folder)

Two levels: **project level** for everything that spans repositories, **repo level** for repo-specific docs.

```
<vault>/<Project>/                    # e.g. Atlas/
├── <Project> MOC.md       # project home: entry point, links repos + domain MOCs
├── Architecture/          # cross-repo system design, how the repos fit together
├── ADRs/                  # decision records (numbered per project — decisions
│                          #   often span repos)
├── MOCs/                  # project-wide Maps of Content (one per major domain)
└── <Repo>/                # one folder per repository, e.g. Web App/, API/
    ├── <Repo> MOC.md      # repo home: entry point for this repo's notes
    ├── Backend/           # services, APIs, data models
    ├── Frontend/          # apps, components, state
    ├── Features/          # one subfolder per feature: Features/<Feature Name>/
    ├── Architecture/      # repo-internal design (cross-repo design goes up a level)
    ├── Diagrams/          # shared/cross-cutting diagrams only — feature-specific
    │                      #   diagrams live NEXT TO their feature note
    ├── Integrations/      # third-party services this repo talks to
    └── Infrastructure/    # AWS, CI/CD, deployment for this repo
```

Placement rule: if a note concerns one repo, it lives under that repo's folder; if it spans repos (system architecture, project decisions, shared integrations), it lives at project level. A single-repo project still uses a repo subfolder — it keeps the structure uniform and painless to extend when repo #2 arrives.

Create subfolders automatically when context warrants it (e.g., `Features/Payments/`, `Backend/GraphQL/`). Never dump notes at the project or repo root (except their MOCs) or the vault root.

**Machine files live in a hidden folder.** The auto-generated index and log (owned by obsidian-query) live at `<vault>/.claude-docs/<Project>/index.md` and `log.md`. Obsidian hides dot-folders entirely, so the vault stays clean for humans while skills keep fast retrieval. Nothing else goes in `.claude-docs/`.

## Note naming

- Human-readable Title Case: `Payment Processing.md`, not `payment-processing.md`
- ADRs: `ADR-NNNN - Short Decision Title.md` (zero-padded, sequential)
- Diagrams: `<Topic> <Type>.excalidraw.md`, e.g. `Auth Flow.excalidraw.md`
- MOCs: `<Domain> MOC.md`, e.g. `Features MOC.md`
- Never use characters that break Obsidian filenames or wiki links: `/ \ : # ^ | [ ]` (and avoid `?` `*` `"` for cross-platform safety)
- Acronyms keep their casing: `AWS Setup.md`, `GraphQL Gateway.md` — not `Aws Setup.md`
- Add frontmatter `aliases:` for acronyms and common synonyms so links resolve naturally: a note `Continuous Integration.md` with `aliases: [CI]` lets writers link `[[CI]]`
- **Renaming an existing note is a maintenance operation.** Renames break inbound links, so only obsidian-maintenance may rename, and always with a vault-wide link rewrite in the same pass.

## Frontmatter template

Include frontmatter on every generated note:

```yaml
---
created: 2026-06-12
updated: 2026-06-12
type: feature | architecture | adr | diagram | moc | integration | infrastructure
status: draft | active | deprecated
tags: [feature, backend/graphql]
related: ["[[Other Note]]"]
aliases: []          # optional: acronyms/synonyms for natural linking
project: ""          # business project (top-level folder), e.g. "Atlas"
repo: ""             # repository this note documents, e.g. "acme/atlas-web"
source: ""           # for code-derived notes: commit sha, PR, or branch
---
```

Use real dates. Update `updated` whenever a note changes. Keep `tags` in frontmatter as the canonical list; inline tags in body text are optional reinforcement.

**Provenance**: when a note is derived from code, fill `source:` with the commit sha or PR (`source: "a1b2c3d"` / `source: "PR #142"`), and refresh it whenever the note is updated against newer code. This gives obsidian-maintenance an objective staleness signal — "documented at commit X, code is now at Y" — instead of guessing from dates.

## Tags (summary — obsidian-tagging owns the full taxonomy)

Hierarchical, lowercase, singular: `#backend`, `#backend/graphql`, `#frontend/react`, `#feature`, `#architecture`, `#adr`, `#aws`, `#aws/lambda`, `#integration`, `#infrastructure`, `#moc`. Reuse existing tags before inventing new ones.

**Project/repo tags**: every note carries `#project/<kebab-name>` (e.g., `#project/atlas`); repo-specific notes additionally carry `#repo/<kebab-name>` (e.g., `#repo/atlas-web`). Both come from config.md. This makes a project's or repo's notes findable by tag and graph filter even when viewed outside their folder.

## Links (summary — obsidian-linking owns the rules)

- Wiki links only: `[[Note Name]]`, with aliases when needed: `[[Payment Processing|payments]]`
- Every link must be bidirectional — if A links to B, ensure B has a "Related" section linking back to A
- Every note must be reachable from at least one MOC. No orphans. The chain is: vault `Home MOC` → `<Project> MOC` → `<Repo> MOC` / domain MOCs → notes.
- Note names may repeat across projects and repos (`Authentication.md` in two repos) — when ambiguous, link by path: `[[Atlas/Web App/Features/Auth/Authentication]]`
- Embed diagrams: `![[Auth Flow.excalidraw]]`

## Writing style

The audience includes non-experts. Documentation explains the **why**, not just the how.

- Simple language; expand or avoid jargon ("the queue retries failed jobs" beats "idempotent DLQ redrive")
- Lead every note with 2–3 sentences of plain-language summary: what this is and why it exists
- Business context before technical detail
- Concrete examples where they help understanding
- Short sections, descriptive headings
- Prefer prose; use tables/lists only where they genuinely clarify

## Update-first principle

Before creating any note, search the vault for existing coverage of the topic. If a related note exists: update or extend it. Only create a new note when the topic is genuinely new. Never produce two notes covering the same thing — that is a maintenance failure.

**Idempotency**: documenting the same feature/topic twice must converge on the same note, updated — never a second copy. Re-running a documentation request is always an update run.

## Grounding and accuracy (anti-hallucination)

Documentation that's plausible but wrong is the most dangerous output this system can produce. Every skill follows these rules:

- **Every factual claim must trace to a source**: the git diff, actual code, the conversation, or an existing vault note. Endpoint names, table names, component names, config values — copy them from code, never reconstruct them from memory of "how these things are usually named".
- **Mark unknowns instead of guessing.** When something can't be verified, write an explicit callout rather than a confident sentence:

  ```markdown
  > [!question] TODO: confirm
  > Retry count assumed to be 3 — not found in code or conversation.
  ```

- **Never fabricate the business "why".** Purpose and rationale come from the conversation or the user — if absent, ask. Reverse-engineering intent from code produces convincing fiction.
- **Conflicts**: code contradicts the conversation → ask the user which is true before writing. Code contradicts an existing note → flag it visibly in the note (callout) and report it; don't silently rewrite history.
- It is always better to ship a shorter, fully-grounded note with two TODO callouts than a complete-looking note with invented details.

## Safety guards

- **No secrets in the vault.** Never copy credentials, API keys, tokens, connection strings, or personal data from code/configs into notes. Reference them by location instead: "API key lives in `STRIPE_SECRET` (see infra config)".
- **Mass-edit cap**: any run about to modify more than ~20 existing notes pauses and presents the batch for approval first (creation of new notes doesn't count).

## Official documentation references

When unsure about Obsidian syntax or behavior, consult the official docs instead of guessing:

- Wiki links & embeds: https://help.obsidian.md/links
- Properties (frontmatter): https://help.obsidian.md/properties
- Tags: https://help.obsidian.md/tags
- Callouts (used for TODO/confirm markers): https://help.obsidian.md/callouts
- Aliases: https://help.obsidian.md/aliases
- Excalidraw plugin (file format, embedding): https://github.com/zsviczian/obsidian-excalidraw-plugin
