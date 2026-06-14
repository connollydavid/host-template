# host-template

A template for starting a project as an **agentic host**: a thin governance shell
that holds the *thought* about a piece of software — its plans, decisions,
specifications, and the people it serves — while the software itself, the
*action*, lives in a submodule beneath it.

The point is separation. Thought lives in the host, where it is versioned,
audited, and readable without a checkout. Action lives in the submodule. An agent
working in the host reads `CLAUDE.md` and knows exactly how to proceed.

## What's in the box

Five rooms, one per question you ask about any work:

| Question | Room | Holds |
|----------|------|-------|
| Who  | `cast/` | personas — the people the software serves |
| What | `plan/<milestone>/spec/` | specs: behaviour (`.allium`), timing (`.tla`) |
| When | `plan/` | the milestone index and folders |
| Where | `<software>/` | the hosted software submodule — you add it |
| Why  | `call/` | decisions, in MADR format |
| How  | `CLAUDE.md` + `tools/` | the operating manual and the verification tools |

`CLAUDE.md` is the operating manual — read it first. `STRUCTURE.md` is the
one-page map.

### The verification tools (`tools/`)

Each tool is a referenced submodule, kept under its own license — we orchestrate
and wrap, never patch or vendor:

- `tools/host-lint` (Unlicense) — naming hygiene; a git hook that flags ordinal
  labels and bare numerals in commit messages, headers, and comments.
- `tools/host-lifecycle` (Unlicense) — a token-free generator that allocates
  numbers and scaffolds milestones, decisions, and personas. It shares its rules
  crate (`host-grammar`) with host-lint, so what it generates is exactly what the
  checker accepts.
- `tools/allium` (MIT, by JUXT) — requirements and property-based testing.
- `tools/specula` (Apache-2.0) — timing and concurrency, via TLA+.

`.claude/skills/` are symlinks into those submodules' skills — reference, not
copy.

## Instantiate

```bash
git clone --recurse-submodules https://github.com/connollydavid/host-template my-project
cd my-project
git submodule update --init --recursive   # if you skipped --recurse-submodules
```

Then make it yours:

1. Replace the example personas in `cast/` (`mara.md`, `wren.md`) with your own,
   following `cast/applying-personas.md`.
2. Add your software as the hosted submodule:
   `git submodule add <url> <software>/`.
3. Start your first milestone under `plan/`, and read `CLAUDE.md` before you
   write anything.

## Provenance

The four working principles in `CLAUDE.md` are rewritten, in our own words, from
[Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876)
on LLM coding pitfalls, by way of
[Jiayuan Zhang (@forrestchang)](https://github.com/forrestchang) and the
[andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
contributors, whose `CLAUDE.md` this repository began as a fork of. The persona
process follows Powell, Keenan and McDaid (2007) on personas in XP.

## License

Released into the public domain under the [Unlicense](LICENSE). The provenance
above is acknowledgement, not a license obligation.
