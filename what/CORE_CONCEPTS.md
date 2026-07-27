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

---

## Wave 2 — Essential Core (Phase 4)

### ERROR_UNION

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-023](../how/decision_records/architecture/EDR-023-error-union.md) |
| **Specification** | [`concepts/ERROR_UNION.md`](concepts/ERROR_UNION.md) |
| **Classification** | Language (D-03) |
| **Summary** | Zig-style `!T` tag-only error union. Inferred error sets computed by compiler from every fallible call in function body. Structural widening (subset → superset coercion, no explicit conversion). Coexists with `Result<T, E>` for payload-bearing errors. `anyerror` escape hatch for boundaries. `?` operator shared with `Result` for unified propagation. No `try`/`catch`. |
| **Primitive Decomposition** | `!T` type former → not decomposable — new syntax; error tag literal → `literal` (unit-like tag); error set inference → compiler-level call-graph analysis beyond primitive composition; structural widening → type-system coercion rule. The inference and widening semantics add compiler-level behaviour beyond primitive composition. |

### GENERICS

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-024](../how/decision_records/architecture/EDR-024-generics.md) |
| **Specification** | [`concepts/GENERICS.md`](concepts/GENERICS.md) |
| **Classification** | Language (D-03) |
| **Summary** | Trait-bounded parametric polymorphism. `where T: TraitA + TraitB` for complex bounds, `[T: Trait]` shorthand for simple. Static dispatch by default (monomorphisation). Dynamic dispatch via `dyn Trait` (opt-in). Invariant by default; covariance/contravariance declared via trait method signatures. No HKT in v0.1. No negative bounds in v0.1. Cross-ref COMPILE_TIME_EXECUTION (Plan 04-03). |
| **Primitive Decomposition** | Generic function → `function` + type parameters (new abstraction); monomorphised instantiation → `function` + concrete type substitution (compiler transformation); trait bound → `identifier` (trait name) + type-system constraint. The type-level abstraction, bound resolution, and monomorphisation add compiler-level semantics beyond primitive composition. |

### PATTERN_MATCHING

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-025](../how/decision_records/architecture/EDR-025-pattern-matching.md) |
| **Specification** | [`concepts/PATTERN_MATCHING.md`](concepts/PATTERN_MATCHING.md) |
| **Classification** | Language (D-03) |
| **Summary** | Exhaustive, expression-oriented structural matching. `match value { case pattern => expr }`. Exhaustiveness required (compile-time error). Destructuring of tuples, records, variants. Wildcard `_` catch-all. Guards (`if condition`). Or patterns (`A \| B`). First-match precedence. Consumption by default; borrowing explicit via `&`. Depends on TRAITS (EDR-019) for sealed trait exhaustiveness. |
| **Primitive Decomposition** | `match` expression → `function` (match arms) + `call` (pattern evaluation) + `scope` (arm bodies); destructuring → `pack`/`unpack`; guard → `function` + `call`; wildcard → `identifier` (ignored binding). The exhaustiveness verification, decision tree compilation, and type-variant enumeration add compiler-level semantics beyond primitive composition. |

### PATTERN_MATCHING_DISPATCH

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-026](../how/decision_records/architecture/EDR-026-pattern-matching-dispatch.md) |
| **Specification** | [`concepts/PATTERN_MATCHING_DISPATCH.md`](concepts/PATTERN_MATCHING_DISPATCH.md) |
| **Classification** | Language (D-03) |
| **Summary** | Multimethod dispatch via definition-site `match` declaration form. Pattern matching on function arguments, resolved at call site. Exhaustiveness across argument type combinations. Specificity resolution (most concrete pattern wins; ties are compile-time error). Complements trait dispatch (single-receiver → traits; multi-argument → dispatch). Cross-ref COMMAND_PATTERN_VIA_DELEGATE (Plan 04-07). Depends on PATTERN_MATCHING + TRAITS. |
| **Primitive Decomposition** | `match` declaration form → `function` + `match` (per EDR-025) + `scope`; argument pattern → `identifier` + `pack`/`unpack`; specificity resolution → compiler-level pattern comparison. The dispatch tree generation, specificity analysis, and cross-argument exhaustiveness add compiler-level semantics beyond primitive composition. |

### TYPE_INFERENCE

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-027](../how/decision_records/architecture/EDR-027-type-inference.md) |
| **Specification** | [`concepts/TYPE_INFERENCE.md`](concepts/TYPE_INFERENCE.md) |
| **Classification** | Language (D-03) |
| **Summary** | Local bidirectional inference (top-down + bottom-up within function bodies). Explicit annotations at public API boundaries (parameters and return types). Generic type argument inference at call sites. Turbofish `::<T>` disambiguation. No cross-module inference. Inferred types inspectable via Schema Provider. Defer `: Type` concrete syntax to Phase 5. Depends on EQUALITY (EDR-017) for type unification. |
| **Primitive Decomposition** | Inferred type → compiler-determined, not primitive-expressible; type annotation at API boundary → `identifier` (type name) + `scope`; type unification → `===` applied to type structures per EDR-017. The inference algorithm, constraint solving, and type unification are compiler-level services beyond primitive composition. |

### TYPE_LEVEL_NULL_SAFETY

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md) |
| **Specification** | [`concepts/TYPE_LEVEL_NULL_SAFETY.md`](concepts/TYPE_LEVEL_NULL_SAFETY.md) |
| **Classification** | Language (D-03) |
| **Summary** | Flow-sensitive type narrowing on `Option<T>`. After `match` on `Some(x)`, type narrowed to `T` in matching arm. After `if value != None`, type narrowed to `T` in true branch. Per-variable tracking; resets on reassignment. Conservative — if unsure, stays `Option<T>`. `!` escape hatch for programmer knowledge. `?T` syntactic sugar deferred to Phase 5. Depends on NULL_SAFETY (EDR-018) and PATTERN_MATCHING (EDR-025). |
| **Primitive Decomposition** | Narrowed type → compiler-determined; `match` narrowing → pattern matching (EDR-025) + compiler type tracking; `if` check narrowing → `function` + compiler type tracking across control flow; `!` escape hatch → `function` (unwrapping with panic contract). The flow-sensitive type tracking across control flow edges is a compiler-level analysis beyond primitive composition. |

