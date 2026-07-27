# Engineering Decision Record Index

> Unified journal of all engineering decisions in the Orthon project.
> Each entry links to the full EDR in its category subdirectory.
>
> **Categories:** Architecture, Technology, Tooling, Process, Delivery,
> Operations, Quality, Security, Governance, Data, AI, Documentation,
> Knowledge, Collaboration, Product.

---

## All Records

| ID | Category | Title | Status | Date | Supersedes |
|----|----------|-------|--------|------|------------|
| EDR-001 | Process | [Engineering Decision Records (EDR) System](process/EDR-001-edr-system.md) | Accepted | 2026-07-18 | TDR-007, TDR-008 |
| EDR-002 | Process | [Gate System](process/EDR-002-gate-system.md) | Accepted | 2026-07-18 | TDR-001 |
| EDR-003 | Process | [Validation Methods](process/EDR-003-validation-methods.md) | Accepted | 2026-07-18 | TDR-002 |
| EDR-004 | Process | [Language Design Checklist](process/EDR-004-language-design-checklist.md) | Accepted | 2026-07-18 | TDR-003 |
| EDR-005 | Quality | [Fitness Functions](quality/EDR-005-fitness-functions.md) | Accepted | 2026-07-18 | TDR-004 |
| EDR-006 | Process | [Implementation Policies & Strategies](process/EDR-006-policies-and-strategies.md) | Accepted | 2026-07-18 | TDR-005 |
| EDR-007 | Process | [Concept Design Review](process/EDR-007-concept-design-review.md) | Accepted | 2026-07-18 | TDR-006 |
| EDR-010 | Architecture | [Layered Architecture](architecture/EDR-010-layered-architecture.md) | Accepted | 2026-07-18 | TDR-009 |
| EDR-011 | Process | [Process Inventory](process/EDR-011-process-inventory.md) | Accepted | 2026-07-18 | TDR-010 |
| EDR-012 | Architecture | [Semantic Dependency Architecture](architecture/EDR-012-semantic-dependency-architecture.md) | Accepted | 2026-07-21 | — |
| EDR-013 | Architecture | [Orthon Semantic Model](architecture/EDR-013-semantic-model.md) | Accepted | 2026-07-27 | — |
| EDR-014 | Architecture | [LLM Generability Gate](architecture/EDR-014-llm-generability-gate.md) | Accepted | 2026-07-19 | — |
| EDR-015 | Process | [Decision Log](process/EDR-015-decision-log.md) | Accepted | 2026-07-27 |
| EDR-016 | Architecture | [Primitive Blocks — minimal orthogonal set of primitive operations](architecture/EDR-016-primitive-blocks.md) | Accepted | 2026-07-27 | — |
| EDR-017 | Architecture | [Equality — Three-Operator Model (Value, Semantic, Identity)](architecture/EDR-017-equality.md) | Accepted | 2026-07-27 | — |
| EDR-018 | Architecture | [Null Safety — Option Type Without Null Sentinel](architecture/EDR-018-null-safety.md) | Accepted | 2026-07-27 | — |
| EDR-019 | Architecture | [Traits — Nominal Trait System for Polymorphic Behaviour](architecture/EDR-019-traits.md) | Accepted | 2026-07-27 | — |
| EDR-020 | Architecture | [Error Handling — Result Type with Explicit Propagation](architecture/EDR-020-error-handling.md) | Accepted | 2026-07-27 | — |
| EDR-021 | Architecture | [Lazy Sequence Generators — `emit` Keyword for Lazy Production](architecture/EDR-021-lazy-sequence-generators.md) | Accepted | 2026-07-27 | — |
| EDR-022 | Architecture | [Iterator Protocol — Trait-Based Lazy Consumption](architecture/EDR-022-iterator-protocol.md) | Accepted | 2026-07-27 | — |
| EDR-023 | Architecture | [Error Union — Zig-Style Tag-Only Error Union](architecture/EDR-023-error-union.md) | Accepted | 2026-07-27 | — |
| EDR-024 | Architecture | [Generics — Trait-Bounded Parametric Polymorphism](architecture/EDR-024-generics.md) | Accepted | 2026-07-27 | — |
| EDR-025 | Architecture | [Pattern Matching — Exhaustive Expression-Oriented Structural Matching](architecture/EDR-025-pattern-matching.md) | Accepted | 2026-07-27 | — |
| EDR-026 | Architecture | [Pattern Matching Dispatch — Multimethod Dispatch via Definition-Site Pattern Matching](architecture/EDR-026-pattern-matching-dispatch.md) | Accepted | 2026-07-27 | — |
| EDR-027 | Architecture | [Type Inference — Local Bidirectional Inference with Explicit API Boundaries](architecture/EDR-027-type-inference.md) | Accepted | 2026-07-27 | — |
| EDR-028 | Architecture | [Type-Level Null Safety — Option&lt;T&gt; Type-Level Tracking with Compiler Narrowing](architecture/EDR-028-type-level-null-safety.md) | Accepted | 2026-07-27 | — |
| EDR-029 | Architecture | [AST Macros as Functions](architecture/EDR-029-ast-macros.md) | Accepted | 2026-07-27 | — |
| EDR-030 | Architecture | [Compiler as Static Analyzer](architecture/EDR-030-compiler-as-static-analyzer.md) | Accepted | 2026-07-27 | — |
| EDR-031 | Architecture | [Unified Compile-Time Execution](architecture/EDR-031-compile-time-execution.md) | Accepted | 2026-07-27 | — |
| EDR-032 | Architecture | [Composable Collection Operations](architecture/EDR-032-composable-collection-ops.md) | Accepted | 2026-07-27 | — |
| EDR-033 | Architecture | [Delegate-Based Concurrency Model](architecture/EDR-033-concurrency-model.md) | Accepted | 2026-07-27 | — |

