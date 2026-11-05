---
title: Decide heterogeneous execution model
date: 2026-11-06
priority: low
status: pending
---

## What

The heterogeneous-execution axis (GPU, SIMD, massively parallel targets)
has no Orthon position. Witnesses Mojo, Bend, and Flint vectors each take
a different stance; the Execution Program model is adjacent but does not
state how a program expresses heterogeneous work.

## Action

1. Reconstruct the axis: poles (explicit device kernels, implicit
   parallelism, SIMD primitives), witnesses (Mojo, Bend, Flint, CUDA),
   and the contradiction each creates (portability vs control).
2. Decide whether heterogeneous execution is a language concern or an
   Implementation Strategy / Policy concern, per the Execution Program
   model and [`IMPLEMENTATION_STRATEGIES.md`](../../../how/strategies/IMPLEMENTATION_STRATEGIES.md).
3. Record the decision in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md),
   moving the row from Section B to Section A or marking it deferred.
