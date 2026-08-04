---
phase: 04
plan: 03
subsystem: "Derived Features & Decision Pipeline"
tags: ["essential-core", "ast-macros", "comptime", "concurrency", "collection-ops", "static-analyzer", "decision-pipeline"]
requires: ["04-01-essential-core-wave1"]
provides: ["concept-specs-wave2-essential", "edrs-029-033"]
affects: ["what/CORE_CONCEPTS.md", "what/GLOSSARY.md", "how/decision_records/INDEX.md", "how/process/DECISION_PIPELINE.md"]
tech-stack:
  added: []
  patterns: ["Concept-as-EDR pipeline", "Gate Validation tables", "Unified comptime model"]
key-files:
  created:
    - what/concepts/AST_MACROS.md
    - what/concepts/COMPILER_AS_STATIC_ANALYZER.md
    - what/concepts/COMPILE_TIME_EXECUTION.md
    - what/concepts/COMPOSABLE_COLLECTION_OPS.md
    - what/concepts/CONCURRENCY_MODEL.md
    - how/decision_records/architecture/EDR-029-ast-macros.md
    - how/decision_records/architecture/EDR-030-compiler-as-static-analyzer.md
    - how/decision_records/architecture/EDR-031-compile-time-execution.md
    - how/decision_records/architecture/EDR-032-composable-collection-ops.md
    - how/decision_records/architecture/EDR-033-concurrency-model.md
  modified:
    - how/process/DECISION_PIPELINE.md
    - how/decision_records/INDEX.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
decisions:
  - "AST_MACROS: Macros as functions on typed AST nodes, building on unified comptime — Language classification"
  - "COMPILER_AS_STATIC_ANALYZER: Built-in static analysis as part of compiler pipeline — Platform classification"
  - "COMPILE_TIME_EXECUTION: Unified comptime model (Zig-inspired) coexisting with declared generics — Platform classification"
  - "COMPOSABLE_COLLECTION_OPS: Chained combinator API on iterators — StdLib classification"
  - "CONCURRENCY_MODEL: Delegate-based concurrency with act modifier, message passing, no shared-state threads — Language classification"
metrics:
  duration: "~30 min"
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 3: Essential Remaining Core — Metaprogramming, Comptime, Concurrency

## One-liner

Processed **5 essential-tier core concepts** (AST_MACROS, COMPILER_AS_STATIC_ANALYZER, COMPILE_TIME_EXECUTION, COMPOSABLE_COLLECTION_OPS, CONCURRENCY_MODEL) through the full acceptance pipeline: Decision Pipeline, D-03 classification, 7 Validation Gates, Concept Design Review, EDR acceptance, concept docs, and registry updates.

## Files Created

### EDRs (5 files)

| EDR | Concept | Classification | Key Decision |
|-----|---------|---------------|--------------|
| EDR-029 | AST Macros | Language | Macros as functions on typed AST nodes. `@macro` annotation. Builds on comptime. Hygienic. |
| EDR-030 | Compiler as Static Analyzer | Platform | Compiler provides built-in static analysis API — always available, no external linters needed. |
| EDR-031 | Compile-Time Execution | Platform | Unified comptime model (Zig-inspired). Same code runs at compile time or runtime. Coexists with declared generics (EDR-024). |
| EDR-032 | Composable Collection Ops | Standard Library | Chained combinator API (`.map()`, `.filter()`, `.reduce()`) on ITERATOR_PROTOCOL. Loop fusion as Implementation Strategy concern. |
| EDR-033 | Concurrency Model | Language | Delegate-based model. `act` modifier for isolated execution. Message passing. No shared-state threads by default. |

### Concept Specification Documents (5 files)