> **Note:** EDR-008 and EDR-009 are intentionally skipped. TDR-007 and TDR-008
> are superseded by EDR-001, not migrated.

## Gap Registry

| Slot | Status | Rationale |
|------|--------|-----------|
| EDR-008 | Skipped | TDR-007 (ADR System) and TDR-008 (TDR System) are superseded by EDR-001 (the EDR system itself). The gap preserves the original TDR numbering so anyone mapping old TDR records to EDRs can see the correspondence: TDR-007 → superseded, TDR-008 → superseded. |
| EDR-009 | Reserved | Reserved for future use if a decision needs to occupy the TDR-009 position in the mapping. Currently unfilled. |

---

## By Category

### Architecture
| ID | Title | Status | Date |
|----|-------|--------|------|
| EDR-010 | [Layered Architecture](architecture/EDR-010-layered-architecture.md) | Accepted | 2026-07-18 |
| EDR-012 | [Semantic Dependency Architecture](architecture/EDR-012-semantic-dependency-architecture.md) | Accepted | 2026-07-21 |
| EDR-013 | [Orthon Semantic Model](architecture/EDR-013-semantic-model.md) | Accepted | 2026-07-27 |
| EDR-014 | [LLM Generability Gate](architecture/EDR-014-llm-generability-gate.md) | Accepted | 2026-07-19 |
| EDR-016 | [Primitive Blocks](architecture/EDR-016-primitive-blocks.md) | Accepted | 2026-07-27 |
| EDR-017 | [Equality — Three-Operator Model](architecture/EDR-017-equality.md) | Accepted | 2026-07-27 |
| EDR-018 | [Null Safety — Option Type](architecture/EDR-018-null-safety.md) | Accepted | 2026-07-27 |
| EDR-019 | [Traits — Nominal Trait System](architecture/EDR-019-traits.md) | Accepted | 2026-07-27 |
| EDR-020 | [Error Handling — Result Type](architecture/EDR-020-error-handling.md) | Accepted | 2026-07-27 |
| EDR-021 | [Lazy Sequence Generators — `emit` Keyword](architecture/EDR-021-lazy-sequence-generators.md) | Accepted | 2026-07-27 |
| EDR-022 | [Iterator Protocol — Trait-Based Consumption](architecture/EDR-022-iterator-protocol.md) | Accepted | 2026-07-27 |
| EDR-023 | [Error Union — Zig-Style Tag-Only Error Union](architecture/EDR-023-error-union.md) | Accepted | 2026-07-27 |
| EDR-024 | [Generics — Trait-Bounded Parametric Polymorphism](architecture/EDR-024-generics.md) | Accepted | 2026-07-27 |
| EDR-025 | [Pattern Matching — Exhaustive Expression-Oriented Structural Matching](architecture/EDR-025-pattern-matching.md) | Accepted | 2026-07-27 |
| EDR-026 | [Pattern Matching Dispatch — Multimethod Dispatch via Definition-Site Pattern Matching](architecture/EDR-026-pattern-matching-dispatch.md) | Accepted | 2026-07-27 |
| EDR-027 | [Type Inference — Local Bidirectional Inference with Explicit API Boundaries](architecture/EDR-027-type-inference.md) | Accepted | 2026-07-27 |
| EDR-028 | [Type-Level Null Safety — Option&lt;T&gt; Type-Level Tracking with Compiler Narrowing](architecture/EDR-028-type-level-null-safety.md) | Accepted | 2026-07-27 |
| EDR-029 | [AST Macros as Functions](architecture/EDR-029-ast-macros.md) | Accepted | 2026-07-27 |
| EDR-030 | [Compiler as Static Analyzer](architecture/EDR-030-compiler-as-static-analyzer.md) | Accepted | 2026-07-27 |
| EDR-031 | [Unified Compile-Time Execution](architecture/EDR-031-compile-time-execution.md) | Accepted | 2026-07-27 |
| EDR-032 | [Composable Collection Operations](architecture/EDR-032-composable-collection-ops.md) | Accepted | 2026-07-27 |
| EDR-033 | [Delegate-Based Concurrency Model](architecture/EDR-033-concurrency-model.md) | Accepted | 2026-07-27 |

