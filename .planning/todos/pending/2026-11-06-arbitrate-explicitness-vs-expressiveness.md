---
title: Arbitrate explicitness vs Python-level expressiveness
date: 2026-11-06
priority: medium
status: pending
---

## What

VISION promises a language "as expressive as Python", while Explicit
Semantics requires mutation, ownership, and allocation to be visible in
the syntax. Every explicit marker is surface area. The Manifesto asserts
both poles as principles and does not arbitrate between them — a deferred
contradiction, not a resolved one.

## Action

1. Decide whether the "as expressive as Python" claim is literal or
   aspirational, and state the expressiveness bar against the cost of
   explicit markers.
2. Either clarify the VISION/Manifesto wording (Tier 1, requires an EDR),
   or record a documented trade-off that the expressiveness goal is
   bounded by Explicitness.
3. Record the arbitration in
   [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md)
   (a cross-cutting principle tension spanning the Data, Memory, and
   Syntax axes).

## Coupled to

- [`why/VISION.md`](../../../why/VISION.md) ("as expressive as Python, as controllable as Rust")
- [`why/MANIFESTO.md`](../../../why/MANIFESTO.md) ("Explicit Semantics")
- [`how/DESIGN_PRINCIPLES.md`](../../../how/DESIGN_PRINCIPLES.md) § Explicitness
