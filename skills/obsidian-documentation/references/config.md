# Configuration

The single place where this skill system finds your vault and maps repositories to business projects. Edit `vault_path` once; every skill reads this file. No per-project CLAUDE.md entries needed.

## Vault

```yaml
vault_path: ""          # e.g. /Users/me/Documents/MyVault  ← REQUIRED, fill this in
```

## Projects and their repositories

A **project** is a business initiative (one top-level vault folder); it usually contains **several repositories**, each getting its own subfolder. Skills match the current repo (by git remote name or directory name) to find the right project and repo folder.

```yaml
projects:
  # - project: "Atlas"               # business project = top-level vault folder
  #   tag: "project/atlas"           # project tag on every note
  #   repos:
  #     - repo: "acme/atlas-web"     # git remote (org/name) or directory name
  #       folder: "Web App"             # subfolder: <vault>/Atlas/Web App/
  #       tag: "repo/atlas-web"      # repo tag on every note from this repo
  #     - repo: "acme/atlas-api"
  #       folder: "API"
  #       tag: "repo/atlas-api"
```

New entries are appended automatically the first time a repo is documented: skills propose the project (existing one if the repo clearly belongs to it, otherwise a new name derived from the repo) and a repo folder name, confirm both with you, then write the entry here.

## Notes for skills reading this file

- Empty `vault_path` or path doesn't exist → ask the user, then **write the answer back here** so it's never asked again.
- Current repo not listed → propose project + folder, confirm with the user, append here. Never guess silently.
- This file is the source of truth for repo↔project mapping. MOC frontmatter may mirror it for convenience, but this file wins on conflict.
