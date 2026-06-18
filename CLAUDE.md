# CLAUDE.md — operating manual for an agentic project

This file tells you, the agent, how to work in this repository. Follow it
exactly. The rules are written to be literal: when one says "do X", do X — do
not look for a cleverer path. When something is unclear or could mean two
things, stop and ask the human before you act. Clarity beats cleverness here.

## What this repository is

This repository is an **agentic project** (e.g. `agentic-acme`). It is the
externalized *thought* about a piece of software: the plans, the decisions, the
specifications, the people it serves, and the rules you work under. The software
itself — the *action* — lives beneath the project as a bare store with worktrees
(the *Where* room). You write thought in the project and action in
the worktree. Keep them separate.

You are working in a *template*. A real project replaces the example personas
with its own and adds its software as the hosted bare store with worktrees. The
structure below stays the same.

## The five rooms

The host has five rooms, one for each question you ask about any piece of work.
Put each kind of file in its room. Do not invent new top-level folders.

| Question | Room | What goes here |
|----------|------|----------------|
| Who  | `cast/` | personas — the people (human or agent) the software serves |
| What | `<software>/` (with the code) | specifications: behaviour (`.allium`), timing (`.tla`), verified in the software's own CI |
| When | `plan/` | the milestone index and one folder per milestone |
| Where | `<software>/` | the hosted software — a bare store with worktrees; you add it |
| Why  | `call/` | decisions, in MADR format (see `call/0000`) |
| How  | `CLAUDE.md` + `tools/` | this manual, and the verification tools |

`STRUCTURE.md` is the short map of the same thing. Read it once.

## How you work — four principles

These four principles govern every change. They exist because language models
make predictable mistakes; each principle blocks one.

### 1. Think before coding

- State your assumptions in plain text before you write code.
- If the request could mean more than one thing, list the meanings and ask which
  one. Do not pick one silently.
- If a simpler approach exists than the one you first reached for, say so.
- If any part of the request is unclear, stop and name the unclear thing. Ask.
  Do not guess.

The goal: the human never reads your output and says "that is not what I meant."

### 2. Keep it simple

- Write the least code that solves the stated problem. Nothing speculative.
- Do not add features that were not asked for.
- Do not add an abstraction (a base class, an interface, a wrapper) for
  something used in exactly one place. Write the concrete thing.
- Do not add configuration for a value that has exactly one setting. Hardcode it.
- Do not handle errors that cannot happen given the current inputs.
- If your code is long and the same result fits in a fraction of the lines,
  rewrite it short before you show it.

### 3. Make surgical changes

- Touch only the lines the request needs. Clean up only the mess you made.
- Do not "improve" nearby code, comments, names, or whitespace that the request
  did not ask about.
- Do not refactor working code that is not part of the request.
- Match the existing style exactly — tabs or spaces, snake_case or camelCase,
  whatever the file already uses.
- If your change leaves an import or variable unused, remove it in the same
  commit. If you spot unrelated dead code or a bug, mention it to the human; do
  not silently fix or delete it.

Check: every line you changed must trace to the request. If a line does not,
revert it.

### 4. Drive to a verifiable goal

Turn every task into a goal you can check, then loop until the check passes.

- "Add validation" becomes: write tests for the invalid inputs, then write code
  until those tests pass.
- "Fix the bug" becomes: write a test that reproduces the bug, then change code
  until that test passes.
- "Refactor X" becomes: confirm the tests pass, refactor, confirm they still pass.

For any task with more than one step, write a short numbered plan first, and give
each step a check:

```
<what you will do> -> verify by: <how you will confirm it worked>
```

If the success check is weak ("make it work"), ask the human to make it concrete
before you start.

## Names, numbers, and milestones

Numbers are identity. Slugs are content. Ordering lives in the index, never in
the name.

- A milestone is a folder in `plan/` named `NNNN-slug`: a four-digit
  zero-padded number, a hyphen, then a lowercase hyphenated slug — for example
  `0001-example-milestone`. Decisions in `call/` use the same `NNNN-slug` form.
- Name a milestone after its content, not its position. `0003-ci-pipeline` is
  good. Do **not** name things by ordinal position (`phase-one`, `M2`) or by a
  bare numeral (a header that is just "3" or "5.5"). Positions move when a plan
  is re-cut; content names stay attached to their content.
- The number is assigned when the work is accepted, and it never changes. To
  read sequence, read the index, not the filenames.
