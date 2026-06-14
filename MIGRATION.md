# MIGRATION.md — bring an existing repo under the agentic-host methodology

This is the protocol for *adopting* the methodology into a repo that already
exists, and for *upgrading* a repo that adopted an earlier version. Follow it in
order. It is written to be literal: do each step, run the stated check, and stop
to ask the human at the points marked **(human)**.

Two things vary, independently:

- **Case** — what governance the repo already has (its starting state).
- **Mode** — how much you change, and whether history moves (the blast radius).

Pick the case first — it decides how you establish governance — then the mode,
which decides how you apply the audit.

## Cases

| Case | The repo has | You will |
|---|---|---|
| **a** | no `CLAUDE.md` | drop in the canonical manual and record the repo's own conventions |
| **b** | a `CLAUDE.md` that predates this methodology | **merge** its rules with the spine **(human)** |
| **c** | a `.host` stamp (it adopted an earlier revision) | **upgrade** by diffing template revisions |

`host-lifecycle classify <dir>` prints `a`, `b`, or `c` for you.

## Modes and how to choose

| Mode | What it touches | Use when |
|---|---|---|
| **Preview** | nothing (report only) | always — run it first |
| **Shallow PR** *(default)* | one branch/PR; live files only; history untouched | almost always |
| **Staged** | several PRs; live files only | the live changes are too large to review in one PR |
| **Deep rewrite** | history (archive-first, force-push) | only when history coherence is worth more than its provenance |

**Selection rule.** Default to **Shallow**. Go to **Staged** when the live diff is
too big to review in one sitting. Choose **Deep** *only* when rewriting history
buys coherence worth more than the disruption — and **never** on history that
carries provenance you do not control (an upstream patch series, tags others
have pulled). Deep always means: create and push a protected `archive/` branch
first, then rewrite, then force-push with lease.

History is **immutable by default**. In every mode except Deep, tells already in
the commit log are *acknowledged*, not rewritten.

## The stamp — `.host`

A repo records which template revision it adopted, at the repo root:

```
template = "https://github.com/connollydavid/host-template"
revision = "<sha-or-tag>"
adopted  = "YYYY-MM-DD"
```

`host-lifecycle adopt` writes it; `host-lifecycle version` reads it. The
`revision` is what a later case-(c) upgrade diffs from, so it must be exact.

## The protocol

### 0. Preview

- `host-lifecycle classify <dir>` → the case.
- `host-lint --all` → naming tells in **live tracked files** (you will fix these).
- `host-lint --log` → tells in **history** (informational; do not rewrite unless Deep).
- From the output, write down the **rename map** (each ordinal-named file →
  its content-named home under `plan/`) and, for case (b), the **merge plan**.
  The rename map becomes the `.host-remap` dictionary in step 4. Apply nothing yet.

### 1. Establish governance (by case)

- **(a)** Copy this template's `CLAUDE.md` in unchanged. Then find the repo's
  implicit conventions — its build, test, and style rules — by reading the code
  or asking the human, and record them under a project-specifics heading. Do not
  impose a rule that contradicts the repo's existing style.
- **(b) (human)** Merge. For each rule in the existing `CLAUDE.md`, decide:
  *subsumed* by the spine (drop it, note it in provenance), *project-specific*
  (keep it, move it under the project-specifics heading), or *conflicts* with the
  spine (stop, get a human ruling). Preserve any attribution or license the old
  file carried. The result is: the canonical spine + the kept specifics + a
  provenance note.
- **(c)** Upgrade. Read the stamp's `revision`; diff it against this template's
  current revision; re-apply the spine changes; leave the repo's project-specifics
  untouched. If tool submodules were renamed since, re-point them.

### 2. Scaffold the rooms

`host-lifecycle adopt <dir> <current-template-revision>` creates `cast/ plan/
call/` (idempotently — existing rooms are left alone) and writes the stamp. Add
your software as the hosted submodule under its own folder (the *Where* slot).

### 3. Wire the tools

Add the verification tools as submodules and symlink their skills:

```
tools/host-lint  tools/host-lifecycle  tools/allium  tools/specula
```

Install the host-lint git hooks (`pre-commit` and `commit-msg`) so new commits
are gated from here on.

Apply the migration in two layers — the live files, and the append-only record.

**Live layer — rename with the dictionary.** Turn the rename map into a
`.host-remap` dictionary at the repo root, `old => new` per line: a milestone
header (`Phase 4: Command Execution => Command Execution`), a path
(`plan/PHASE4.md => plan/0004-command-execution/README.md`), a bare ordinal
mention (`Phase 4 => Command Execution`). `git mv` each ordinal-named file to its
content-named home, then let the tool rewrite the references deterministically:

- `host-lifecycle remap --check <dir>` lists every tell that would *remain* after
  the dictionary applies. Disposition each: add a dictionary entry, add a
  `.host-lint-allow` entry (genuine vocabulary — version strings, release tags),
  or confirm it sits in an excluded path (below). Iterate until it is clean.
- Commit first (a clean git tree is the verbatim archive), then
  `host-lifecycle remap --apply <dir>` writes the substitutions — only the
  declared ones, so the rewrite is map-only by construction: nothing outside the
  dictionary is touched, and there is nothing to invent. Commit the result, then
  `git rm .host-remap` — the dictionary names the old concepts, so it does not
  stay in the tree; its durable copy goes in the `call/` decision.
  - *Shallow*: one PR. *Staged*: split governance → tooling → the bulk rename.
  - *Deep (human, opt-in)*: archive-first, then also rewrite history so blame and
    `log --follow` stay coherent; force-push with lease.

**Record layer — exclude, don't rewrite.** The append-only record (`MEMORY.md`
and the closed milestone bodies) is history; nothing re-scans it, so leave it
verbatim and exclude it from the audit with a `.host-lintignore` at the repo root:

```
MEMORY.md
plan/*/README.md
```

The renamed `plan/<NNNN-slug>/` folders are the old→new map a later reader needs.
Rewrite the record only if a human asks — still map-only and archive-first; it is
rarely worth the cost.

**History findings** (`host-lint --log`): acknowledge them. Do not rewrite unless
you are in Deep mode and the human opted in.

### 5. Stamp and record

The stamp is already written (when you scaffolded the rooms). Record a decision in `call/` — "adopted
template @ `<revision>`" (or "upgraded `<old>` → `<new>`") — and add a `MEMORY.md`
entry. These make the migration auditable from a fresh session.

### 6. Verify

- `host-lifecycle validate plan/` and `host-lifecycle validate call/` → `ok`.
- `host-lint --all` → clean on the live files (the record is excluded via
  `.host-lintignore`).
- Make a throwaway commit with a tell in its message → the hook blocks it.
- If the repo ships an mdBook site, it builds.

## Notes

- **Idempotent and resumable.** `adopt` skips rooms that exist; re-running the
  protocol is safe. If interrupted, re-run `classify` and continue.
- **Token-free where mechanical.** `host-lifecycle` does the scaffolding,
  numbering, stamping, and the dictionary `remap`; `host-lint` does the audit.
  Spend model effort only on the judgment: establishing governance (the case-(b)
  merge, eliciting case-(a) conventions), naming each milestone, and the triage
  during the audit. The substitution itself is deterministic.
- **Repo-root config the tools read.** `.host-remap` (the rename dictionary,
  transient), `.host-lint-allow` (sanctioned vocabulary the audit never flags),
  and `.host-lintignore` (paths the audit excludes — the record). All are plain
  line-oriented files.
