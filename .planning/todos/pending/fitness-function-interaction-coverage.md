---
title: Add Interaction Coverage fitness function and EDR
date: 2026-11-03
priority: medium
status: pending
---

## What

Design and add an Interaction Coverage fitness function that identifies relevant concept interactions whose combined behaviour is not yet documented.

## Action

1. Review the interaction matrix, conflict registry, and existing Orthogonality and Nyquist checks. Define relevance, applicability, measurement, evidence, threshold, and treatment of justified exclusions without assuming all pairs are meaningful.
2. Record the rationale, alternatives, consequences, and gate validation in a separate Quality EDR using [EDR-101](../../../how/decision_records/quality/EDR-101-invariant-violation-detectability.md) as an example.
3. Add the function to [FITNESS_FUNCTIONS.md](../../../how/architecture/FITNESS_FUNCTIONS.md), link its EDR, and register that EDR in both index tables.