- `tools/host-lifecycle` allocates these numbers and checks the names for you.
  Use it (see below) instead of numbering by hand.

## Decisions — `call/`

When you make a choice that someone later will ask "why?", record it as a
decision in `call/`, in MADR (Markdown Any Decision Record) format. The bootstrap
decision `call/0000` explains the format and links to the MADR spec. One file per
decision, `NNNN-slug.md`, number assigned when the decision is accepted.

`call/` records decisions about **the software under development** — why your
software is built as it is. It does **not** hold methodology decisions. The
methodology is settled in this spine (`CLAUDE.md` + `STRUCTURE.md`), inherited by
copy-at-version; a change to it is made in the template and propagated by
`host-lifecycle upgrade`, never re-litigated as a project `call/`.

**Anti-ouroboros.** A project must not feed on its own methodological tail. If a
`call/` decision restates a methodology rule that is then settled or changed
upstream in the spine, retire it the MADR way: set `Status: superseded by the
spine` in place (records are immutable — you change status, never delete). The
live (`accepted`) Why room then holds only decisions still in force.
`host-lifecycle validate` fails an `accepted` decision that is missing a `Scope:`
header or declares `Scope: methodology`. The template ships `call/0000` only as a
worked example; replace it with your software's own decisions.

**Inherit from the source, not from a host.** You inherit the methodology from
*this template* (the versioned source you copy-at-version) alone. A host or
management repo's **top-level instance contents** — its `call/`, `plan/`,
`MEMORY.md`, the project-specific parts of its `CLAUDE.md` — are that project's own
rooms and bind no adopter. Do not read them as normative.

## Specs — with the software they constrain

A spec states what the software must do; a tool turns it into a check. Specs live
**with the software they describe** — in the software repo, beside the code, and
verified by **that repo's CI** — the way tests live next to code, so a spec and the
code it constrains move, version, and break together:

- Behaviour and requirements as `.allium` files — authored and maintained through
  the allium skills (`elicit`/`distill`/`tend`/`weed`/`propagate`), checked by
  `tools/allium` (`allium check` validates structure; `allium analyse` adds
  data-flow, reachability, terminal-state and deadlock analysis; `allium plan`
  derives the test obligations the suite must discharge). The software's CI runs
  `check` + `analyse` + `plan` and fails on any error or warning.
- Timing and concurrency as `.tla` files — checked by `tools/specula` (TLA+/TLC),
  run by the software's CI.

A present spec carries its full lane (see "Mandatory when used", below) — wire the
tool and its skills before authoring.

The host's `plan/<milestone>/` *references* a spec (by path and the software pin);
it does not contain it. Do not place specs in the host's `plan/` tree — quarantining
a spec from its software is a bad smell (the spec drifts from the code). A spec that
ended up under `plan/*/spec/` is relocated into the software repo.

## Personas — `cast/`

`cast/` holds the personas: short profiles of the people the software serves. A
persona is a hypothetical archetypal user, not a real individual. The examples
here (`mara.md`, a human operator, and `wren.md`, an agentic LLM) show the two
modalities; replace them with your project's own.

Build at least one persona by discussion with the human before planning the work
it serves. `cast/applying-personas.md` gives the cited process for doing this —
follow it.

## The three verification lanes

Different kinds of property need different checkers. There are three lanes; route
each claim to the lane that can prove it.

1. **Hygiene** — `tools/host-lint`. Catches naming tells: ordinal labels and bare
   numerals leaking into commit messages, headers, and comments. Runs as a git
   hook.
