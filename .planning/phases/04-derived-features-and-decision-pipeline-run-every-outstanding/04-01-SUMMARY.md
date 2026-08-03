---
phase: 04
plan: 01
subsystem: "Derived Features & Decision Pipeline"
tags: ["essential-core", "equality", "null-safety", "traits", "decision-pipeline", "gate-validation"]
requires: ["03-01-primitive-blocks", "03-02-primitive-verification"]
provides: ["concept-specs", "edrs-017-019"]
affects: ["what/CORE_CONCEPTS.md", "what/GLOSSARY.md", "how/decision_records/INDEX.md"]
tech-stack:
  added: []
  patterns: ["Concept Design Review (5-step)", "Decision Pipeline (10-question)", "Gate Validation (7-gate)"]
key-files:
  created:
    - what/concepts/EQUALITY.md
    - what/concepts/NULL_SAFETY.md
    - what/concepts/TRAITS.md
    - how/decision_records/architecture/EDR-017-equality.md
    - how/decision_records/architecture/EDR-018-null-safety.md
    - how/decision_records/architecture/EDR-019-traits.md
  modified:
    - how/process/DECISION_PIPELINE.md
    - how/decision_records/INDEX.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
decisions:
  - "EQUALITY: Three-operator model (=, ==, is) — Language classification per D-03"
  - "NULL_SAFETY: Option<T> without null sentinel — Language classification per D-03"
  - "TRAITS: Nominal trait system with orphan rule — Language classification per D-03"
metrics:
  duration: "~45 min"
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 1: Essential Core — Decision Pipeline & Concept Acceptance

## One-liner

Processed **3 foundational essential-tier concepts** (EQUALITY, NULL_SAFETY, TRAITS) through the full acceptance pipeline: Decision Pipeline (10 questions), D-03 classification, 7 Validation Gates, 5-step Concept Design Review → accepted concept docs + EDRs + registry updates.

## Files Created

| File | Description |
|------|-------------|
| `what/concepts/EQUALITY.md` | Accepted concept spec: three-operator equality model (===, ==, is). Structural by default. Transitivity Invariant. NaN deferred. |
| `what/concepts/NULL_SAFETY.md` | Accepted concept spec: Option<T> sum type. No null sentinel. ?., ??, ! operators. Exhaustive match required. |
| `what/concepts/TRAITS.md` | Accepted concept spec: Nominal trait system. Explicit impl. Static dispatch default. dyn Trait opt-in. Orphan rule. Associated types. Template Method pattern. |
| `how/decision_records/architecture/EDR-017-equality.md` | EDR: Three-Operator Model. All 7 gates pass. |
| `how/decision_records/architecture/EDR-018-null-safety.md` | EDR: Option Type Without Null Sentinel. All 7 gates pass. |
| `how/decision_records/architecture/EDR-019-traits.md` | EDR: Nominal Trait System. All 7 gates pass. |

## Files Modified

| File | Change |
|------|--------|
| `how/process/DECISION_PIPELINE.md` | Added Essential Core — Wave 1 subsection with Q1-Q10 for all three concepts |
| `how/decision_records/INDEX.md` | Added EDR-017 through EDR-019 to All Records and Architecture By Category |
| `what/CORE_CONCEPTS.md` | Added Wave 1 registry entries for EQUALITY, NULL_SAFETY, TRAITS |
| `what/GLOSSARY.md` | Added: Identity Equality, Option Type, Orphan Rule, Semantic Equality, Trait, Trait Bound, Value Equality |

## Pipeline Application Results

All three concepts classified as **Language** per D-03 criteria (semantic uniqueness + compiler dependency):

| Concept | Q2 | Q3 | Q5 | D-03 Classification |
|---------|----|----|----|---------------------|
| EQUALITY | Language | Not expressible via primitives | New semantics | Language (=== requires compiler structural comparison) |
| NULL_SAFETY | Language | Not expressible via primitives | New semantics | Language (Option adds ? semantics, compiler must track nullable state) |
| TRAITS | Language | Not expressible via primitives | New semantics | Language (Compiler must resolve trait bounds and dispatch) |

## Gate Validation Summary

All seven gates passed for all three EDRs:

| Gate | EQUALITY | NULL_SAFETY | TRAITS |
|------|----------|-------------|--------|
| USER_VALUE | Pass | Pass | Pass |
| LOGICAL_CONSISTENCY | Pass | Pass | Pass |
| CONCEPTUAL_SIMPLICITY | Pass | Pass | Pass |
| ARCHITECTURAL_INTEGRITY | Pass | Pass | Pass |
| IMPLEMENTATION_INDEPENDENCE | Pass | Pass | Pass |
| LONG_TERM_MAINTAINABILITY | Pass | Pass | Pass |
| LLM_GENERABILITY | Pass | Pass | Pass |

## Deviations from Plan

None — plan executed exactly as written with all required artifacts created.

## Known Stubs

None — all concept specs are specification-quality (no placeholders, no DRAFT markers, no empty sections).

## Threat Flags

None — all created files are documentation-only, no security-relevant surface introduced.

## Self-Check: PASSED

All 6 new files verified present with substantial content (808 total lines). All 4 modified files verified updated. All commits confirmed.
