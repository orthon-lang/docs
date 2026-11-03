---
title: Review Composition Surface and Deterministic Behavior fitness functions
date: 2026-11-03
priority: medium
status: pending
---

## What

Assess whether the measurements for Composition Surface and Deterministic
Behavior in [FITNESS_FUNCTIONS.md](../../../how/architecture/FITNESS_FUNCTIONS.md)
provide useful evidence at the current stage of Orthon's design.

## Questions

- **Composition Surface:** A count of canonical forms decreases when a
  redundant synonym is removed, even if every capability remains expressible.
  Check whether the measure distinguishes loss of composability from removal
  of redundant syntax; compare it with assessing lost capabilities directly.
- **Deterministic Behavior:** The current measure requires compiling and
  running a program under two Implementation Strategies, but no compiler or
  runtime exists yet. Determine what design-stage evidence can be checked now,
  what it can establish, and which executable checks must wait for an
  implementation.

## Action

1. Trace each function's intent and existing sources, including
   [EDR-005](../../../how/decision_records/quality/EDR-005-fitness-functions.md),
   and evaluate its current measure against the cases above.
2. For each function, compare retaining, revising, and retiring the measure;
   record a recommendation, limitations, and a concrete verification method.
3. Coordinate any resulting decisions and individual EDRs with FITNESS-02.
   Follow the decision process before changing an accepted rule; do not treat
   a documented scenario as a passed executable test.