2. **Requirements** — `tools/allium` (MIT, by JUXT). Does the software meet the
   behaviour the spec states? Author and maintain `.allium` specs **through the
   allium skills**, not by hand: `elicit`/`distill` to author, `tend` to evolve,
   `weed` to find spec↔code divergence, `propagate` to generate the tests. Gate
   each spec in the software's CI with `allium check` (structure) + `allium
   analyse` (data flow, reachability, terminal states, deadlock) + `allium plan`
   (test obligations).
3. **Timing and concurrency** — `tools/specula` (Apache-2.0). TLA+ model
   checking: are the orderings and timings correct? Model-check each `.tla` with
   TLC in the software's CI.

**Mandatory when used (RFC-2119).** Adopting a lane is optional — not every
project needs TLA+ — but **once a spec of a kind exists, its tool, skills, and CI
lane are required.** A component carrying any `.allium` spec MUST wire `tools/allium`
and its skills and run `check` + `analyse` + `plan` in that repo's CI, and the
`plan` obligations MUST be discharged by the software's tests. A component carrying
any `.tla` spec MUST wire `tools/specula` and TLC-check it in that repo's CI. A spec
present without its full lane is a **defect**, not a choice — the lanes are not
reference decoration. The tools are referenced submodules; their skills are
generated, gitignored symlinks (`link-skills.sh`), wired before you author a spec.
This is **enforced**, not only stated: `host-lifecycle software --check` raises a
HAZARD when a materialized component carries a `.allium` with no `allium check` +
`allium analyse` CI workflow, or a `.tla` with no TLC lane.

**Obligations are discharged, not just emitted.** `allium plan` derives a test
obligation for every config default, entity, enum, invariant, rule and transition.
Each obligation MUST be **dispositioned** in a sibling `<spec>.obligations` manifest
— the remap-dictionary discipline applied to tests — as `test:<name>` (a named test
discharges it), `structural` (the spec's own `check`/`analyse` lane covers it), or
`waived: <reason>` (an honest, recorded gap). `host-lifecycle obligations <spec>
--tests <dir>` fails on any undispositioned obligation, any stale disposition, and
any `test:<name>` absent from the test sources; the software's CI runs it, and
`software --check` HAZARDs a `.allium` that has no `.obligations` manifest. An
obligation left undispositioned is a defect — discharge is total, per component.

Two rules govern the tools:

- **Reference, don't vendor.** Each tool is a git submodule pinned to a commit.
  Do not copy its code into this repository.
- **Instruct, don't patch.** Drive the tools through this manual and their own
  interfaces. Do not edit a tool's source to make it fit. If a tool needs a
  change, raise it upstream.

The *output* a tool produces about your project (a report, a counterexample, a
generated check) belongs to your project, not to the tool's license — the same
way a compiler's license does not cover the program it compiles.

## The host-* tools

Three of the tools are ours, released into the public domain (Unlicense):

- **host-grammar** — the shared rules for valid names and numbers. A library, not
  a command. Both tools below depend on it.
- **host-lint** — the *checker*. It reads text and flags naming tells.
- **host-lifecycle** — the *generator*. It allocates numbers and scaffolds
  milestones, decisions, and personas without spending model tokens on
  mechanical work. Run `host-lifecycle next <dir>` for the next number and
  `host-lifecycle validate <dir>` to check a folder. It also materialises and
  audits the *Where* room: `host-lifecycle software --materialize|--check <dir>`
  realises the `.host-software` bare store + worktrees and verifies each is at its
  pin.

Because the generator and the checker share `host-grammar`, what host-lifecycle
emits is exactly what host-lint accepts. Trust that symmetry; do not number by
hand.

### The lifecycle phases — mandatory, no opt-out

host-lifecycle ships one Claude **skill per lifecycle phase**, generated into
`.claude/skills/` by `link-skills.sh` exactly as the allium/specula skills are:
**classify** (preview + case), **adopt** (governance + rooms + stamp), **embed**
(the software as a bare store with worktrees), **remap** (the dictionary rename),
**verify** (the gate sweep — `validate`, `software --check`, `obligations`,
`book --check`), **publish** (the doc site), **upgrade** (the ledger). Each owns
the judgment around its mechanical command.

Unlike a verification lane, which is conditional (required only once a spec of its
kind exists), the lifecycle phases are **unconditional**: every agentic project is
scaffolded, embedded, migrated, verified, published and upgraded through them. You
**MUST** drive each phase through its skill and command — **there is no opt-out**.
Operating the methodology ad-hoc (hand-scaffolding rooms, hand-renaming files,
hand-rolling the site, eyeballing an upgrade diff) is a defect.

## Audited plans and append-only memory

Two disciplines keep the host trustworthy across sessions.

- **Audited plans.** Every change to `plan/` (the milestone index or any
  milestone document) and every decision in `call/` is committed and pushed
  immediately, in its own commit, not batched with code. After you finish a step
  in the software, update the plan to say what you actually did, in a separate
  commit.
- **Append-only memory.** `MEMORY.md` is a running log of decisions, discovered
  constraints, and lessons. Add a short entry whenever you finish something
  significant, hit a non-obvious bug, or find an unexpected constraint — as you
  go, not at the end. Commit it on its own and push it. Never rewrite or delete an
  old entry; if one was wrong, add a new entry that corrects it and points back.

The test for both: a new session with no memory of this conversation should be
able to read `plan/`, `call/`, and `MEMORY.md` and continue without repeating a
past mistake.

## Software and submodule discipline

The **tools** are submodules; the **software** is a bare store with worktrees.
Both follow a commit-upstream-first rule:

- **A tool submodule:** commit and push inside it (on `main`) first, then commit
  the updated submodule pointer in the host and push.
- **The software:** commit and push inside the canonical worktree first, then
  record the new SHA as the `.host-software` `pin` and push that host commit. The
  recorded pin is the audit anchor a gitlink used to be.

Never push a host commit whose tool pointer or software pin is not yet pushed. If
a push fails (no network, no auth), stop, tell the human which commits are
unpushed, and do not start work that depends on them.

**Worktree-absence coherence.** A separately-materialized path — the
software worktree, or a tool submodule — is absent (or empty) until materialized; a
fresh clone, CI, and a partial submodule init do not have it. So **do not git-track
an artifact that depends on such a path existing** (a skill symlink into
`<software>/` *or* into `tools/<tool>/skills/`): gitignore it and **generate** it
after materialization — `link-skills.sh` produces `.claude/skills/*` for the tools
present, and the software's links are recreated after `software --materialize`.
Where an automated context genuinely needs the path, it materializes first;
otherwise it must tolerate the absence. `host-lifecycle software --check` flags any
tracked symlink whose target is not itself tracked here as a `HAZARD`. And: an
un-materialized CI job must exercise each runtime-critical artifact — "done" means
the whole CI sweep is green, not one artifact built.

**Reproducible builds — the production anchor.** Software *initiated* under the
methodology has **reproducible builds**: its deployed artifact MUST be byte-reproducible
from the pinned source plus a recorded build recipe (a pinned `toolchain` and `build`
command in `.host-software`). That is what makes the pin a true production anchor — a
clean rebuild from the pin equals what is deployed — rather than just a source pin.
Record per component which line ships (`deploy`) and the artifact's expected hash
(`artifact = <path> <sha256>`); `host-lifecycle software --check` attests these cheaply
(a present artifact that matches is **verified**; one built by a local toolchain that
differs from the canonical hash is **noted, not failed** — the same reasoning
`--install-hooks` uses, since the recorded hash is the pinned build host's output), and
a CI job runs `host-lifecycle software --verify-build` to rebuild from the pin and fail
unless the artifact reproduces. `--verify-build`, not `--check`, is the reproducibility
proof. For **greenfield** software, non-reproducibility is a defect designed out from the
start.

**Multi-platform builds.** A component whose *one* source pin ships on several platforms
records one `[build "<name>" "<platform>"]` subsection per platform under its
`[software "<name>"]` stanza, each carrying its own `build`/`toolchain`/`artifact`/`deploy`
(and optional `repro-exempt`) plus an `attest-host` naming the OS — `linux`, `windows`,
`macos` — that reproduces it. `--check` and `--verify-build` iterate the builds and attest
each only on its `attest-host`, skipping a build whose host is not the current one rather
than failing (a Linux runner cannot reproduce the Windows artifact, and is not asked to).
The flat single-build fields remain the form for a single-platform component.

**Escape clause (migrated software only).** Pre-existing software brought under the
methodology may not be reproducible yet. It may carry `repro-exempt = call/NNNN` citing a
recorded **case decision** — a software-scoped `call/` decision documenting why it is not
yet reproducible and the interim provenance. `--verify-build` then warns and skips the
rebuild comparison; `--check` still requires the citation to resolve. The exemption is
meant to be retired as the component converges on reproducibility, and is **never**
available to greenfield software.

## Upgrading

Adopting is one event; the template moves on. To **upgrade**, re-apply the spine
changes across the revision span *and* the structural migrations it introduced
(re-embedding the software, bumping a tool) — a doc diff shows the prose but not
the actions. `UPGRADING.md` is the ledger of those actions, one `[upgrade
"<revision>"]` stanza each; `host-lifecycle upgrade <dir>` prints every entry
newer than the repo's `.host` stamp. Fetch the template to the target
revision, run it, apply the list, then re-stamp.

## Provenance

The four working principles are rewritten, in our own words, from observations by
Andrej Karpathy on where LLM coding goes wrong, by way of Jiayuan Zhang
(@forrestchang) and the andrej-karpathy-skills contributors. The persona process
in `cast/applying-personas.md` follows Powell, Keenan and McDaid (2007) on
personas in XP, with later agile work cited alongside it. This manual is released
into the public domain (Unlicense); the credit here is acknowledgement, not a
license obligation.
