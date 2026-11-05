---
title: Decide evaluation strategy (strictness, totality)
date: 2026-11-06
priority: medium
status: pending
---

## What

Orthon has no stated position on the evaluation-strategy axis: strict vs
lazy, and total vs partial. This is a blank axis in
[`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md)
Section B.

## Action

1. Run the [Decision Pipeline](../../../how/process/DECISION_PIPELINE.md)
   Q1–Q10 to decide whether evaluation strategy is a language problem or
   a library/strategy concern.
2. Reconstruct the axis dialectically per
   [DIALECTICAL_METHOD.md](../../../how/process/DIALECTICAL_METHOD.md):
   poles (eager vs lazy, total vs partial), witnesses (Haskell, Lean 4,
   Idris 2), and the contradiction each synthesis creates (laziness vs
   effects/memory/equality; totality vs expressiveness — the Halting
   Problem as limit).
3. Record the position, trade-off, and new contradiction; move the row
   from Section B to Section A of the ledger, or write a concept
   hypothesis and register it in the research inbox.
