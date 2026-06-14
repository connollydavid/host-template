# CLAUDE.md — operating manual for an agentic host

This file tells you, the agent, how to work in this repository. Follow it
exactly. The rules are written to be literal: when one says "do X", do X — do
not look for a cleverer path. When something is unclear or could mean two
things, stop and ask the human before you act. Clarity beats cleverness here.

## What this repository is

This repository is an **agentic host**. The host is the externalized *thought*
about a piece of software: the plans, the decisions, the specifications, the
people it serves, and the rules you work under. The software itself — the
*action* — lives in a git submodule beneath the host. You write thought in the
host and action in the submodule. Keep them separate.

You are working in a *template*. A real project replaces the example personas
with its own and adds its software as the hosted submodule. The structure below
stays the same.

## The five rooms

The host has five rooms, one for each question you ask about any piece of work.
Put each kind of file in its room. Do not invent new top-level folders.

| Question | Room | What goes here |
|----------|------|----------------|
| Who  | `cast/` | personas — the people (human or agent) the software serves |
| What | `plan/<milestone>/spec/` | specifications: behaviour (`.allium`), timing (`.tla`) |
| When | `plan/` | the milestone index and one folder per milestone |
| Where | `<software>/` | the hosted software, as a submodule — you add it |
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

## Specs — `plan/<milestone>/spec/`

Each milestone carries its specifications in its own `spec/` folder:

- Behaviour and requirements as `.allium` files — checked by `tools/allium`
  (property-based testing).
- Timing and concurrency as `.tla` files — checked by `tools/specula` (TLA+).

A spec states what the software must do; the tools turn it into a check.

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
2. **Requirements** — `tools/allium` (MIT, by JUXT). Property-based testing: does
   the software meet the behaviour the spec states?
3. **Timing and concurrency** — `tools/specula` (Apache-2.0). TLA+ model
   checking: are the orderings and timings correct?

Two rules govern the tools:

- **Reference, don't vendor.** Each tool is a git submodule pinned to a commit.
  Do not copy its code into this repository.
- **Instruct, don't patch.** Drive the tools through this manual and their own
  interfaces. Do not edit a tool's source to make it fit. If a tool needs a
  change, raise it upstream.

The *output* a tool produces about your project (a report, a counterexample, a
generated check) belongs to your project, not to the tool's license — the same
way a compiler's license does not cover the program it compiles. See `call/0001`.

## The host-* tools

Three of the tools are ours, released into the public domain (Unlicense):

- **host-grammar** — the shared rules for valid names and numbers. A library, not
  a command. Both tools below depend on it.
- **host-lint** — the *checker*. It reads text and flags naming tells.
- **host-lifecycle** — the *generator*. It allocates numbers and scaffolds
  milestones, decisions, and personas without spending model tokens on
  mechanical work. Run `host-lifecycle next <dir>` for the next number and
  `host-lifecycle validate <dir>` to check a folder.

Because the generator and the checker share `host-grammar`, what host-lifecycle
emits is exactly what host-lint accepts. Trust that symmetry; do not number by
hand.

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

## Submodule discipline

The software and the tools are submodules. When you change one:

1. Commit and push inside the submodule first, on its `main` branch.
2. Then commit the updated submodule pointer in the host and push.

Never push a host commit whose submodule pointer is not yet pushed. If a push
fails (no network, no auth), stop, tell the human which commits are unpushed, and
do not start work that depends on them.

## Provenance

The four working principles are rewritten, in our own words, from observations by
Andrej Karpathy on where LLM coding goes wrong, by way of Jiayuan Zhang
(@forrestchang) and the andrej-karpathy-skills contributors. The persona process
in `cast/applying-personas.md` follows Powell, Keenan and McDaid (2007) on
personas in XP, with later agile work cited alongside it. This manual is released
into the public domain (Unlicense); the credit here is acknowledgement, not a
license obligation.
