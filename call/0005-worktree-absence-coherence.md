# References into a separately-materialized path must survive its absence

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

The same hazard applies to the **tool submodules**, not just the software
worktree: a tracked symlink into `tools/<tool>/skills/<name>` dangles whenever
that submodule is uninitialized (a fresh clone, CI, or a host that builds only the
tool it needs). The template carried exactly this bug — 17 tracked `.claude/skills/*`
symlinks, 16 dangling under partial init — while this decision claimed to
generalize. So the rule and its detector must cover **any separately-materialized
path**: a software worktree *or* a tool submodule.

## Decision

Treat every host reference into a worktree as valid only where the worktree is
materialized, and harden three ways:

1. **Prevention — do not track the hazard.** Do not git-track an artifact that
   depends on a separately-materialized path existing (a skill symlink into the
   software worktree *or* a tool submodule). Gitignore it and **generate** it after
   materialization: the template's `.claude/skills/*` are produced by `link-skills.sh`
   for the tools actually present (skipping uninitialized ones), and the software's
   own links are recreated after `software --materialize`. A fresh clone then has no
   dangling symlink at all. The template applies this to itself.
2. **Detection — mechanical, target-not-tracked.** `host-lifecycle software --check`
   flags a `HAZARD` for any tracked symlink whose **resolved target is not itself
   tracked in the repo** — it points into a separately-materialized path (a software
   worktree, or a sub-path of a tool submodule), so it dangles until that path is
   materialized. A symlink to a submodule *root* (a tracked gitlink) is not flagged:
   it resolves to the empty dir git leaves on checkout. Bounded to symlinks
   deliberately: a path-*string* in a script or config cannot be told from prose
   statically, so a wider scan would only reproduce `host-lint --all` noise.
3. **Exercise the un-materialized context, and gate "done" on it.** A fresh
   clone / CI job runs *without* materializing the software — so an un-materialized
   job must exist for each runtime-critical artifact (the docs build is one), and
   "done" means the **whole** CI sweep is green, not one artifact built. Where a
   context genuinely needs the software, it runs `software --materialize` first.

## Consequences

- Good: the dangling-reference class is prevented (untracked) and caught
  mechanically (`--check` exit 1) before it ships.
- Good: the rule and detector now genuinely cover **both** classes — software
  worktrees and tool submodules — and the template eats its own dogfood
  (`.claude/skills/*` untracked + generated, `link-skills.sh`).
- Neutral: a fresh clone must materialize the software (and recreate local skill
  symlinks) before the software's own skills/binaries are available — the honest
  cost of the software being absent until materialized.
- Limit: detection covers the symlink class only; path-string references (a hook
  binary path, a build `--manifest-path`) remain a rule-and-CI concern.
