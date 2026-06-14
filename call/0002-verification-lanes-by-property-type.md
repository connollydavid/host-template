# Split verification into lanes by property type

- Status: accepted
- Date: 2026-06-14

## Context and Problem Statement

Correctness has distinct property classes, and no single tool covers them all
well. Functional and data invariants suit property-based testing; temporal,
ordering, and concurrency invariants need full state-space exploration; and
neither catches naming or process slop. We need a clear division so each concern
is routed to the tool that actually covers it.

## Considered Options

1. One tool for everything (forces a single tool to cover properties it cannot).
2. Lanes by property type — route each concern to the tool suited to it.

## Decision Outcome

Chosen option 2: **three lanes, one per property class.**

| Lane | Property class | Tool | License |
| --- | --- | --- | --- |
| Hygiene | naming and process slop (agentic tells) | host-lint | Unlicense |
| Requirements + property-based testing | functional and data invariants ("for all inputs") | allium | MIT |
| Timing + concurrency | temporal, ordering, interleaving invariants ("for all interleavings"), via model checking | Specula / TLA+ | Apache-2.0 |

TLA+ is reserved strictly for its home ground — timing and concurrency. It is
not a general requirements tool; that is allium's lane.

### Escalation bridge

When an allium property depends on interleaving or timing that property-based
testing cannot adequately explore, it escalates from the property-based lane to
the timing-and-concurrency lane. allium identifies such a property; Specula
proves the temporal ones.

## Consequences

- Good: every concern has a clear owner; the lanes are complementary, not
  overlapping.
- Good: a milestone may legitimately carry obligations in more than one lane.
- Neutral: contributors must learn which lane a property belongs to; the
  escalation bridge is the rule that resolves the boundary cases.
