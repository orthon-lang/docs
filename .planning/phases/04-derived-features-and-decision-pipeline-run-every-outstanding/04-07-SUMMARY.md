---
phase: "04"
plan: "07"
subsystem: "derived-features"
tags: [decision-pipeline, important-tier, contracts, delegation, extension-functions, gradual-typing, smart-cast, copy-on-write, properties, slots, span, sorting, date-time, persistent-data-structures, derive-serialization, command-pattern, context-limited-modules, declarative-constructs, declaration-by-assignment]
requires: [04-01, 04-02, 04-03, 04-04, 04-05, 04-06]
provides: [EDR-056, EDR-057, EDR-058, EDR-059, EDR-060, EDR-061, EDR-062, EDR-063, EDR-064, EDR-065, EDR-066, EDR-067, EDR-068, EDR-069, EDR-070, EDR-071, EDR-072, EDR-073, EDR-074]
affects: [what/CORE_CONCEPTS.md, what/GLOSSARY.md, how/decision_records/INDEX.md, how/process/DECISION_PIPELINE.md]
tech-stack:
  added: []
  patterns: [Concept-as-EDR pipeline (abbreviated Design Review for low-complexity StdLib concepts)]
key-files:
  created:
    - what/concepts/CONTRACTS.md
    - what/concepts/DELEGATION.md
    - what/concepts/EXTENSION_FUNCTIONS.md
    - what/concepts/GRADUAL_TYPING.md
    - what/concepts/SMART_CAST.md
    - what/concepts/COPY_ON_WRITE.md
    - what/concepts/PROPERTIES.md
    - what/concepts/SLOTS.md
    - what/concepts/SPAN.md
    - what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md
    - what/concepts/SORTING.md
    - what/concepts/DECLARATIVE_MULTI_KEY_SORT.md
    - what/concepts/IMMUTABLE_DATE_TIME.md
    - what/concepts/PERSISTENT_DATA_STRUCTURES.md
    - what/concepts/DERIVE_SERIALIZATION.md
    - what/concepts/COMMAND_PATTERN_VIA_DELEGATE.md
    - what/concepts/CONTEXT_LIMITED_MODULES.md
    - what/concepts/DECLARATIVE_CONSTRUCTS.md
    - what/concepts/DECLARATION_BY_ASSIGNMENT.md
    - how/decision_records/architecture/EDR-056-contracts.md
    - how/decision_records/architecture/EDR-057-delegation.md
    - how/decision_records/architecture/EDR-058-extension-functions.md
    - how/decision_records/architecture/EDR-059-gradual-typing.md
    - how/decision_records/architecture/EDR-060-smart-cast.md
    - how/decision_records/architecture/EDR-061-copy-on-write.md
    - how/decision_records/architecture/EDR-062-properties.md
    - how/decision_records/architecture/EDR-063-slots.md
    - how/decision_records/architecture/EDR-064-span.md
    - how/decision_records/architecture/EDR-065-named-and-optional-parameters.md
    - how/decision_records/architecture/EDR-066-sorting.md
    - how/decision_records/architecture/EDR-067-declarative-multi-key-sort.md
    - how/decision_records/architecture/EDR-068-immutable-date-time.md
    - how/decision_records/architecture/EDR-069-persistent-data-structures.md
    - how/decision_records/architecture/EDR-070-derive-serialization.md
    - how/decision_records/architecture/EDR-071-command-pattern-via-delegate.md
    - how/decision_records/architecture/EDR-072-context-limited-modules.md
    - how/decision_records/architecture/EDR-073-declarative-constructs.md
    - how/decision_records/architecture/EDR-074-declaration-by-assignment.md
  modified:
    - how/decision_records/INDEX.md
    - how/process/DECISION_PIPELINE.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
decisions:
  - "CONTRACTS: design-by-contract via compiler-enforced pre/postconditions — Language classification"
  - "DELEGATION: composition via delegation pattern (reuses delegate execution concept) — StdLib classification"
  - "EXTENSION_FUNCTIONS: method-call syntax on external types — Language classification"
  - "GRADUAL_TYPING: optional type annotations with selective static checking — Language classification"
  - "SMART_CAST: flow-sensitive type narrowing after type check — Language classification"
  - "COPY_ON_WRITE: memory optimisation for value semantics — StdLib / Implementation Strategy concern"
  - "PROPERTIES: getter/setter sugar over attribute access — StdLib classification"
  - "SLOTS: compact fixed-field storage annotation — Language classification"
  - "SPAN: safe non-owning memory view/slice — Language classification"
  - "NAMED_AND_OPTIONAL_PARAMETERS: call ergonomics via existing mechanisms — StdLib classification"
  - "SORTING: stable sort by default with explicit unstable variant — StdLib classification"
  - "DECLARATIVE_MULTI_KEY_SORT: syntactic sugar over sorting — StdLib classification"
  - "IMMUTABLE_DATE_TIME: value-semantics date/time types — StdLib classification"
  - "PERSISTENT_DATA_STRUCTURES: immutable collections with structural sharing — StdLib classification"
  - "DERIVE_SERIALIZATION: automatic serialization via trait derivation — StdLib classification"
  - "COMMAND_PATTERN_VIA_DELEGATE: language-level delegate eliminates command pattern need — Language classification"
  - "CONTEXT_LIMITED_MODULES: capability-based module access — Language classification"
  - "DECLARATIVE_CONSTRUCTS: declarative StdLib pattern sugar — StdLib classification"
  - "DECLARATION_BY_ASSIGNMENT: variable introduction via first assignment — Language classification (syntax deferred to Phase 5)"
metrics:
  duration: "~2 hours"
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 7: Important Tier — Remaining Concepts Decision Pipeline Summary

