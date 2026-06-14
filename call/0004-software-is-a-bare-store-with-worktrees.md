# Embed the software as a bare store with named worktrees

- Status: accepted
- Date: 2026-06-14
- Relates: `call/0003` (compose tools as referenced submodules — the software
  embedding is the counterpart question for the *Where* room).

## Context and Problem Statement

The *Where* room (the software under test) was a single git **submodule**: one
gitlink, one pinned SHA, one working tree. That gives audit/reproducibility but
only one live checkout, which serializes work. Two realistic needs break that:
**parallel agents** on one codebase (the prevailing idiom is a bare repo +
worktrees), and **multiple production release branches live at once** (a project
servicing several shipped versions cannot have its single tree flip between them).
The established "many live branches, one object store" idiom is a **bare clone +
worktrees**, which is not a gitlink — so adopting it means deciding what the host
pins and how audit survives.

## Decision

Embed each software component as a **bare object store plus named worktrees**:

- **Layout.** `<name>.git/` is the bare clone (shared object store, no working
  tree). `<name>/` is the **canonical worktree** — the audited state, where CI and
  the verification lanes run. Parallel lines are sibling worktrees `<name>.<line>/`,
  one per agent or release branch; worktrees are never nested inside one another.
- **Recipe, not trees.** The bare store and worktrees are local artifacts
  (gitignored). The host commits a recipe (`.host-software`) — one `[software
  "<name>"]` stanza per component (mirroring `.gitmodules`: `url`, the **pinned
  canonical SHA**, the worktree set). `host-lifecycle software --materialize`
  realises it; `host-lifecycle software --check` audits it.
- **Audit moves to the tooling.** With no gitlink, the recorded pin is the
  reproducibility anchor, and `host-lifecycle software --check` is the audit that
  replaces `git submodule status` (is `<name>/` at its recorded SHA?).
- **Migration (submodule → bare store).** Preserve the gitlink SHA as the pin (no
  software commit is created or moved); de-register the gitlink (`git rm --cached`,
  drop the `.gitmodules` stanza, remove `.git/modules/<name>`); write
  `.host-software`; gitignore the trees. The canonical worktree keeps the path
  `<name>/`, so references to `<name>/…` still resolve — only the embedding
  mechanism changed. "Bump the submodule pointer" becomes "update the recipe pin."

## Consequences

- Good: parallel agents and several live release branches coexist on one object
  store; the canonical worktree stays the single audited, CI-run state.
- Good: symmetric, idiomatic layout; migration is reference-preserving (the
  canonical worktree keeps the conventional path).
- Neutral: audit shifts from native `git submodule status` to `host-lifecycle
  software --check`; a fresh host clone no longer auto-fetches the software — the
  materialize step replaces `git submodule update --init`.
- Bad / accepted: worktrees share git objects but **not** environment — each
  carries its own build output and submodule checkouts. That cost is the price of
  holding several releases (or several agents' branches) materialized at once,
  which a single tree cannot do.
