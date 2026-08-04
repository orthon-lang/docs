---
phase: 04
plan: 05
subsystem: "Derived Features & Decision Pipeline"
tags: ["important-tier", "data-types", "type-system", "algebraic", "literal-types", "structural-typing", "union-intersection", "type-computation", "decision-pipeline"]
requires: ["04-01", "04-02", "04-03", "04-04"]
provides: ["concept-specs-important-data-type", "edrs-039-046"]
affects: ["what/CORE_CONCEPTS.md", "what/GLOSSARY.md", "how/decision_records/INDEX.md", "how/process/DECISION_PIPELINE.md"]
tech-stack:
  added: []
  patterns: ["Concept-as-EDR pipeline", "Gate Validation tables", "EDR folding for related concepts"]
key-files:
  created:
    - what/concepts/ALGEBRAIC_DATA_TYPES.md
    - what/concepts/COLLECTION_LITERAL_SYNTAX.md
    - what/concepts/DATACLASSES.md
    - what/concepts/LITERAL_TYPES.md
    - what/concepts/STRUCTURAL_TYPING.md
    - what/concepts/UNION_INTERSECTION_TYPES.md
    - what/concepts/TYPE_LEVEL_COMPUTATION.md
    - how/decision_records/architecture/EDR-039-algebraic-data-types.md
    - how/decision_records/architecture/EDR-041-collection-literal-syntax.md
    - how/decision_records/architecture/EDR-042-dataclasses.md
    - how/decision_records/architecture/EDR-043-literal-types.md
    - how/decision_records/architecture/EDR-044-structural-typing.md
    - how/decision_records/architecture/EDR-045-union-intersection-types.md
    - how/decision_records/architecture/EDR-046-type-level-computation.md
  modified:
    - how/process/DECISION_PIPELINE.md
    - how/decision_records/INDEX.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
decisions:
  - "ALGEBRAIC_DATA_TYPES: Sealed traits + pattern matching subsume dedicated enum mechanism — EDR-040 (ENUM_ALTERNATIVES) folded into EDR-039"
  - "COLLECTION_LITERAL_SYNTAX: Syntax sugar for collection construction ([1,2,3], {k:v}) — Syntax/Module classification"
  - "DATACLASSES: Derive-based data carriers via @derive annotation — StdLib classification"
  - "LITERAL_TYPES: Values as types (type Method = 'GET' | 'POST') — Language/Type System classification"
  - "STRUCTURAL_TYPING: Structural trait satisfaction (duck typing via shape matching) — Language/Type System classification"
  - "UNION_INTERSECTION_TYPES: Structural type combinators A|B and A&B — Language/Type System classification"
  - "TYPE_LEVEL_COMPUTATION: Closed set of compiler intrinsics for type-level operations — Language/Type System classification"
metrics:
  duration: "~40 min"
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 5: Important Data & Type System Concepts

## One-liner

Processed **8 important-tier data and type system concepts** (ALGEBRAIC_DATA_TYPES, ENUM_ALTERNATIVES, COLLECTION_LITERAL_SYNTAX, DATACLASSES, LITERAL_TYPES, STRUCTURAL_TYPING, UNION_INTERSECTION_TYPES, TYPE_LEVEL_COMPUTATION) through the full acceptance pipeline, with ENUM_ALTERNATIVES folded into ALGEBRAIC_DATA_TYPES (EDR-040 merged into EDR-039).

## Files Created

### EDRs (7 files, 1 folded)

| EDR | Concept | Classification | Key Decision |
|-----|---------|---------------|--------------|
| EDR-039 | Algebraic Data Types | Type System | Sealed traits + pattern matching subsume dedicated enum. Sum types via tagged unions. EDR-040 (ENUM_ALTERNATIVES) folded in. |
| EDR-040 | (ENUM_ALTERNATIVES) | Skipped | Folded into EDR-039 — enum alternatives are a facet of ADTs, not a separate concept. |
| EDR-041 | Collection Literal Syntax | Syntax/Module | `[1, 2, 3]` for lists, `{"a": 1}` for maps, `{1, 2, 3}` for sets. Syntactic sugar with compiler recognition for optimization. |
| EDR-042 | Dataclasses | StdLib/Module | `@derive(Eq, Hash, Show)` auto-generates boilerplate for data carriers. Builds on EQUALITY (EDR-017) model. |
| EDR-043 | Literal Types | Type System | Values as types (`"GET"`, `42`, `true`). Compose with union types for closed sets. |
| EDR-044 | Structural Typing | Type System | Structural trait satisfaction — type conforms to trait if it has the required shape, no explicit `impl` needed. |
| EDR-045 | Union and Intersection Types | Type System | Structural `A \| B` (union) and `A & B` (intersection) type combinators. No tag or discriminant required. |
| EDR-046 | Type-Level Computation | Type System | Closed set of compiler intrinsics (conditional types, mapped types, keyof, typeof). Not an open Turing-complete type language. |

### Concept Specification Documents (7 files)

