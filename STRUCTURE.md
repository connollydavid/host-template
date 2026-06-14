# Structure

The agentic host is the externalized *thought* about the work; the software
under development is the *action*, hosted beneath it. The host's rooms map to the
five W's.

| W | room | holds |
|---|---|---|
| Who | `cast/` | personas (Powell, Keenan & McDaid 2007) — examples Mara + Wren |
| What | `plan/NNNN-*/spec/` | behavioural (`.allium`) + temporal (`.tla/`) specs |
| When | `plan/` | the milestone index and folders |
| Where | `<software>/` | the hosted-software submodule — you add it |
| Why | `call/` | decisions (MADR; see `call/0000`) |
| How | `CLAUDE.md` + `tools/` | the verification lanes |

`tools/` are referenced submodules, each under its own license — we orchestrate
and wrap, never patch:

- `tools/no-phase-skill` (Unlicense) — hygiene / anti-slop.
- `tools/allium` (MIT, JUXT) — requirements + property-based testing.
- `tools/specula` (Apache-2.0) — timing and concurrency via TLA+.

`.claude/skills/` are symlinks into those submodules' skills — reference, not
copy. Tool *outputs* are project-owned (see `call/0001`).

To instantiate: clone, `git submodule update --init`, replace the `cast/`
examples with your own personas, and add your software as the hosted submodule.

Not yet wired: the `CLAUDE.md` methodology rewrite and the `LICENSE` flip to
Unlicense (coupled — the rewrite is what makes the content ours to dedicate),
and the token-free Rust lifecycle binary that scaffolds milestones and decisions
and validates the tree.
