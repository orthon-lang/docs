---
phase: 04
plan: 02
subsystem: "Essential Core — Wave 2"
tags: [essential-core, decision-pipeline, concept-design-review, edr]
requires: [04-01]
provides: [ERROR_UNION, GENERICS, PATTERN_MATCHING, PATTERN_MATCHING_DISPATCH, TYPE_INFERENCE, TYPE_LEVEL_NULL_SAFETY]
affects: [what/CORE_CONCEPTS.md, what/GLOSSARY.md, how/decision_records/INDEX.md, how/process/DECISION_PIPELINE.md]
tech-stack:
  added: []
  patterns: [Concept-as-EDR pipeline, Gate Validation tables, Bidirectional inference model]
key-files:
  created:
    - how/decision_records/architecture/EDR-023-error-union.md
    - how/decision_records/architecture/EDR-024-generics.md
    - how/decision_records/architecture/EDR-025-pattern-matching.md
    - how/decision_records/architecture/EDR-026-pattern-matching-dispatch.md
    - how/decision_records/architecture/EDR-027-type-inference.md
    - how/decision_records/architecture/EDR-028-type-level-null-safety.md
    - what/concepts/ERROR_UNION.md
    - what/concepts/GENERICS.md
    - what/concepts/PATTERN_MATCHING.md
    - what/concepts/PATTERN_MATCHING_DISPATCH.md
    - what/concepts/TYPE_INFERENCE.md
    - what/concepts/TYPE_LEVEL_NULL_SAFETY.md
  modified:
    - how/process/DECISION_PIPELINE.md
    - how/decision_records/INDEX.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
decisions:
  - "ERROR_UNION: Adopt Zig-style !T as primary error representation, coexisting with Result<T,E> for payload-bearing errors. Inferred error sets. Structural widening. ? operator shared."
  - "GENERICS: Trait-bounded parametric polymorphism. Static dispatch by default (monomorphisation). Invariant by default. No HKT in v0.1. Cross-ref COMPILE_TIME_EXECUTION."
  - "PATTERN_MATCHING: Exhaustive, expression-oriented match. Destructuring, guards, or patterns. First-match precedence. Depends on TRAITS for sealed trait exhaustiveness."
  - "PATTERN_MATCHING_DISPATCH: Multimethod dispatch via definition-site match declaration form. Specificity resolution. Exhaustiveness across arguments. Complements trait dispatch."
  - "TYPE_INFERENCE: Local bidirectional inference. Annotations at public API boundaries. No cross-module inference. Turbofish ::<T> for disambiguation. Defer :Type syntax to Phase 5."
  - "TYPE_LEVEL_NULL_SAFETY: Flow-sensitive narrowing on Option<T> after match/check. Per-variable, conservative, resets on reassignment. ! escape hatch."
metrics:
  duration: ""
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 2: Essential Core — Wave 2 Summary

**One-liner:** Processed 6 essential-tier core concepts (ERROR_UNION, GENERICS, PATTERN_MATCHING, PATTERN_MATCHING_DISPATCH, TYPE_INFERENCE, TYPE_LEVEL_NULL_SAFETY) through the full Decision Pipeline — Pipeline Q&A, D-03 Classification, 7 Validation Gates, EDR acceptance, and concept specification writing.

## Overview

Executed Plan 04-02 of Phase 04 (Derived Features & Decision Pipeline). For each of 6 concepts, followed the full procedure: Decision Pipeline Q&A → Classification per D-03 → 7 Validation Gates → EDR → concept doc → registry update.

## Files Created

### EDRs (6 files)

| EDR | Concept | Classification | Key Decision |
|-----|---------|---------------|--------------|
| EDR-023 | Error Union | Language | Zig-style `!T` tag-only error union with inferred error sets. Coexists with `Result<T,E>`. |
| EDR-024 | Generics | Language | Trait-bounded parametric polymorphism. Monomorphisation default. Invariant default. |
| EDR-025 | Pattern Matching | Language | Exhaustive, expression-oriented. Destructuring, guards, or patterns. |
| EDR-026 | Pattern Matching Dispatch | Language | Definition-site `match` declaration form. Specificity resolution. |
| EDR-027 | Type Inference | Language | Local bidirectional. Annotations at public API boundaries. No cross-module. |
| EDR-028 | Type-Level Null Safety | Language | Flow-sensitive narrowing after match/check. Conservative. Per-variable. |

### Concept Specification Documents (6 files)

| File | Source EDR | Dependencies |
|------|-----------|--------------|
| `what/concepts/ERROR_UNION.md` | EDR-023 | ERROR_HANDLING (EDR-020) |
| `what/concepts/GENERICS.md` | EDR-024 | TRAITS (EDR-019), COMPILE_TIME_EXECUTION cross-ref |
| `what/concepts/PATTERN_MATCHING.md` | EDR-025 | TRAITS (EDR-019), SMART_CAST cross-ref |
| `what/concepts/PATTERN_MATCHING_DISPATCH.md` | EDR-026 | PATTERN_MATCHING (EDR-025), TRAITS (EDR-019) |
| `what/concepts/TYPE_INFERENCE.md` | EDR-027 | EQUALITY (EDR-017) for type unification |
| `what/concepts/TYPE_LEVEL_NULL_SAFETY.md` | EDR-028 | NULL_SAFETY (EDR-018), PATTERN_MATCHING (EDR-025) |

## Registry Updates

- **`how/process/DECISION_PIPELINE.md`** — Added "Essential Core — Wave 2" subsection with full Pipeline Q&A for all 6 concepts (6×10 questions each)
- **`how/decision_records/INDEX.md`** — Added EDR-023 through EDR-028 to both All Records table and Architecture By Category table; updated Status Summary count from 17 to 23
- **`what/CORE_CONCEPTS.md`** — Added Wave 2 entries for all 6 concepts with EDR references, classifications, summaries, and primitive decomposition paths
- **`what/GLOSSARY.md`** — Added 12 new terms: Error Set, Error Tag, Error Union, Exhaustiveness, Flow-Sensitive Narrowing, Generics, Multimethod Dispatch, Narrowing, Pattern Matching, Pattern Matching Dispatch, Type Inference, Type-Level Null Safety

## Key Decisions Made

1. **ERROR_UNION** — Classification Language, not composable from primitives. Coexists with `Result<T,E>`. Unified `?` operator. No `try`/`catch`.
2. **GENERICS** — Invariant by default. No HKT in v0.1. Cross-references COMPILE_TIME_EXECUTION (Plan 04-03) for comptime interactions.
3. **PATTERN_MATCHING** — Exhaustiveness required (compile-time error), not warnings-only. Expression-oriented.
4. **PATTERN_MATCHING_DISPATCH** — Definition-site declaration (all arms together). Specificity resolution with compile-time error on ties.
5. **TYPE_INFERENCE** — Local bidirectional. No cross-module inference. Concrete `: Type` syntax deferred to Phase 5.
6. **TYPE_LEVEL_NULL_SAFETY** — Conservative narrowing. Per-variable, flow-sensitive. Resets on reassignment.

## Deviations from Plan

None — plan executed exactly as written.

## Commit

`59eba0b` — 16 files changed, 2267 insertions, 18 deletions

## Self-Check: PASSED
