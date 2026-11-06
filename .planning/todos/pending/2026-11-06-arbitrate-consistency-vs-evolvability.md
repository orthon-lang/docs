---
title: Arbitrate consistency vs evolvability (library evolution)
date: 2026-11-06
priority: high
status: pending
---

## What

The current Orthon synthesis creates a tension between coherence and
evolvability. "Consistency over historical compatibility" pushes all
change out of the core and into libraries, but the stability question
does not disappear — it relocates to the standard library, which has
neither versioning nor a deprecation policy. `EVOLUTION_MODEL.md` is a
Phase 8 stub, and `VERSIONING_POLICY.md` has no library-evolution
section. Prediction: Orthon's hardest problem will be library evolution,
not core evolution. Orthon is on a synthetic position on the "core" axis
but on an extreme on the "library" axis.

## Action

1. Verify the prediction against the corpus: confirm that core-change
   policy (Core Change Criterion) exists but library-change policy does
   not.
2. Decide whether "consistency over historical compatibility" extends to
   the standard library, and what versioning and deprecation the library
   gets. Fill the missing library-evolution section of
   `EVOLUTION_MODEL.md` / `VERSIONING_POLICY.md`.
3. Record the arbitration in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md)
   § 13, moving the axis from Section B to Section A.

## Coupled to

- [`2026-11-06-decide-evolution-and-compatibility-model.md`](2026-11-06-decide-evolution-and-compatibility-model.md)
  (the axis-level TODO; this one arbitrates the principle-level
  consistency-vs-evolvability tension it surfaces).
