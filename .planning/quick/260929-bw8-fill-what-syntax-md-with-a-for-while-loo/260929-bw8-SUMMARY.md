---
phase: quick-260929-bw8
plan: 01
subsystem: what-layer-specification
status: complete
tags: [syntax, iteration, execution-model, homing, EDR-053, EDR-094]
requires: [EDR-053, EDR-094, EDR-021, EDR-022, EDR-085]
provides:
  - what/syntax/ITERATION_SYNTAX.md (accepted iteration surface-form record)
  - what/EXECUTION_MODEL.md § Iteration Semantics
affects:
  - what/SYNTAX.md (hub table)
  - what/syntax/README.md (Records index)
tech-stack:
  added: []
  patterns: [RANGE_SYNTAX.md syntax-record precedent, Semantics-vs-Execution framing]
key-files:
  created:
    - what/syntax/ITERATION_SYNTAX.md
  modified:
    - what/SYNTAX.md
    - what/syntax/README.md
    - what/EXECUTION_MODEL.md
decisions:
  - Homed EDR-053 / EDR-094 into their "What"-layer destinations; re-decided nothing.
metrics:
  duration: ~10m
  completed: 2026-09-29
  tasks: 2
  files: 4
actuals:
  tasks: 2
  commits: 2
  plan_head_before: 794510fc32f47f54f1548c008fdc22529584f965
  plan_head_after: a37bc22
---

# Quick 260929-bw8: Home accepted iteration syntax into what/syntax + EXECUTION_MODEL Summary

Homed two LOCKED iteration decisions (EDR-053 iteration loop; EDR-094 for-expression → `Optional<T>` and iteration ownership) into their "What"-layer destinations: a new `what/syntax/ITERATION_SYNTAX.md` surface-form record (registered in the hub + index) and a new "Iteration Semantics" section in `what/EXECUTION_MODEL.md`. Closed the two deferrals ITERATION_LOOP.md § Affected Documents had marked against `what/SYNTAX.md` and `what/EXECUTION_MODEL.md`. Pure homing — no syntax invented, no ownership matrix re-derived, no C-style loop introduced.

## Tasks

### Task 1 — Create ITERATION_SYNTAX.md + register (commit 02401c5)
- Created `what/syntax/ITERATION_SYNTAX.md` following the RANGE_SYNTAX.md shape: accepted-note blockquote, `## Canonical forms` (one orthon block), `## Rules` (8 numbered), `## Cross-References`.
- Canonical forms copied verbatim from ITERATION_LOOP.md § Model: `for`/`while`/`loop`, `break`/`continue`, the `for`-expression (`-> Some/None`, `found : Optional<Row>`), and the three source-side ownership markers (`coll` → `&T`, `&mut coll` → `&mut T`, `$coll` → `T`). No C-style loop; ownership matrix not reproduced.
- Registered a hub row in `what/SYNTAX.md` (Decision EDR-053 / EDR-094) and a `## Records` entry in `what/syntax/README.md`.

### Task 2 — Iteration Semantics section (commit a37bc22)
- Inserted `## Iteration Semantics` into `what/EXECUTION_MODEL.md` after `## Invocation Semantics` (post `### Distribution Operator`) and before `## EDR`, preserving the doc's Semantics-vs-Execution framing.
- `### Language Guarantees`: single `IntoIterator::iter()` + `next()` protocol (EDR-022, references concept doc for desugaring), lazy/single-pass, no hidden allocation, borrow-by-default (EDR-094), generators as stackless state machines (EDR-021).
- `### Implementation Latitude`: three-column table (Aspect | Language | Implementation) matching the doc's existing shape; noted these are Strategy-Profile choices.
- `### Iteration and Execution Contexts`: `spawn()`/`fork()` results join the generator stream, drained via `next()`/`stop()` or `grab`/`gather`, referencing (not duplicating) the Execution Contexts table above (EDR-085).

## Deviations from Plan

None — plan executed exactly as written.

## Verification

- Task 1 automated verify: PASS (file exists, cites EDR-053/EDR-094, registered in hub + README, all prerequisite files present).
- Task 2 automated verify: PASS (`## Iteration Semantics`, single-pass, IntoIterator, gather, EDR-022, EDR-021 all present).
- All introduced relative links resolve (10 links checked from `what/syntax/` and `what/`).
- Out-of-scope files untouched: `what/concepts/ITERATION_LOOP.md`, all EDRs, `how/decision_records/INDEX.md`. ROADMAP.md not modified.

## Self-Check: PASSED

- what/syntax/ITERATION_SYNTAX.md — FOUND
- Commit 02401c5 — FOUND
- Commit a37bc22 — FOUND
