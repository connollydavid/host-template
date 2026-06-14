# References to the software worktree must survive its absence

- Status: accepted
- Date: 2026-06-15
- Refines: `call/0004` (the bare-store-with-worktrees embedding).

## Context and Problem Statement

`call/0004` embeds the software as a bare store with worktrees instead of a gitlink
submodule. A submodule gave the software **auto-presence**: `git checkout` always
left at least an empty directory, so symlinks into it resolved and tree-scans did
not choke. The bare-store model **removes that** — a worktree is genuinely absent
in any fresh clone or CI job until `host-lifecycle software --materialize` runs.

So every host artifact that reaches into `<software>/` — a skill symlink, an
mdBook `src` scan, a hook's binary path, a CI build step — is valid where the
software is materialized but **breaks where it is not**. `call/0004`'s path
continuity (references resolve to the same *path*) is not enough; it assumes the
path is *populated*. The missing guarantee is about **presence**, not path.

## Decision

Treat every host reference into a worktree as valid only where the worktree is
materialized, and harden three ways:

1. **Prevention — do not track the hazard.** Do not git-track an artifact that
   depends on the worktree existing (a skill symlink into `<software>/`). Gitignore
   it and recreate it after `software --materialize`, so a fresh clone has no
   dangling worktree reference at all.
2. **Detection — bounded and mechanical.** `host-lifecycle software --check` flags
   a `HAZARD` for any host-tracked symlink whose target resolves into a worktree
   path. Bounded to symlinks deliberately: a path-*string* in a script or config
   cannot be told from prose statically, so a wider scan would only reproduce
   `host-lint --all` noise. Those stay on the rule and on CI.
3. **Exercise the un-materialized context, and gate "done" on it.** A fresh
   clone / CI job runs *without* materializing the software — so an un-materialized
   job must exist for each runtime-critical artifact (the docs build is one), and
   "done" means the **whole** CI sweep is green, not one artifact built. Where a
   context genuinely needs the software, it runs `software --materialize` first.

## Consequences

- Good: the dangling-reference class is prevented (untracked) and caught
  mechanically (`--check` exit 1) before it ships.
- Good: the rule generalises to every migration, which all carry skill symlinks,
  hooks, and CI that assumed the old auto-presence.
- Neutral: a fresh clone must materialize the software (and recreate local skill
  symlinks) before the software's own skills/binaries are available — the honest
  cost of the software being absent until materialized.
- Limit: detection covers the symlink class only; path-string references (a hook
  binary path, a build `--manifest-path`) remain a rule-and-CI concern.
