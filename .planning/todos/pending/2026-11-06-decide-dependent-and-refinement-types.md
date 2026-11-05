---
title: Decide dependent and refinement types
date: 2026-11-06
priority: low
status: pending
---

## What

Dependent and refinement types are a research-grade open axis, not yet
committed. Orthon has a `REFINEMENT_TYPES.md` hypothesis in the
deferrable tier, but no position against dependent types (Idris 2,
Lean 4, Dafny).

## Action

1. Reconstruct the axis: poles (untyped vs refinement vs dependent),
   witnesses (Dafny, Vera, Idris 2, Lean 4), and the contradiction each
   creates (term equality becomes undecidable → totality restrictions).
2. Run the [Decision Pipeline](../../../how/process/DECISION_PIPELINE.md)
   to decide level: core, pattern, or library; a dependent core is the
   antithesis of Minimal Core, so the likely outcome is a library-level
   boundary.
3. Record the decision in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md),
   moving the row from Section B to Section A or filing a rejection.