### AST_MACROS

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-029](../how/decision_records/architecture/EDR-029-ast-macros.md) |
| **Specification** | [`concepts/AST_MACROS.md`](concepts/AST_MACROS.md) |
| **Classification** | Language (D-03) |
| **Summary** | AST macros as functions — ordinary Orthon functions annotated with `@macro`, executing at compile time via comptime. Typed AST node contracts (input type, output type). Hygienic by default with `#` opt-in for unhygienic access. `@derive(Trait)` as declarative sugar. Single-pass expansion — no recursive macros, no phase-ordering bugs. Deterministic, side-effect-free. |
| **Primitive Decomposition** | `@macro` function → `function` + comptime annotation (compiler-recognized); `@derive(Trait)` → compiler-resolved macro registry lookup; AST types → compiler-internal type system (not user-visible beyond macro API). The macro registry, AST type contracts, and expansion ordering add compiler-level semantics beyond primitive composition. |

### COMPILER_AS_STATIC_ANALYZER

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-030](../how/decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) |
| **Specification** | [`concepts/COMPILER_AS_STATIC_ANALYZER.md`](concepts/COMPILER_AS_STATIC_ANALYZER.md) |
| **Classification** | Language (D-03) |
| **Summary** | Compiler IS the static analyzer. Seven progressive verification layers: Syntax, Name Resolution, Type Checking, Ownership & Borrowing, Effect Verification, Exhaustiveness & Completeness, Contract Verification (optional). Layers 1–6 always enabled. `--relaxed` mode for prototyping. LLM-friendly structured diagnostics with machine-readable error codes. No undefined behaviour. |
| **Primitive Decomposition** | Not directly decomposable — the static analyzer is the compiler pipeline itself. Verification layers are meta-operations on the compiler, not expressible via user-visible primitives. |

### COMPILE_TIME_EXECUTION

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-031](../how/decision_records/architecture/EDR-031-compile-time-execution.md) |
| **Specification** | [`concepts/COMPILE_TIME_EXECUTION.md`](concepts/COMPILE_TIME_EXECUTION.md) |
| **Classification** | Language (D-03) |
| **Summary** | Unified comptime model (Zig-inspired). Same semantics, earlier phase. NOT a separate language. `comptime` keyword for parameters and blocks. Comptime IS the generic mechanism — `comptime T: type` replaces `<T>` syntax. Explicit trait bounds on public comptime parameters for LLM discoverability. Reflection via comptime (`@typeInfo`, `@field`, `@hasDecl`). Deterministic, sandboxed. LLM Generability Gate is critical — documented restrictions. |
| **Primitive Decomposition** | Comptime parameter → `function` parameter + comptime annotation; comptime block → `scope` + comptime annotation; `@typeInfo` → comptime-evaluated `call` to compiler intrinsic; monomorphisation → compiler specialization of `function` + types. The comptime execution phase and evaluation engine add compiler-level semantics beyond primitive composition. |

### COMPOSABLE_COLLECTION_OPS

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-032](../how/decision_records/architecture/EDR-032-composable-collection-ops.md) |
| **Specification** | [`concepts/COMPOSABLE_COLLECTION_OPS.md`](concepts/COMPOSABLE_COLLECTION_OPS.md) |
| **Classification** | **StdLib** (D-03) — all operations are compositions of `Iterator[T]` protocol methods. No new language semantics required. |
| **Summary** | Declarative collection operations built on ITERATOR_PROTOCOL: map, filter, reduce, fold, find, any, all, count, collect, take, skip, chain, zip, enumerate. Lazy by default. Materialisation explicit (`.collect()`, `.to_list()`). Loop fusion is Implementation Strategy concern, not language semantics. No comprehension syntax in v0.1. |
| **Primitive Decomposition** | Each combinator (`map`, `filter`, `fold`, etc.) → `function` implementation on `Iterator[T]` trait + `call` to `next()` + `scope` + `pack`/`unpack` for result construction + `function` (closure parameter). Fully expressible via primitive composition — no new compiler semantics. |

### CONCURRENCY_MODEL

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-033](../how/decision_records/architecture/EDR-033-concurrency-model.md) |
| **Specification** | [`concepts/CONCURRENCY_MODEL.md`](concepts/CONCURRENCY_MODEL.md) |
| **Classification** | Language (D-03) |
| **Summary** | Delegate-based concurrency model. `act` modifier for concurrent type declarations. `delegate` keyword creates isolated execution contexts. `<-` message operator for asynchronous communication. No shared-state threads — all concurrency through message passing. Explicit ownership transfer (`$`) across delegate boundaries. Error propagation via `Result<T,E>`. Trait dispatch on delegates. Implementation-independent — no dependency on specific threading/async runtime. Cross-ref with ERROR_HANDLING (EDR-020), TRAITS (EDR-019), and CONCURRENCY (Plan 04-06). |
| **Primitive Decomposition** | `act` modifier → type declaration modifier (compiler-enforced isolation semantics); `delegate` → `reference` + isolated `scope` + message queue; `<-` operator → compiler-recognized message-send syntax; ownership transfer (`$`) → existing `reference` + ownership semantics across boundaries. The isolation guarantee, message ordering, and single-threaded processing per delegate add compiler-level semantics beyond primitive composition. |
