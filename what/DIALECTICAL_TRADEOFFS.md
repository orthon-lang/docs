# Dialectical Trade-offs

> Registry of Orthon's position on each axis of language design: the
> competing positions, the trade-off accepted, and the new contradiction
> each committed position creates. The axis tree itself lives in
> [`why/DESIGN_INFLUENCES.md`](../why/DESIGN_INFLUENCES.md); this document
> overlays Orthon on it. Reference index, not specification — canonical
> semantics stay in the linked EDRs and concept docs.

## Section A — Committed Axes

Each row records a position Orthon has committed to (via an EDR or an
accepted concept), the positions it declined, the trade-off it accepted,
and the new contradiction the position opens. A row is complete only when
all five cells are filled; see
[`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md)
§ Acceptance Bar.

| Axis | Orthon position | Competing positions | Trade-off accepted | New contradiction |
|------|-----------------|---------------------|--------------------|-------------------|
| Concurrency — task model | Colourless functions; execution policy is an explicit Execution Context value; the scheduler is an Implementation Strategy, not source semantics ([EDR-085](../how/decision_records/architecture/EDR-085-execution-context-invocation.md), [EDR-016](../how/decision_records/architecture/EDR-016-primitive-blocks.md)) | Coloured `async`/`await` (Python, JS, Rust); goroutines (Go); virtual threads (Java); actor isolation (Erlang, Elixir, Swift); structured concurrency (Kotlin, Swift) | Explicit policy values (`defer`, `delegate`, `spawn`, `fork`) and explicit materialisation (`await`, `take`, `next`, `gather`, `stop`) replace function-colour keywords — more surface syntax than `async`/`await` | Worker-lifecycle resolution moves from the runtime scheduler to the source: unresolved `spawn`/`fork` work cannot be silently abandoned (EDR-085), so bounded group completion (parent-child completion, automatic cancellation) becomes a recurring source-level obligation. Its general form is undecidable, so it re-appears as a policy the programmer re-declares per context |

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
