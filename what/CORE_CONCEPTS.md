# Core Concepts

> **📋 REGISTRY — Accepted Orthon Concepts.**
> This document lists concepts that have passed the Concept Design Review
> and been accepted via EDR (Architecture category). Only crystallized
> Orthon-specific specifications belong here — research and draft analyses
> live in `how/concepts/research/`.
>
> **Status:** No concepts have been accepted yet. Active research and
> design proposals are in [`how/concepts/research/`](../how/concepts/research/).
> See [`how/concepts/README.md`](../how/concepts/README.md) for the concept
> pipeline. Concepts are processed through the Decision Pipeline
> ([`how/process/DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md))
> before entering Concept Design Review.
>
> **Last updated:** 2026-07-27

---

## Wave 1 — Essential Core (Phase 4)

### EQUALITY

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-017](../how/decision_records/architecture/EDR-017-equality.md) |
| **Specification** | [`concepts/EQUALITY.md`](concepts/EQUALITY.md) |
| **Classification** | Language (D-03) |
| **Summary** | Three distinct equality operators: `===` (value/structural), `==` (semantic/user-defined via `Eq` trait), `is` (identity). Structural by default. Transitivity Invariant enforced. NaN deferred to Standard Library. |
| **Primitive Decomposition** | `===` → compiler-generated field-by-field comparison of `pack`/`unpack` structure; `==` → `function` + trait dispatch; `is` → `reference` identity check. |

### NULL_SAFETY

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md) |
| **Specification** | [`concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md) |
| **Classification** | Language (D-03) |
| **Summary** | No `null` sentinel. `Option<T>` sum type with `Some(T)` and `None` variants. Operators: `?.` (elvis/chaining), `??` (fallback), `!` (forced unwrap). Compiler-enforced exhaustive matching on `Option`. |
| **Primitive Decomposition** | `Option<T>` → sum type via `pack`/`unpack` + pattern matching; `?.` → compiler-enforced short-circuit semantics beyond primitive composition; `??` → default-value desugaring; `!` → match + panic. |

### TRAITS

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-019](../how/decision_records/architecture/EDR-019-traits.md) |
| **Specification** | [`concepts/TRAITS.md`](concepts/TRAITS.md) |
| **Classification** | Language (D-03) |
| **Summary** | Nominal trait system. Explicit `impl Trait for Type`. Static dispatch by default (monomorphisation). Dynamic dispatch via `dyn Trait` (vtable, opt-in). Orphan rule for coherence. Associated types. Default method implementations. Blanket implementations. No inheritance — bounds via `where T: A + B`. |
| **Primitive Decomposition** | Trait declaration → `function` signatures + `identifier` + `scope`; `impl` → `function` implementations + `scope`; `dyn Trait` → `reference` + vtable; static dispatch → monomorphisation of generics (`function` + type parameters). Coherence and bound resolution add compiler-level semantics. |

### ERROR_HANDLING

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-020](../how/decision_records/architecture/EDR-020-error-handling.md) |
| **Specification** | [`concepts/ERROR_HANDLING.md`](concepts/ERROR_HANDLING.md) |
| **Classification** | Language (D-03) |
| **Summary** | `Result<T, E>` monadic type with `Ok(T)` and `Error(E)` variants. `?` operator for short-circuit propagation. Combinators: `map`, `and_then`, `or_else`, `unwrap`, `unwrap_or`, `unwrap_or_else`. Pattern matching for exhaustive handling. No exceptions — all fallibility declared in signatures. Error union support for multi-source error composition. |
| **Primitive Decomposition** | `Result<T,E>` → sum type via `pack`/`unpack` + `literal` (Ok/Error variants) + pattern matching; `?` → compiler-enforced short-circuit propagation beyond primitive composition; combinators → `function` + pattern matching. Exhaustiveness checking adds compiler semantics beyond primitive composition. |

### LAZY_SEQUENCE_GENERATORS

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-021](../how/decision_records/architecture/EDR-021-lazy-sequence-generators.md) |
| **Specification** | [`concepts/LAZY_SEQUENCE_GENERATORS.md`](concepts/LAZY_SEQUENCE_GENERATORS.md) |
| **Classification** | Language (D-03) |
| **Summary** | Generator functions with `emit` keyword for lazy sequence production. Three equivalent canonical forms: `emit value`, `return sequence(value)`, `return value ->`. Lazy by default (Phase 3 D-06). Generators implement `Iterator[T]`. State machine compilation — no heap allocation. Infinite sequences valid. Composition without intermediate allocation. |
| **Primitive Decomposition** | Generator function → `function` + state-machine transformation (compiler-generated); `emit value` → iterator protocol `next()` call + suspension/resumption; `return sequence(value)` → iterator completion + value emission; `return value ->` → equivalent desugaring. The state-machine transformation and lazy evaluation semantics add compiler-level semantics beyond primitive composition. |

### ITERATOR_PROTOCOL

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-022](../how/decision_records/architecture/EDR-022-iterator-protocol.md) |
| **Specification** | [`concepts/ITERATOR_PROTOCOL.md`](concepts/ITERATOR_PROTOCOL.md) |
| **Classification** | Language (D-03) |
| **Summary** | Trait-based: `Iterator[T] { fn next(self) -> Option[T] }`. Lazy, single-pass, composable. `for` loop desugars to iterator protocol. `IntoIterator[T]` for collections. Standard combinators as StdLib (map, filter, take, skip, fold, collect, etc.). Range expressions (0..10, 0..=10, step). `@` prefix for protocol method calls per D-07. Single-pass semantics. Zero-cost via monomorphisation. |
| **Primitive Decomposition** | `Iterator[T]` trait → trait declaration (`trait` + `function` + `identifier`) per TRAITS model; `for item in iter` → loop + `call` to `next()` + pattern match on `Option`; range `0..10` → syntax desugaring to `RangeIterator` constructor + `literal`; combinators → `function` implementations on `Iterator[T]` (StdLib). The `for` loop desugaring and range-syntax translation add compiler-level semantics beyond primitive composition. |