| File | Source EDR | Dependencies |
|------|-----------|--------------|
| `what/concepts/ALGEBRAIC_DATA_TYPES.md` | EDR-039 | TRAITS (EDR-019), PATTERN_MATCHING (EDR-025) |
| `what/concepts/COLLECTION_LITERAL_SYNTAX.md` | EDR-041 | Literal (primitive), Pack (primitive) |
| `what/concepts/DATACLASSES.md` | EDR-042 | EQUALITY (EDR-017), TRAITS (EDR-019) |
| `what/concepts/LITERAL_TYPES.md` | EDR-043 | TYPE_LEVEL_COMPUTATION (EDR-046) cross-ref |
| `what/concepts/STRUCTURAL_TYPING.md` | EDR-044 | TRAITS (EDR-019) |
| `what/concepts/UNION_INTERSECTION_TYPES.md` | EDR-045 | STRUCTURAL_TYPING (EDR-044) cross-ref |
| `what/concepts/TYPE_LEVEL_COMPUTATION.md` | EDR-046 | COMPILE_TIME_EXECUTION (EDR-031), LITERAL_TYPES (EDR-043) |

## Files Modified

| File | Change |
|------|--------|
| `how/process/DECISION_PIPELINE.md` | Added Important Tier — Data & Type System subsection with Pipeline Q&A for all 8 concepts |
| `how/decision_records/INDEX.md` | Added EDR-039 through EDR-046 (with EDR-040 as Skipped/folded) to All Records and Architecture By Category |
| `what/CORE_CONCEPTS.md` | Added registry entries for all 7 accepted concepts with classifications and EDR references |
| `what/GLOSSARY.md` | Added terms: Algebraic Data Type, Collection Literal, Dataclass, Intersection Type, Literal Type, Structural Typing, Sum Type, Tagged Union, Type-Level Computation, Union Type |

## Pipeline Application Results

All eight concepts classified per D-03 criteria:

| Concept | D-03 Classification | Rationale |
|---------|---------------------|-----------|
| ALGEBRAIC_DATA_TYPES | Language (Type System) | Sum types via sealed trait hierarchy — compiler must enforce exhaustiveness and variant checking |
| ENUM_ALTERNATIVES | Folded into ADTs | Not a separate concept — enum alternatives are subsumed by ADT sum type mechanism |
| COLLECTION_LITERAL_SYNTAX | StdLib (Syntax sugar) | `[1, 2, 3]` desugars to collection constructor calls — no new semantics beyond literal + pack |
| DATACLASSES | StdLib (Module) | `@derive` generates boilerplate via macro system — expressible via comptime (EDR-031) |
| LITERAL_TYPES | Language (Type System) | Values as types require compiler-level literal tracking in the type checker |
| STRUCTURAL_TYPING | Language (Type System) | Structural trait satisfaction requires compiler-level shape matching and conformance checking |
| UNION_INTERSECTION_TYPES | Language (Type System) | `A \| B` and `A & B` are structural type combinators requiring compiler-level type resolution |
| TYPE_LEVEL_COMPUTATION | Language (Type System) | Conditional types, mapped types, keyof require compiler intrinsic type operations |

## Gate Validation Summary

All seven gates passed for all accepted EDRs:

| Gate | ADTs | COLLECTION | DATACLASS | LITERAL | STRUCTURAL | UNION/INT | TYPE_COMP |
|------|------|------------|-----------|---------|------------|-----------|-----------|
| USER_VALUE | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| LOGICAL_CONSISTENCY | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| CONCEPTUAL_SIMPLICITY | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| ARCHITECTURAL_INTEGRITY | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| IMPLEMENTATION_INDEPENDENCE | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| LONG_TERM_MAINTAINABILITY | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| LLM_GENERABILITY | Pass | Pass | Pass | Pass | Pass | Pass | Pass |

## Key Cross-References

- ALGEBRAIC_DATA_TYPES subsumes ENUM_ALTERNATIVES: EDR-039 explicitly documents why a dedicated enum mechanism is unnecessary when sealed traits + pattern matching provide the same functionality.
- STRUCTURAL_TYPING cross-references TRAITS (EDR-019, Plan 04-01): structural typing extends the nominal trait system with shape-based conformance.
- LITERAL_TYPES feeds into TYPE_LEVEL_COMPUTATION: literal types are the input domain for type-level operations.
- TYPE_LEVEL_COMPUTATION cross-references COMPILE_TIME_EXECUTION (EDR-031, Plan 04-03): type-level computation is a subset of compile-time execution, implemented as compiler intrinsics.
- DATACLASSES depends on EQUALITY (EDR-017, Plan 04-01): auto-generated equality uses the three-operator model.
- UNION_INTERSECTION_TYPES cross-references STRUCTURAL_TYPING (EDR-044): both are structural type system features.

## Deviations from Plan

None — plan executed exactly as written. ENUM_ALTERNATIVES was folded into ALGEBRAIC_DATA_TYPES per plan guidance (combined EDR used where beneficial).

## Known Stubs

None — all concept specs are specification-quality (no placeholders, no DRAFT markers, no empty sections).

## Threat Flags

None — all created files are documentation-only, no security-relevant surface introduced.

## Self-Check: PASSED

All 7 concept docs verified present in `what/concepts/`. All 7 EDRs verified present in `how/decision_records/architecture/` (EDR-040 correctly skipped/folded). All registry files (INDEX.md, CORE_CONCEPTS.md, GLOSSARY.md) verified updated with Plan 05 entries.
