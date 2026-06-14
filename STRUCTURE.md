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

- `tools/host-lint` (Unlicense) — hygiene / anti-slop; checks names against `host-grammar`.
- `tools/host-lifecycle` (Unlicense) — token-free scaffolder/validator; generates names from `host-grammar`, so what it emits is exactly what `host-lint` accepts.
- `tools/allium` (MIT, JUXT) — requirements + property-based testing.
- `tools/specula` (Apache-2.0) — timing and concurrency via TLA+.

(`host-grammar`, the shared naming/numbering rules crate, is a build dependency of `host-lint` and `host-lifecycle` — not a host submodule.)

`.claude/skills/` are symlinks into those submodules' skills — reference, not
copy. Tool *outputs* are project-owned (see `call/0001`).

To instantiate: clone, `git submodule update --init`, replace the `cast/`
examples with your own personas, and add your software as the hosted submodule.

The methodology lives in `CLAUDE.md` — read it first. The whole template is
released into the public domain (Unlicense); see `README.md` for provenance.