## One-Liner

Processed **19 important-tier concepts** (CONTRACTS, DELEGATION, EXTENSION_FUNCTIONS, GRADUAL_TYPING, SMART_CAST, COPY_ON_WRITE, PROPERTIES, SLOTS, SPAN, NAMED_AND_OPTIONAL_PARAMETERS, SORTING, DECLARATIVE_MULTI_KEY_SORT, IMMUTABLE_DATE_TIME, PERSISTENT_DATA_STRUCTURES, DERIVE_SERIALIZATION, COMMAND_PATTERN_VIA_DELEGATE, CONTEXT_LIMITED_MODULES, DECLARATIVE_CONSTRUCTS, DECLARATION_BY_ASSIGNMENT) through the full acceptance pipeline — producing 19 EDRs (EDR-056 through EDR-074), 19 concept docs, and registry updates. This was the largest single plan in Phase 4; low-complexity StdLib concepts received abbreviated Design Review while still meeting D-02's full treatment requirement.

## Classification Decisions

| Concept | EDR | Classification | Rationale |
|---------|-----|---------------|-----------|
| CONTRACTS | EDR-056 | **Language** | Compiler-enforced pre/postconditions; requires compiler-level verification. |
| DELEGATION | EDR-057 | **StdLib** | Composition via delegation pattern reusing the delegate execution concept. |
| EXTENSION_FUNCTIONS | EDR-058 | **Language** | Method-call syntax on external types requires name-resolution support. |
| GRADUAL_TYPING | EDR-059 | **Language** | Optional annotations with selective static checking; type-system concern. |
| SMART_CAST | EDR-060 | **Language** | Flow-sensitive type narrowing in the type checker. |
| COPY_ON_WRITE | EDR-061 | **StdLib** | Memory optimisation for value semantics; Implementation Strategy concern. |
| PROPERTIES | EDR-062 | **StdLib** | Getter/setter sugar over attribute access; no new semantics. |
| SLOTS | EDR-063 | **Language** | Compact fixed-field storage annotation affects data layout semantics. |
| SPAN | EDR-064 | **Language** | Safe non-owning memory view; memory-safety semantics. |
| NAMED_AND_OPTIONAL_PARAMETERS | EDR-065 | **StdLib** | Call ergonomics expressible via existing mechanisms. |
| SORTING | EDR-066 | **StdLib** | Sort algorithms, not a language feature. |
| DECLARATIVE_MULTI_KEY_SORT | EDR-067 | **StdLib** | Syntactic sugar over the sorting API. |
| IMMUTABLE_DATE_TIME | EDR-068 | **StdLib** | Value-semantics date/time types. |
| PERSISTENT_DATA_STRUCTURES | EDR-069 | **StdLib** | Immutable collections with structural sharing. |
| DERIVE_SERIALIZATION | EDR-070 | **StdLib** | Automatic serialization via trait derivation. |
| COMMAND_PATTERN_VIA_DELEGATE | EDR-071 | **Language** | Language-level delegate makes the command pattern unnecessary. |
| CONTEXT_LIMITED_MODULES | EDR-072 | **Language** | Capability-based module access requires compiler-level checks. |
| DECLARATIVE_CONSTRUCTS | EDR-073 | **StdLib** | Declarative pattern sugar over existing constructs. |
| DECLARATION_BY_ASSIGNMENT | EDR-074 | **Language** | Variable introduction via first assignment; concrete syntax deferred to Phase 5. |

## Key Decisions Made

1. **Delegation reuses the delegate execution concept** — `DELEGATION` (pattern) builds on the `DELEGATE` execution primitive from Plan 04-03; no new runtime semantics.
2. **Contracts may constrain traits** — `CONTRACTS` is a Language construct that can be layered on `TRAITS` (Plan 04-01); design-by-contract via pre/postconditions.
3. **Smart cast distinct from pattern matching** — `SMART_CAST` is flow-sensitive narrowing in the type checker, complementary to (not subsumed by) `PATTERN_MATCHING`.
4. **COPY_ON_WRITE is StdLib/Strategy** — an implementation concern for value semantics, not a language feature.
5. **DECLARATION_BY_ASSIGNMENT syntax deferred to Phase 5** — the semantic decision is made (variable introduction by first assignment) but the concrete `let x = ...` syntax belongs to the Syntax Design phase.
6. **Abbreviated Design Review for low-complexity StdLib concepts** — SORTING, SLOTS, IMMUTABLE_DATE_TIME, NAMED_AND_OPTIONAL_PARAMETERS, etc. received Pipeline + Gates + EDR treatment with abbreviated Design Review, per D-02's volume allowance.

## Deviations from Plan

None — plan executed exactly as specified. All 19 concepts processed with the Decision Pipeline, classified per D-03, validated against the 7 gates, and documented with concept files + EDRs. Registry updates applied to CORE_CONCEPTS.md, GLOSSARY.md, DECISION_PIPELINE.md, and INDEX.md.

## Files Created/Modified

- **19 concept docs** in `what/concepts/` — one per accepted concept
- **19 EDRs** in `how/decision_records/architecture/` — EDR-056 through EDR-074
- **Updated:** `how/decision_records/INDEX.md`, `how/process/DECISION_PIPELINE.md`, `what/CORE_CONCEPTS.md`, `what/GLOSSARY.md`

## Self-Check

- [x] All 19 concepts accepted with EDRs
- [x] All 19 concept files present in `what/concepts/`
- [x] Classifications recorded in LIBRARY_BOUNDARY.md (9 Language + 10 StdLib)
- [x] INDEX.md updated with EDR-056 through EDR-074
