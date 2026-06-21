# Structure

An agentic project (e.g. `agentic-acme`) is the externalized *thought* about the
work; the software under development is the *action*, hosted beneath it. Its
rooms map to the five W's.

| W | room | holds |
|---|---|---|
| Who | `cast/` | personas (Powell, Keenan & McDaid 2007) — examples Mara + Wren |
| What | `<software>/` (with the code) | behavioural (`.allium`) + temporal (`.tla`) specs, verified in the software's CI; the host `plan/` references them |
| When | `plan/` | the milestone index and folders |
| Where | `<software>/` | the hosted software — a bare store with worktrees; you add it |
| Why | `call/` | decisions about the software (MADR; see `call/0000`) — methodology lives in the spine, not here |
| How | `CLAUDE.md` + `tools/` | the verification lanes |

`tools/` are referenced submodules, each under its own license — we orchestrate
and wrap, never patch:

- `tools/host-lint` (Unlicense) — hygiene / anti-slop; checks names against `host-grammar`.
- `tools/host-lifecycle` (Unlicense) — token-free scaffolder/validator; generates names from `host-grammar`, so what it emits is exactly what `host-lint` accepts.
- `tools/allium` (MIT, JUXT) — requirements + property-based testing.
- `tools/specula` (Apache-2.0) — timing and concurrency via TLA+.

(`host-grammar`, the shared naming/numbering rules crate, is a build dependency of `host-lint` and `host-lifecycle` — not a host submodule.)

A lane is **mandatory once a spec of its kind exists** (RFC-2119 MUST; see `CLAUDE.md`): a `.allium` requires `tools/allium` + its skills + a `check`/`analyse`/`plan` CI lane with the obligations discharged by tests; a `.tla` requires `tools/specula` + a TLC lane. Adopting a lane is optional; ignoring a present spec's lane is a defect.

`.claude/skills/` are symlinks into those submodules' skills — reference, not
copy. They are **generated, not tracked**: `link-skills.sh` creates a link for each
*materialized* tool (skipping uninitialized submodules), because a tracked symlink
into an uninitialized tool dangles and trips any tree-walking tool.
Run it after `git submodule update --init`. Tool *outputs* are
project-owned. host-lifecycle ships one skill **per lifecycle phase** (classify,
adopt, embed, remap, verify, publish, upgrade, release), enumerated once in the
tool-readable `lifecycle.manifest`; unlike a lane, the lifecycle is driven by the
tool, and the rule is **every phase emits a receipt** (`done`/`skip`/`n-a` in
`.host-receipts`), not "every phase runs" — a phase with no receipt is a
`software --check` HAZARD (`CLAUDE.md`).

The *Where* room is the software under test — **one or more** components, each
embedded as a **bare store with worktrees**: `<name>.git/` is the shared object
store, `<name>/` is the canonical worktree (the audited state, where CI runs), and
`<name>.<line>/` are parallel worktrees — one per agent or live release branch.
These trees are local and gitignored; the host commits a recipe (`.host-software`)
with **one `[software "<name>"]` stanza per component** (mirroring `.gitmodules`),
each recording the source URL, the pinned canonical SHA, and the worktree set,
which `host-lifecycle software --materialize` realises and `--check` audits.
The recorded pin replaces a submodule gitlink as the
reproducibility anchor, so several branches stay materialized at once where a
single submodule tree could not. Software initiated under the methodology has
**reproducible builds** — the stanza also records the `build`/`toolchain` recipe and
the deployed `artifact` hash, so `host-lifecycle software --verify-build` can rebuild
from the pin and prove the deployed binary; migrated software not yet reproducible
carries a `repro-exempt = call/NNNN` case decision (see `CLAUDE.md`). A component that
ships on several platforms records one `[build "<name>" "<platform>"]` subsection per
platform (each a distinct toolchain/artifact of the *same* pin, with an `attest-host`
naming the OS that reproduces it); the flat single-build form stays valid for the
single-platform case.

To instantiate: clone, `git submodule update --init` (the tools), run
`./link-skills.sh` (regenerate the skill symlinks for the tools you initialized),
replace the `cast/` examples with your own personas, and set up your software as a
bare store with worktrees (above). To bring an *existing* repo under the
methodology instead, follow the `host` repo (`github.com/connollydavid/host`).

To publish docs with mdBook, run **`host-lifecycle book .`** — the canonical
publisher, so you do not hand-roll a generator that drops a room or re-derives the
src-scoping wrong. It writes a `book.toml` scoped to a generated `docs/` (never
`src = "."`, which would walk the tool submodules and the un-materialized software
worktree and trip over whatever is not present) and a `SUMMARY.md` in
**lifecycle order**: Cast (Who) → Plan (When) → Software/Where (the What specs live
with the code, read as a stub from `.host-software`) → Call (Why) →
Reference/CLAUDE (How) → Memory. Then
`host-lifecycle book --check .` fails the build unless every room with source
renders a page, so a half-room site cannot ship. `book.toml` and `docs/` are
generated output (gitignored); the reference Site workflow under
`.github/workflows/` runs both before `mdbook build`.

A migrated or instantiated repo carries a `.host` stamp at its root
recording the template revision it adopted (`template`/`revision`/`adopted`),
written by `host-lifecycle adopt`. It is what a later upgrade diffs from. An
optional `name` line pins the published book's title, so `host-lifecycle book`
does not derive it from the checkout directory.

The methodology lives in `CLAUDE.md` — read it first. The whole template is
released into the public domain (Unlicense); see `README.md` for provenance.
