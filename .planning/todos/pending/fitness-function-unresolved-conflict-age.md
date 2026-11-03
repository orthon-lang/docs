---
title: Add Unresolved Conflict Age fitness function and EDR
date: 2026-11-03
priority: medium
status: pending
---

## What

Design and add an Unresolved Conflict Age fitness function to make persistent unresolved concept-boundary conflicts visible.

## Action

1. Review [CONFLICT_REGISTRY.md](../../../what/CONFLICT_REGISTRY.md) and define the source of dates, which statuses count as unresolved, the measurement, review trigger, and treatment of deferred conflicts. Choose the threshold through the decision process; do not assume one.
2. Record the rationale, alternatives, consequences, and gate validation in a separate Quality EDR using [EDR-101](../../../how/decision_records/quality/EDR-101-invariant-violation-detectability.md) as an example.
3. Add the function to [FITNESS_FUNCTIONS.md](../../../how/architecture/FITNESS_FUNCTIONS.md), link its EDR, and register that EDR in both index tables.
