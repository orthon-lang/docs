---
title: Record a separate EDR for each existing fitness function
date: 2026-11-03
priority: medium
status: pending
---

## What

Review every function in [FITNESS_FUNCTIONS.md](../../../how/architecture/FITNESS_FUNCTIONS.md) and create a separate Quality EDR for each one that lacks an individual record. Use [EDR-101](../../../how/decision_records/quality/EDR-101-invariant-violation-detectability.md) as the format and retain it for Invariant Violation Detectability.

## Action

1. Inventory all functions and their existing sources and EDRs. Reconstruct the original purpose, measurement, threshold, and consequences from available evidence; identify unknown history explicitly rather than inventing it.
2. For each function without its own record, document the problem, decision, measurement, consequences, compliance, alternatives, and applicable gate validation. Distinguish an existing decision from a proposed change and route substantive changes through the decision process.
3. Add each new EDR to both index tables and link it from the corresponding function. Verify paths, status, and numbering; do not duplicate EDR-101.
