# Dialectical Trade-offs

> Registry of Orthon's position on each axis of language design: the
> competing positions, the trade-off accepted, and the new contradiction
> each committed position creates. The axis tree itself lives in
> [`why/DESIGN_INFLUENCES.md`](../why/DESIGN_INFLUENCES.md); this document
> overlays Orthon on it. Reference index, not specification — canonical
> semantics stay in the linked EDRs and concept docs.

## Section A — Committed Axes

Each entry records a position Orthon has committed to (via an EDR or an
accepted concept), the positions it declined (the negated poles), the
trade-off it accepted (the holding mechanism), and the new contradiction
the position opens (the cost). An entry is complete only when all four
fields are filled; see
[`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md)
§ Acceptance Bar. Axes with no committed concept live in Section B.

### 1. Data and Value Model

- **Orthon position:** immutable-by-default value semantics; structural
  equality by default ([EDR-017](../how/decision_records/architecture/EDR-017-equality.md));
  fixed-field slots ([EDR-063](../how/decision_records/architecture/EDR-063-slots.md));
  pack/unpack symmetry ([EDR-096](../how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md));
  1-based indexing as the single norm ([EDR-082](../how/decision_records/architecture/EDR-082-1-based-indexing.md)).
- **Competing positions:** reference identity (Java); mutable-by-default
  (C, Java, Python); 0-based indexing (C, Rust, Python); separate
  pack/spread operators (Rust, Python).
- **Trade-off accepted:** value semantics with no hidden aliasing; the
  cost of copying is pushed to the compiler via copy-on-write
  ([EDR-061](../how/decision_records/architecture/EDR-061-copy-on-write.md)),
  so the programmer never expresses sharing.
- **New contradiction:** "no hidden aliasing" relocates the *is-this-
  shared* decision from the source to a runtime sharing check, so the
  observable cost of mutation depends on a state the source does not
  name. Separately, 1-based indexing negates C/Rust/Python 0-based, and
  its cost is FFI-boundary friction: the `+1` arithmetic the language
  owns must be re-reconciled at every foreign boundary (`0..<N` is
  FFI-only).

### 2. Memory and Lifetime

- **Orthon position:** non-owning, lifetime-tracked views
  ([SPAN, EDR-064](../how/decision_records/architecture/EDR-064-span.md);
  [SLICE, EDR-084](../how/decision_records/architecture/EDR-084-slice.md));
  allocation and deallocation are Policy, not source semantics
  ([EDR-034](../how/decision_records/architecture/EDR-034-allocation.md)).
- **Competing positions:** manual `free` (C); tracing GC (Java, Go);
  ARC (Swift); ownership and borrowing (Rust).
- **Trade-off accepted:** no GC and no full borrow checker — safety via
  Span lifetime tracking plus copy-on-write value semantics, buying
  predictability without ownership ceremony.
- **New contradiction:** lifetime tracking narrower than a borrow checker
  means the general *who-outlives-whom* question is only partially
  decided statically; programs with complex lifetimes fall through to
  runtime/CoW. The axis is not closed — see
  [`notes/ownership-open-questions.md`](../notes/ownership-open-questions.md).

### 3. Type System

- **Orthon position:** gradual typing ([EDR-059](../how/decision_records/architecture/EDR-059-gradual-typing.md));
  local bidirectional inference with explicit API boundaries
  ([EDR-027](../how/decision_records/architecture/EDR-027-type-inference.md));
  nominal-by-default with opt-in structural ([EDR-044](../how/decision_records/architecture/EDR-044-structural-typing.md));
  untagged unions ([EDR-045](../how/decision_records/architecture/EDR-045-union-intersection-types.md));
  a closed set of type-level intrinsics ([EDR-046](../how/decision_records/architecture/EDR-046-type-level-computation.md)).
- **Competing positions:** full static (Java, Rust); dynamic (Python);
  full Hindley-Milner inference (Haskell, OCaml); structural-by-default
  (TypeScript); dependent types (Idris 2, Lean 4).
- **Trade-off accepted:** explicitness and LLM-generability over maximal
  inference — no full HM, no HKT, no dependent types, no recursive
  type-level language; the annotation burden is concentrated at public
  API boundaries.
- **New contradiction:** gradual typing moves the soundness guarantee to
  runtime boundary checks, so "no hidden behaviour" collides with type
  assertions that can fail only when code runs; and boundary-level
  annotation means internal type freedom and external type strictness can
  diverge — the guarantee is only as strong as the boundary.

### 4. Abstraction and Composition

- **Orthon position:** nominal traits, no inheritance
  ([EDR-019](../how/decision_records/architecture/EDR-019-traits.md));
  ADTs ([EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md));
  exhaustive matching ([EDR-025](../how/decision_records/architecture/EDR-025-pattern-matching.md));
  multimethod dispatch ([EDR-026](../how/decision_records/architecture/EDR-026-pattern-matching-dispatch.md));
  extension functions ([EDR-058](../how/decision_records/architecture/EDR-058-extension-functions.md)).
- **Competing positions:** class inheritance (Java, C++); type classes
  (Haskell); protocols (Swift); structural interfaces (Go).
- **Trade-off accepted:** composition over inheritance, with coherence
  and static dispatch by default for determinism and LLM-generability.
- **New contradiction:** exhaustive `match` over a closed ADT *breaks*
  when a new variant is added, so the synthesis relocates the "add a
  variant" cost to every existing `match` site — the Expression Problem's
  two-axis tension, made visible by Orthon's own choice
  (see [LANGUAGE_PROBLEMS.md](LANGUAGE_PROBLEMS.md) § Expression Problem).

### 5. Functions and Control

- **Orthon position:** a single iteration construct (`for`/`while`/`loop`,
  [EDR-053](../how/decision_records/architecture/EDR-053-iteration-loop.md));
  emit-only generators ([EDR-021](../how/decision_records/architecture/EDR-021-lazy-sequence-generators.md),
  [EDR-092](../how/decision_records/architecture/EDR-092-generator-expression-syntax.md));
  trait-based iteration ([EDR-022](../how/decision_records/architecture/EDR-022-iterator-protocol.md)).
- **Competing positions:** C-style `for(;;)`; bidirectional `yield`
  (Python); callback APIs; implicit context (dependency-injection
  frameworks).
- **Trade-off accepted:** one-way `emit` and a single loop form buy
  simplicity and composability; bidirectional yield was withdrawn
  ([EDR-091](../how/decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md)).
- **New contradiction:** withdrawing bidirectional yield leaves
  *two-way communication with a generator* (send in, receive out) without
  a home — the coroutine pattern is demoted to a deferrable hypothesis,
  so coroutine-style control must be re-encoded via Execution Contexts or
  dropped.

### 6. Concurrency and Parallelism *(pilot — first worked example)*

- **Orthon position:** colourless functions; execution policy is an
  explicit Execution Context value; the scheduler is an Implementation
  Strategy, not source semantics
  ([EDR-085](../how/decision_records/architecture/EDR-085-execution-context-invocation.md),
  [EDR-016](../how/decision_records/architecture/EDR-016-primitive-blocks.md)).
- **Competing positions:** coloured `async`/`await` (Python, JS, Rust);
  goroutines (Go); virtual threads (Java); actor isolation (Erlang,
  Elixir, Swift); structured concurrency (Kotlin, Swift).
- **Trade-off accepted:** explicit policy values (`defer`, `delegate`,
  `spawn`, `fork`) and explicit materialisation (`await`, `take`, `next`,
  `gather`, `stop`) replace function-colour keywords — more surface
  syntax than `async`/`await`.
- **New contradiction:** worker-lifecycle resolution moves from the
  runtime scheduler to the source: unresolved `spawn`/`fork` work cannot
  be silently abandoned (EDR-085), so bounded group completion
  (parent-child completion, automatic cancellation) becomes a recurring
  source-level obligation — its general form is undecidable, so it
  re-appears as a policy the programmer re-declares per context.

### 8. Errors and Absence

- **Orthon position:** `Option<T>` instead of null
  ([EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md));
  `Result<T, E>` instead of exceptions ([EDR-020](../how/decision_records/architecture/EDR-020-error-handling.md));
  inferred error unions ([EDR-023](../how/decision_records/architecture/EDR-023-error-union.md));
  contracts ([EDR-056](../how/decision_records/architecture/EDR-056-contracts.md));
  a system/domain error boundary ([EDR-089](../how/decision_records/architecture/EDR-089-system-error-taxonomy.md)).
- **Competing positions:** null (C, Java); exceptions (Java, Python);
  panics (Go); checked exceptions (Java); proof obligations (Dafny,
  SPARK, Vera).
- **Trade-off accepted:** all fallibility declared in signatures; no
  hidden control flow; exhaustive handling enforced — at the price of
  explicit `?` propagation through every fallible call.
- **New contradiction:** no exceptions means every error is threaded
  through every intermediate function, and the `?` operator relocates the
  propagation decision to a marker that must appear at every fallible
  call — a checkable burden that grows with call depth. Contracts
  relocate "what may a condition call" to a closed purity list, and the
  provable/runtime-checked boundary is the undecidability line.

### 9. Modules and Compilation

- **Orthon position:** explicit public API, declared dependencies, and
  effect isolation in the module header
  ([EDR-072](../how/decision_records/architecture/EDR-072-context-limited-modules.md)).
- **Competing positions:** header files (C); packages (Java); crates
  (Rust); functors (OCaml).
- **Trade-off accepted:** the module header is the semantic index —
  dependencies and effects declared once, visible to both humans and
  LLMs.
- **New contradiction:** the header must declare `effects`, but the
  effect model itself is open (axis 7 has no committed concept) — the
  module system commits to a surface before its semantics are defined.
  One concept carries the whole axis; build, packaging, and separate
  compilation are unaddressed.

### 10. Metaprogramming

- **Orthon position:** macros as comptime functions
  ([EDR-029](../how/decision_records/architecture/EDR-029-ast-macros.md));
  unified comptime, comptime as the generics mechanism
  ([EDR-031](../how/decision_records/architecture/EDR-031-compile-time-execution.md)).
- **Competing positions:** Lisp macros; C preprocessor; Rust macros;
  Zig `comptime`; D.
- **Trade-off accepted:** single-pass, hygienic-by-default, no recursive
  macros — determinism and LLM-generability over maximal metaprogramming.
- **New contradiction:** comptime relocates execution into the build, so
  the Halting Problem moves into compilation — "deterministic, sandboxed"
  must be enforced by eval quotas and single-pass limits, which means
  some legitimate recursive or phase-crossing metaprograms are
  unexpressible.

### 14. Tooling and LLM-Facing Surface

- **Orthon position:** the compiler is the static analyzer — seven
  verification layers, LLM-friendly structured diagnostics, no undefined
  behaviour ([EDR-030](../how/decision_records/architecture/EDR-030-compiler-as-static-analyzer.md)).
- **Competing positions:** separate linters and analyzers (C, Python);
  warning-based compilers; IDE-only analysis.
- **Trade-off accepted:** correctness is a compile-time guarantee rather
  than a linter convention.
- **New contradiction:** correctness is defined only as what the seven
  layers can check — any semantic outside them falls outside the
  guarantee; and machine-readable diagnostics constrain what the compiler
  may report, tying the error surface to LLM consumability.

## Section B — Open Axes

Axes where Orthon has no committed position, or only a hypothesis. Each
entry names the witness or classical problem that demands the axis, and
routes the gap to the Decision Pipeline.

| Axis | Demanded by | Orthon status | Next step |
|------|-------------|---------------|-----------|
| Evaluation strategy — strictness / totality | Haskell, Lean 4, Idris 2 (laziness, totality); Halting Problem as a limit | blank — no stated position | Decision Pipeline Q1 |
| Dependent and refinement types | Idris 2, Lean 4, Dafny, Vera; [`REFINEMENT_TYPES.md`](../how/concepts/research/deferrable/REFINEMENT_TYPES.md) | research | Decision Pipeline Q1 |
| Effect tracking — algebraic effects | Koka, Vera, Idris 2; the module `effects` header; [`EFFECT_FOOTPRINT.md`](../how/concepts/research/essential/EFFECT_FOOTPRINT.md) | research | Decision Pipeline Q1 |
| Evolution and compatibility | [`EVOLUTION_MODEL.md`](../how/EVOLUTION_MODEL.md) placeholder; Java (never break) vs Python 3 (epoch) | blank | Decision Pipeline Q1 |
| Heterogeneous execution — GPU / SIMD | Mojo, Bend, Flint vectors | blank | Decision Pipeline Q1 |

## Section C — Concept ↔ Axis Index

Maps the accepted **Language** concepts (the committed syntheses) to the
14 axes. StdLib concepts are composition over primitives, not syntheses,
and are omitted here; the 14-axis tree and its witness positions live in
[`why/DESIGN_INFLUENCES.md`](../why/DESIGN_INFLUENCES.md). Superseded
concepts are marked — they are the syntheses a later decision negated,
which is exactly the raw material for the negation of negation.

### 1. Data and Value Model
- [EQUALITY](../how/decision_records/architecture/EDR-017-equality.md) — three-operator value/semantic/identity model
- [UNPACKING](../how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md) — pack/unpack symmetry
- [SLOTS](../how/decision_records/architecture/EDR-063-slots.md) — fixed fields as the default shape
- [DECLARATION_BY_ASSIGNMENT](../how/decision_records/architecture/EDR-074-declaration-by-assignment.md) — first assignment introduces the binding
- [INDEXING](../how/decision_records/architecture/EDR-082-1-based-indexing.md) — `a[i]` ≡ `a@get(i)`, single 1-based norm
- [RANGE](../how/decision_records/architecture/EDR-083-range.md) — first-class inclusive-inclusive `1..N` value

### 2. Memory and Lifetime
- [SPAN](../how/decision_records/architecture/EDR-064-span.md) — non-owning, lifetime-tracked view
- [SLICE](../how/decision_records/architecture/EDR-084-slice.md) — range applied to a composite, no copy

### 3. Type System
- [TYPE_INFERENCE](../how/decision_records/architecture/EDR-027-type-inference.md) — local bidirectional inference
- [GRADUAL_TYPING](../how/decision_records/architecture/EDR-059-gradual-typing.md) — optional annotations, boundary checks
- [TYPE_LEVEL_NULL_SAFETY](../how/decision_records/architecture/EDR-028-type-level-null-safety.md) — flow narrowing on `Option`
- [SMART_CAST](../how/decision_records/architecture/EDR-060-smart-cast.md) — flow-sensitive narrowing
- [LITERAL_TYPES](../how/decision_records/architecture/EDR-043-literal-types.md) — values as singleton types
- [STRUCTURAL_TYPING](../how/decision_records/architecture/EDR-044-structural-typing.md) — opt-in structural trait satisfaction
- [UNION_INTERSECTION_TYPES](../how/decision_records/architecture/EDR-045-union-intersection-types.md) — untagged structural unions
- [TYPE_LEVEL_COMPUTATION](../how/decision_records/architecture/EDR-046-type-level-computation.md) — closed set of intrinsics
- [CONSTRAINED_TYPES](../how/decision_records/architecture/EDR-080-constrained-types.md) — nominal constraint predicates

### 4. Abstraction and Composition
- [TRAITS](../how/decision_records/architecture/EDR-019-traits.md) — nominal behaviour composition, no inheritance
- [GENERICS](../how/decision_records/architecture/EDR-024-generics.md) — trait-bounded parametric polymorphism
- [ALGEBRAIC_DATA_TYPES](../how/decision_records/architecture/EDR-039-algebraic-data-types.md) — sum types
- [PATTERN_MATCHING](../how/decision_records/architecture/EDR-025-pattern-matching.md) — exhaustive structural matching
- [PATTERN_MATCHING_DISPATCH](../how/decision_records/architecture/EDR-026-pattern-matching-dispatch.md) — multimethod dispatch
- [EXTENSION_FUNCTIONS](../how/decision_records/architecture/EDR-058-extension-functions.md) — method-call syntax on external types
- [COMMAND_PATTERN_VIA_DELEGATE](../how/decision_records/architecture/EDR-071-command-pattern-via-delegate.md) — composition over a dedicated construct

### 5. Functions and Control
- [ITERATOR_PROTOCOL](../how/decision_records/architecture/EDR-022-iterator-protocol.md) — trait-based lazy consumption
- [LAZY_SEQUENCE_GENERATORS](../how/decision_records/architecture/EDR-021-lazy-sequence-generators.md) — `emit` for lazy production
- [GENERATORS](../how/decision_records/architecture/EDR-050-generators.md) — `gen(...)` comprehension
- [EMIT_AS_INTERMEDIATE_RESULT](../how/decision_records/architecture/EDR-052-emit-as-intermediate-result.md) — *(superseded by EDR-095)*
- [ITERATION_LOOP](../how/decision_records/architecture/EDR-053-iteration-loop.md) — `for`/`while`/`loop` only
- [REQUIRE_USING_DEPENDENCY_SLOTS](../how/decision_records/architecture/EDR-081-require-using-dependency-slots.md) — dependency declaration and resolution

### 6. Concurrency and Parallelism
- [CONCURRENCY_MODEL](../how/decision_records/architecture/EDR-033-concurrency-model.md) — *(superseded by EDR-085)*
- [ASYNC_AWAIT](../how/decision_records/architecture/EDR-047-async-await.md) — *(superseded by EDR-085)*
- [EXECUTION_CONTEXT_INVOCATION](../how/decision_records/architecture/EDR-085-execution-context-invocation.md) — unified invocation, colourless functions

### 7. Effects and Purity
- *No committed Language concept.* Only the module `effects` header exists
  (see [CONTEXT_LIMITED_MODULES](../how/decision_records/architecture/EDR-072-context-limited-modules.md));
  the effect-tracking model itself is open — see Section B.

### 8. Errors and Absence
- [NULL_SAFETY](../how/decision_records/architecture/EDR-018-null-safety.md) — `Option<T>`, no null sentinel
- [ERROR_HANDLING](../how/decision_records/architecture/EDR-020-error-handling.md) — `Result<T, E>`, no exceptions
- [ERROR_UNION](../how/decision_records/architecture/EDR-023-error-union.md) — inferred tag-only error sets
- [CONTRACTS](../how/decision_records/architecture/EDR-056-contracts.md) — pre/postconditions and invariants
- [SYSTEM_ERROR](../how/decision_records/architecture/EDR-089-system-error-taxonomy.md) — domain/system boundary by representability

### 9. Modules and Compilation
- [CONTEXT_LIMITED_MODULES](../how/decision_records/architecture/EDR-072-context-limited-modules.md) — explicit API, declared dependencies, effect isolation

### 10. Metaprogramming
- [AST_MACROS](../how/decision_records/architecture/EDR-029-ast-macros.md) — macros as comptime functions
- [COMPILE_TIME_EXECUTION](../how/decision_records/architecture/EDR-031-compile-time-execution.md) — unified comptime model

### 11. Syntax and Surface
- *Tracked in `what/syntax/` and the syntax pipeline, not in this registry.*
  No concept here is a semantic synthesis; syntax records live separately.

### 12. Execution and Deployment Model
- *No Language concept in this registry.* The Execution Program is a
  Policy/manifesto commitment (see
  [`why/MANIFESTO.md`](../why/MANIFESTO.md) "Build once, deploy
  anywhere"), routed to strategies, not crystallised here.

### 13. Evolution and Compatibility
- *Open.* See Section B.

### 14. Tooling and LLM-Facing Surface
- [COMPILER_AS_STATIC_ANALYZER](../how/decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) — the compiler is the verifier; LLM-friendly diagnostics

## Method

See [`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md).
