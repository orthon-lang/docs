---
title: Decide effect-tracking model
date: 2026-11-06
priority: medium
status: pending
---

## What

Orthon already declares `effects` in module headers (per AGENTS.md §2a)
and has `EFFECT_FOOTPRINT.md` in the essential tier, but no accepted
position on the effect-tracking axis: monads vs algebraic effects vs
capabilities. The axis is part-committed, part-research, and needs a
reconciled decision.

## Action

1. Reconstruct the axis: poles (untyped effects, monads, algebraic
   effects, capabilities), witnesses (Haskell, Koka, Vera, Idris 2,
   Unison), and the contradiction each creates (effect inference
   non-locality; handler abstraction limits).
2. Reconcile the module-header `effects` declaration with
   `EFFECT_FOOTPRINT.md` and the Execution Context model (EDR-085):
   which effects are tracked, where the boundary is drawn.
3. Run the [Decision Pipeline](../../../how/process/DECISION_PIPELINE.md)
   and record the position, trade-off, and new contradiction in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md).
