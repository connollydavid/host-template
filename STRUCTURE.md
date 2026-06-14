# Structure

The agentic host is the externalized *thought* about the work; the software
under development is the *action*, hosted beneath it. The host's rooms map to the
five W's.

| W | room | holds |
|---|---|---|
| Who | `cast/` | personas (Powell, Keenan & McDaid 2007) — examples Mara + Wren |
| What | `plan/NNNN-*/spec/` | behavioural (`.allium`) + temporal (`.tla/`) specs |
| When | `plan/` | the milestone index and folders |
| Where | `<software>/` | the hosted software — a bare store with worktrees; you add it |
| Why | `call/` | decisions (MADR; see `call/0000`) |
| How | `CLAUDE.md` + `tools/` | the verification lanes |

`tools/` are referenced submodules, each under its own license — we orchestrate
and wrap, never patch:

- `tools/host-lint` (Unlicense) — hygiene / anti-slop; checks names against `host-grammar`.
- `tools/host-lifecycle` (Unlicense) — token-free scaffolder/validator; generates names from `host-grammar`, so what it emits is exactly what `host-lint` accepts.
- `tools/allium` (MIT, JUXT) — requirements + property-based testing.
- `tools/specula` (Apache-2.0) — timing and concurrency via TLA+.

(`host-grammar`, the shared naming/numbering rules crate, is a build dependency of `host-lint` and `host-lifecycle` — not a host submodule.)

`.claude/skills/` are symlinks into those submodules' skills — reference, not
copy. Tool *outputs* are project-owned (see `call/0001`).

The *Where* room is the software under test, embedded as a **bare store with
worktrees**: `<software>.git/` is the shared object store, `<software>/` is the
canonical worktree (the audited state, where CI runs), and `<software>.<line>/`
are parallel worktrees — one per agent or live release branch. These trees are
local and gitignored; the host commits a recipe (`.host-software`) recording the
source URL, the pinned canonical SHA, and the worktree set, which a setup step
materializes. The recorded pin replaces a submodule gitlink as the
reproducibility anchor, so several branches stay materialized at once where a
single submodule tree could not.

To instantiate: clone, `git submodule update --init` (the tools), replace the
`cast/` examples with your own personas, and set up your software as a bare store
with worktrees (above). To bring an *existing* repo under the methodology
instead, follow `MIGRATION.md`.

A migrated or instantiated repo carries a `.host` stamp at its root
recording the template revision it adopted (`template`/`revision`/`adopted`),
written by `host-lifecycle adopt`. It is what a later upgrade diffs from.

The methodology lives in `CLAUDE.md` — read it first. The whole template is
released into the public domain (Unlicense); see `README.md` for provenance.
