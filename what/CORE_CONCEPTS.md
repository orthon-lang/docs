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
