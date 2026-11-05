---
title: Decide evolution and compatibility model
date: 2026-11-06
priority: high
status: pending
---

## What

The evolution-and-compatibility axis is Orthon's central bet
("consistency over historical compatibility") and its hardest blank:
[`how/EVOLUTION_MODEL.md`](../../../how/EVOLUTION_MODEL.md) is still a
Phase 8 placeholder, and no document states how the language and the
standard library evolve without violating the manifesto.

## Action

1. Reconstruct the axis: poles (never-break vs epoch-breaking vs
   stability guarantees), witnesses (Java, Python 3, Go, Rust), and the
   contradiction each creates — notably that "consistency over
   historical compatibility" pushes all change into libraries, which
   have no stated evolution policy.
2. Fill `EVOLUTION_MODEL.md`: versioning scheme, deprecation, library
   vs core evolution, and the boundary where a contradiction in the
   library becomes a core change (Core Change Criterion).
3. Record the position, trade-off, and new contradiction in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md),
   moving the row from Section B to Section A.
