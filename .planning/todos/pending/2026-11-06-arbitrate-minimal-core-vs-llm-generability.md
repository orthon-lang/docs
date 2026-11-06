---
title: Arbitrate minimal core vs LLM generability
date: 2026-11-06
priority: high
status: pending
---

## What

Minimal Core grows expressiveness through modifiers and libraries, but an
LLM predicts not the core — it predicts the whole vocabulary available to
it, which grows by construction. "Small surface, consistent rules" is a
claim about the core, not about what the generator actually sees. This is
a direct contradiction between Minimal Core and Idempotent Generation
(and LLM Readiness, Goal 4) once the standard library becomes large.

## Action

1. Reconcile Minimal Core with Idempotent Generation: define the
   "prediction surface" an LLM must hold (core + stdlib surface), and
   decide whether the small-surface claim extends to the standard
   library or is a core-only property.
2. Run the Decision Pipeline to decide whether this is a principle
   arbitration (Tier 1 note/EDR) or a documented trade-off at the
   library boundary.
3. Record the arbitration in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md)
   § 14.

## Coupled to

- [`how/DESIGN_PRINCIPLES.md`](../../../how/DESIGN_PRINCIPLES.md) § Minimal Core and § Idempotent Generation (the two principles in tension).
