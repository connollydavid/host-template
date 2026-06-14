# Tool outputs are project-owned, not under allium or Specula licenses

- Status: accepted
- Date: 2026-06-14

## Context and Problem Statement

The agentic-host methodology composes three referenced tools as submodules:
host-lint (Unlicense, ours), allium (MIT, JUXT Ltd), and Specula (Apache-2.0,
specula-org). Running them produces artifacts inside our project — allium's
`propagate` emits obligations; Specula emits a modeling brief, `base.tla` /
`MC.tla` specs, traces, and bug reports into `.specula-output/`. We need a
recorded position on whether those generated artifacts carry the tools'
licenses, so our project content stays license-clean and we avoid accidental
contamination.

## Considered Options

1. Treat generated artifacts as project-owned, outside the tools' license scope.
2. Treat them as derivative works of the tools (carrying MIT / Apache-2.0).
3. Never generate into our tree (run the tools out-of-tree only).

## Decision Outcome

Chosen option 1: **artifacts the tools generate from our own input are our
project's work product, outside allium's and Specula's license scope** — the
same way a compiled binary is not a derivative of the compiler that produced it.

### Basis (verified against the source repos)

- Both licenses govern only "the Work" / "the Software" — the tools' own files —
  not artifacts produced by running them. allium ships standard MIT (© JUXT Ltd)
  with no output clause. Specula ships standard Apache-2.0 whose "Derivative
  Works" definition refers to works based on Specula's code, not specs that model
  our system. Specula's DCO governs inbound contributions only.
- Both tools frame outputs as the user's: allium "generates tests from the
  formal behaviours of your system"; Specula writes outputs to
  `.specula-output/` under the project root.
- Both authors chose permissive licenses; a verification tool that claimed the
  specs you write about your own system would be self-defeating. So this reading
  is consistent with — not contrary to — the authors' intent.

### Carve-out

Verbatim boilerplate a tool injects — a Specula trace-module or harness
skeleton, an allium obligation template — remains part of that tool's Work and
keeps its license. We reference such boilerplate (for example `EXTENDS` or an
include from the submodule), never vendor it into our tree.

## Consequences

- Good: generated specs and bug reports under a milestone's `spec/` are
  project-owned and Unlicense-clean — no attribution obligation downstream.
- Good: the tools stay in their submodules under their own licenses; the
  separation is clean by construction.
- Risk: the cleanliness depends on referencing tool boilerplate rather than
  copying it; a generated artifact that embeds tool source verbatim carries that
  fragment's license.
- Note: this is practical reasoning, not legal advice; revisit before any
  commercially load-bearing use.
