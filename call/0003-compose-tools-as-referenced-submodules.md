# Compose verification tools as referenced submodules

- Status: accepted
- Date: 2026-06-14

## Context and Problem Statement

The three lanes (see decision 0002) are owned by tools under three different
licenses: host-lint (Unlicense, ours), allium (MIT), and Specula (Apache-2.0).
They must fit our milestone (decision-record) lifecycle and obligations model
without contaminating our license posture, forcing us to maintain a fork, or
turning a tool's internal workflow into a competing project axis.

## Considered Options

1. Vendor each tool's files into our tree and edit them to fit.
2. Reference each tool as a submodule and steer it from our own layer.

## Decision Outcome

Chosen option 2: **each tool is a referenced submodule; we orchestrate, never
edit.**

- **Reference, never vendor.** Each tool is a git submodule; our methodology
  (CLAUDE.md) refers to its skills. We never copy or edit its files — behaviour
  is steered by instruction plus thin wrappers in our own tree. This keeps us
  clean of Apache-2.0 obligations and keeps submodule updates trivial.
- **Specs are tagged by lane and co-located with tests.** A spec is `.allium`
  (allium) or `.tla` (Specula); both live in the milestone's `spec/` area. A
  tool's generated artifacts (allium's behavioural spec and generated tests,
  Specula's modeling brief, TLA+ specs, traces, confirmed bugs) are the discharge
  evidence, and are project-owned per decision 0001.
- **A tool's workflow discharges, it does not milestone.** A tool's internal
  pipeline runs inside one milestone to discharge that milestone's obligations;
  it is not a parallel milestone axis. Specula's steps are referred to by name
  (Code Analysis, Spec Generation, and so on), never as ordinal phases — keeping
  our own host-lint convention.
- **Lane choice is a decision.** Choosing a lane for a concern, or classifying a
  Specula target category, is itself an architectural decision and gets its own
  record.
- **Boilerplate is referenced, not vendored.** Verbatim tool boilerplate (a
  Specula trace-module, an allium template) is pulled in by reference, never
  copied — the carve-out from decision 0001.

## Consequences

- Good: clean license separation by construction; our content stays Unlicense;
  submodule bumps are mechanical.
- Good: one numbered milestone register, undisturbed by the tools' internal
  step naming.
- Risk: correctness depends on the reference-not-vendor and instruct-not-patch
  discipline; patching a submodule is a designed-around failure mode, not a
  routine tool.