### Process
| ID | Title | Status | Date |
|----|-------|--------|------|
| EDR-001 | [EDR System](process/EDR-001-edr-system.md) | Accepted | 2026-07-18 |
| EDR-002 | [Gate System](process/EDR-002-gate-system.md) | Accepted | 2026-07-18 |
| EDR-003 | [Validation Methods](process/EDR-003-validation-methods.md) | Accepted | 2026-07-18 |
| EDR-004 | [Language Design Checklist](process/EDR-004-language-design-checklist.md) | Accepted | 2026-07-18 |
| EDR-006 | [Policies & Strategies](process/EDR-006-policies-and-strategies.md) | Accepted | 2026-07-18 |
| EDR-007 | [Concept Design Review](process/EDR-007-concept-design-review.md) | Accepted | 2026-07-18 |
| EDR-011 | [Process Inventory](process/EDR-011-process-inventory.md) | Accepted | 2026-07-18 |
| EDR-015 | [Decision Log](process/EDR-015-decision-log.md) | Accepted | 2026-07-27 |

### Quality
| ID | Title | Status | Date |
|----|-------|--------|------|
| EDR-005 | [Fitness Functions](quality/EDR-005-fitness-functions.md) | Accepted | 2026-07-18 |

### Technology, Tooling, Delivery, Operations, Security, Governance, Data, AI, Documentation, Knowledge, Collaboration, Product
*No records yet.*

---

## Status Summary

| Status | Count |
|--------|-------|
| Accepted | 31 |
| Proposed | 0 |
| Deprecated | 0 |
| Superseded | 0 |

---

*Last updated: 2026-07-27*
