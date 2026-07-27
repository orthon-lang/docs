# Core Concepts

> **📋 REGISTRY — Accepted Orthon Concepts.**
> This document lists concepts that have passed the Concept Design Review
> and been accepted via EDR (Architecture category). Only crystallized
> Orthon-specific specifications belong here — research and draft analyses
> live in `how/concepts/research/`.
>
> **Status:** 18 concepts accepted through Phase 4. Waves 1–2 (essential
> core: 13 concepts) plus Wave 4 (important tier: 5 concepts) accepted
> via Decision Pipeline. Wave 3 (Policy-classified + borderline concepts)
> processed via Decision Pipeline — see
> [`how/process/DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md)
> § Essential — Policy Level and § Essential — Derived Features (Wave 3).
> Policy concepts (ALLOCATION, REGION_BASED_MEMORY, EXECUTION_PROGRAM)
> are routed to `how/strategies/` area per D-04. Borderline concepts
> (CONTEXT_PARAMETERS, REPRESENTATION_MODIFIERS) resolved as corrections
> to existing documents — see [EDR-037](../how/decision_records/architecture/EDR-037-context-parameters.md)
> and [EDR-038](../how/decision_records/architecture/EDR-038-representation-modifiers.md).
> Wave 5 concepts (ALGEBRAIC_DATA_TYPES through TYPE_LEVEL_COMPUTATION,
> EDR-039–046) registered separately.
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

---

## Wave 4 — Important Tier (Phase 4)

### ALGEBRAIC_DATA_TYPES

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md) |
| **Specification** | [`concepts/ALGEBRAIC_DATA_TYPES.md`](concepts/ALGEBRAIC_DATA_TYPES.md) |
| **Classification** | Language (D-03) |
| **Summary** | Sum types via `type Name = Variant(fields) \| Variant(fields)` syntax. Automatic discriminant generation. Exhaustive pattern matching. Recursive types with termination checking. Generic ADTs. Subsumes dedicated enum construct — payload-free variants (`type Color = Red \| Green \| Blue`) serve the enum use case. Variant fields named by default. `@derive` compatible. Builds on TRAITS (EDR-019) + PATTERN_MATCHING (EDR-025). |
| **Primitive Decomposition** | `type Name = Var(fields) \| Var(fields)` → sealed trait declaration (per TRAITS) + variant constructors; automatic discriminant → compiler-generated tag; exhaustiveness → PATTERN_MATCHING (EDR-025) sealed variant checking; recursive termination → compiler analysis beyond primitive composition. |

### COLLECTION_LITERAL_SYNTAX

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-041](../how/decision_records/architecture/EDR-041-collection-literal-syntax.md) |
| **Specification** | [`concepts/COLLECTION_LITERAL_SYNTAX.md`](concepts/COLLECTION_LITERAL_SYNTAX.md) |
| **Classification** | StdLib (D-03) |
| **Summary** | Collection literals as syntactic sugar for StdLib constructors. `[1, 2, 3]` desugars to `List(1, 2, 3)`. Immutable by default; `mut` qualifier for mutable variants. Concrete syntax deferred to Phase 5 (candidates: `[]` lists, `{}` maps, `{}` sets). No arbitrary size limits. |
| **Primitive Decomposition** | Each element → `literal` or `identifier`; literal as whole → `call` to collection constructor. Fully expressible via primitive composition — no new compiler semantics. |

### DATACLASSES

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-042](../how/decision_records/architecture/EDR-042-dataclasses.md) |
| **Specification** | [`concepts/DATACLASSES.md`](concepts/DATACLASSES.md) |
| **Classification** | StdLib (D-03) |
| **Summary** | Dataclass pattern via existing `@derive` mechanism (EDR-029). `@derive(init, eq, repr, hash)` generates constructor, structural equality, string representation, and hash. No dedicated keyword. Immutable by default. `with` expression for copy-with-modify (compiler intrinsic). StdLib provides derive implementations registered in the macro registry. |
| **Primitive Decomposition** | Each derive target (`init`, `eq`, `repr`, `hash`) → `@macro` function invocation per EDR-029; `with` expression → field-by-field copy + selective reassignment (compiler intrinsic beyond primitive composition). |

### LITERAL_TYPES

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-043](../how/decision_records/architecture/EDR-043-literal-types.md) |
| **Specification** | [`concepts/LITERAL_TYPES.md`](concepts/LITERAL_TYPES.md) |
| **Classification** | Language (D-03) |
| **Summary** | Literal values as singleton types. `let x = "GET"` → type `"GET"`; `var y = "GET"` → type `String` (widened). One explicit widening rule. Primitive scalars only (String, Int, Float, Bool). Composes with UNION_INTERSECTION_TYPES (EDR-045) for closed sets. Input to TYPE_LEVEL_COMPUTATION (EDR-046) `KeyOf<T>`. |
| **Primitive Decomposition** | Singleton type → compiler-determined, not primitive-expressible; widening → type-system coercion rule (`let` preserves, `var` widens); narrowing → pattern matching (EDR-025) + compiler type tracking. The singleton type tracking adds compiler-level semantics beyond primitive composition. |

### STRUCTURAL_TYPING

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-044](../how/decision_records/architecture/EDR-044-structural-typing.md) |
| **Specification** | [`concepts/STRUCTURAL_TYPING.md`](concepts/STRUCTURAL_TYPING.md) |
| **Classification** | Language (D-03) |
| **Summary** | Structural trait satisfaction as opt-in via `structural` keyword on trait declaration. Nominal-by-default (explicit `impl` required for most traits). Explicit `impl` overrides structural matching. Static dispatch by default. `@derive` generates explicit `impl` blocks (priority over structural). Ambiguity detection for conflicting structural matches. Builds on TRAITS (EDR-019). |
| **Primitive Decomposition** | `structural` trait keyword → trait declaration modifier (compiler-recognized); structural matching → method signature comparison (type-system operation); priority rule → explicit `impl` > structural match (compiler resolution). The structural matching and conflict resolution add compiler-level semantics beyond primitive composition. |

### UNION_INTERSECTION_TYPES

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-045](../how/decision_records/architecture/EDR-045-union-intersection-types.md) |
| **Specification** | [`concepts/UNION_INTERSECTION_TYPES.md`](concepts/UNION_INTERSECTION_TYPES.md) |
| **Classification** | Language (D-03) |
| **Summary** | Structural union types via `A | B` combinator. Untagged — no discriminant at runtime. Narrowing via `match` or `is` checks (flow-sensitive per EDR-028). Named types or literal types only — no anonymous structural shapes. No exhaustiveness guarantee (unlike ADTs). Intersection types NOT accepted for v0.1 (redundant with product types). |
| **Primitive Decomposition** | `A | B` type former → new syntax + type-system combinator; narrowing → match (EDR-025) + compiler type tracking (EDR-028); runtime representation → value itself (no tag, no boxing). The union type formation and narrowing semantics add compiler-level semantics beyond primitive composition. |

### TYPE_LEVEL_COMPUTATION

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-046](../how/decision_records/architecture/EDR-046-type-level-computation.md) |
| **Specification** | [`concepts/TYPE_LEVEL_COMPUTATION.md`](concepts/TYPE_LEVEL_COMPUTATION.md) |
| **Classification** | Language (D-03) |
| **Summary** | Closed set of 8 non-recursive compiler intrinsics: `KeyOf<T>`, `Pick<T, K>`, `Omit<T, K>`, `Partial<T>`, `Required<T>`, `Record<K, V>`, `Readonly<T>`, `ElementOf<T>`. NO user-extensible type-level language. NO recursion. NO `infer`. Composable (e.g., `Partial<Omit<User, "password">>`). Derive/macro mechanism (EDR-029) is the escape hatch for custom type-level operations. LLM-generable — fixed, documented semantics per intrinsic. |
| **Primitive Decomposition** | Each intrinsic → compiler-evaluated type transformation (not expressible via value-level primitives). Intrinsics are built-in type-system operations — no decomposition path to user-visible primitives. |

---

## Wave 4 — Important Tier (Phase 4)

### ASYNC_AWAIT

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-047](../how/decision_records/architecture/EDR-047-async-await.md) |
| **Specification** | [`concepts/ASYNC_AWAIT.md`](concepts/ASYNC_AWAIT.md) |
| **Classification** | Language (D-03) |
| **Summary** | Async as orthogonal execution modifier on `proc`/`fun`/`new`, not a separate abstraction. Combined with ASYNC_AS_EXPLICIT_MODIFIER. Stackless coroutines with `await` suspension points. Colourless model — `Future<T>` as first-class value; `await` required only when result needed. `spawn` for explicit parallelism, `scope` for structured concurrency. `exclusive` modifier separates suspension from access serialisation. Task cancellation and timeouts. Async lambdas. |
| **Primitive Decomposition** | `async` modifier → compiler-recognized execution modifier on `function`/`scope`; state machine transformation → compiler-level coroutine compilation; `Future<T>` → compiler-managed type + suspension/resumption; `spawn` + `scope` → compiler-enforced lifecycle management. The coroutine transformation and suspension tracking add compiler-level semantics beyond primitive composition. |

### GENERATORS

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-050](../how/decision_records/architecture/EDR-050-generators.md) |
| **Specification** | [`concepts/GENERATORS.md`](concepts/GENERATORS.md) |
| **Classification** | Language (D-03) |
| **Summary** | Bidirectional `yield` — consumer can send values back to generator during iteration. `yield` without expression ≡ `emit` (EDR-021). Generator expressions — parenthesised inline syntax: `(x * x for x in 1..10)`. `yield from` for generator delegation. `BidirectionalGenerator[T, U]` trait. Builds on LAZY_SEQUENCE_GENERATORS (EDR-021). |
| **Primitive Decomposition** | `yield` without expr → equivalent to `emit` (EDR-021); bidirectional `yield expr` → `emit` + consumer receive slot in state machine; generator expression → desugaring to anonymous generator function; `yield from` → `for` loop calling `emit` on sub-generator. The bidirectional state machine slot adds compiler-level semantics beyond EDR-021's one-way `emit`. |

### EMIT_AS_INTERMEDIATE_RESULT

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-052](../how/decision_records/architecture/EDR-052-emit-as-intermediate-result.md) |
| **Specification** | [`concepts/EMIT_AS_INTERMEDIATE_RESULT.md`](concepts/EMIT_AS_INTERMEDIATE_RESULT.md) |
| **Classification** | Language (D-03) — semantic refinement of EDR-021 |
| **Summary** | `emit` serves dual purpose: lazy sequence production (EDR-021) AND intermediate result publication. A function with `emit` + `return` produces intermediate values during computation and a final result accessible via `.final()`. No new syntax — the `emit` keyword already exists. Specification refinement of EDR-021. |
| **Primitive Decomposition** | Same as EDR-021 — `emit` + `return` pattern already supported by generator state machine. `.final()` accessor stores and exposes the return value from the iterator. No new primitive operations. |

### ITERATION_LOOP

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-053](../how/decision_records/architecture/EDR-053-iteration-loop.md) |
| **Specification** | [`concepts/ITERATION_LOOP.md`](concepts/ITERATION_LOOP.md) |
| **Classification** | Language (D-03) |
| **Summary** | `for item in sequence` — the only iteration construct. `while condition` — separate condition-based loop. `loop { }` — infinite loop with optional `break value`. No C-style `for (;;)`. `break` and `continue` in all loop forms. Destructuring in loop variables. Range syntax (`0..n`, `0..=n`). `for` desugars to ITERATOR_PROTOCOL (EDR-022). |
| **Primitive Decomposition** | `for item in sequence` → `IntoIterator::iter()` + `loop` + `match` + `next()` per EDR-022; `while condition` → `loop` + conditional `break`; `loop` → primitive infinite loop; `break`/`continue` → primitive control flow. The `for` desugaring and range-to-iterator conversion add compiler-level semantics beyond primitive composition. |

### UNPACKING

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **EDR** | [EDR-055](../how/decision_records/architecture/EDR-055-unpacking.md) |
| **Specification** | [`concepts/UNPACKING.md`](concepts/UNPACKING.md) |
| **Classification** | Language (D-03) |
| **Summary** | Destructuring assignment matching pack/unpack symmetry (PRIMITIVE_BLOCKS). Tuple destructuring: `let (x, y) = point`. Record destructuring: `let {name, age} = person`. Rename syntax, rest patterns (`..rest`), ignore patterns (`_`), nested destructuring. Function parameter destructuring. `for` loop destructuring. All forms desugar to `pack`/`unpack` primitives — no new runtime semantics. |
| **Primitive Decomposition** | All destructuring forms → `pack`/`unpack` + `identifier` + `call`. Fully expressible via primitive composition — desugaring is a syntactic transformation, not new runtime behaviour. |