| File | Source EDR | Dependencies |
|------|-----------|--------------|
| `what/concepts/AST_MACROS.md` | EDR-029 | COMPILE_TIME_EXECUTION (EDR-031) |
| `what/concepts/COMPILER_AS_STATIC_ANALYZER.md` | EDR-030 | COMPILE_TIME_EXECUTION (EDR-031) |
| `what/concepts/COMPILE_TIME_EXECUTION.md` | EDR-031 | GENERICS (EDR-024) cross-ref |
| `what/concepts/COMPOSABLE_COLLECTION_OPS.md` | EDR-032 | ITERATOR_PROTOCOL (EDR-022) |
| `what/concepts/CONCURRENCY_MODEL.md` | EDR-033 | ERROR_HANDLING (EDR-020), TRAITS (EDR-019) |

## Files Modified

| File | Change |
|------|--------|
| `how/process/DECISION_PIPELINE.md` | Added Essential Core — Wave 2 subsection with Pipeline Q&A for all 5 concepts |
| `how/decision_records/INDEX.md` | Added EDR-029 through EDR-033 to All Records and Architecture By Category |
| `what/CORE_CONCEPTS.md` | Added registry entries for all 5 concepts with classifications and EDR references |
| `what/GLOSSARY.md` | Added terms: AST Macro, Comptime, Concurrency Model, Delegate (execution), Message Passing |

## Pipeline Application Results

All five concepts classified per D-03 criteria:

| Concept | D-03 Classification | Rationale |
|---------|---------------------|-----------|
| AST_MACROS | Language | Macros operate on typed AST nodes at compile time — requires compiler-level AST type exposure |
| COMPILER_AS_STATIC_ANALYZER | Platform | Compiler provides static analysis API — compiler IS the analyzer, no separate tool |
| COMPILE_TIME_EXECUTION | Platform | Comptime is a compiler-level execution mode — same semantics evaluated at a different phase |
| COMPOSABLE_COLLECTION_OPS | StdLib | `.map()`, `.filter()`, `.reduce()` are compositions of ITERATOR_PROTOCOL — no new semantics |
| CONCURRENCY_MODEL | Language | Delegate-based concurrency with compiler-guaranteed isolation — data-race freedom requires compiler enforcement |

## Gate Validation Summary

All seven gates passed for all five EDRs:

| Gate | AST_MACROS | COMPILER | COMPTIME | COLLECTION_OPS | CONCURRENCY |
|------|------------|----------|----------|----------------|-------------|
| USER_VALUE | Pass | Pass | Pass | Pass | Pass |
| LOGICAL_CONSISTENCY | Pass | Pass | Pass | Pass | Pass |
| CONCEPTUAL_SIMPLICITY | Pass | Pass | Pass | Pass | Pass |
| ARCHITECTURAL_INTEGRITY | Pass | Pass | Pass | Pass | Pass |
| IMPLEMENTATION_INDEPENDENCE | Pass | Pass | Pass | Pass | Pass |
| LONG_TERM_MAINTAINABILITY | Pass | Pass | Pass | Pass | Pass |
| LLM_GENERABILITY | Pass | Pass | Pass | Pass | Pass |

## Key Cross-References

- COMPILE_TIME_EXECUTION cross-references GENERICS (EDR-024, Plan 04-02): comptime coexists with declared generics rather than replacing them. Generics are type-safe at definition site; comptime is evaluated at instantiation.
- CONCURRENCY_MODEL cross-references CONCURRENCY (Plan 04-06, important tier): CONCURRENCY_MODEL defines the model; CONCURRENCY defines concrete StdLib utilities.
- AST_MACROS depends on COMPILE_TIME_EXECUTION: macros are comptime functions with `@macro` annotation.
- COMPOSABLE_COLLECTION_OPS depends on ITERATOR_PROTOCOL (EDR-022, Plan 04-01): operations chain on the iterator trait.

## Deviations from Plan

None — plan executed exactly as written with all required artifacts created.

## Known Stubs

None — all concept specs are specification-quality (no placeholders, no DRAFT markers, no empty sections).

## Threat Flags

None — all created files are documentation-only, no security-relevant surface introduced.

## Self-Check: PASSED

All 5 concept docs verified present in `what/concepts/`. All 5 EDRs verified present in `how/decision_records/architecture/`. All registry files (INDEX.md, CORE_CONCEPTS.md, GLOSSARY.md) verified updated with Plan 03 entries.
