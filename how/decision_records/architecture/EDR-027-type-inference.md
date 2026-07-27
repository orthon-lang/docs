# EDR-027: Type Inference — Local Bidirectional Inference with Explicit API Boundaries

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

How does Orthon determine types without sacrificing either readability or compiler rigour? The trade-offs between annotation verbosity and inference reach directly affect programmer ergonomics and code clarity.

Research in `how/concepts/research/essential/TYPE_INFERENCE.md` analysed the inference spectrum:

- **Full annotation** (Java pre-10, Go) — every variable and function signature carries an explicit type. Maximum clarity, maximum verbosity.
- **Global inference** (OCaml, Haskell) — types inferred for all expressions. Maximum conciseness, but errors can be opaque.
- **Local inference** (Rust, Kotlin, C# `var`) — types inferred within function bodies, required at function boundaries. Balanced.
- **Gradual inference** (TypeScript, mypy) — some code typed, some not; inference crosses the boundary.

The design question for Orthon is nuanced: the Manifesto's Explicitness principle suggests annotations at all boundaries, while LLM Readability and programmer ergonomics benefit from inference within function bodies.

Type inference depends on EQUALITY (EDR-017) for type unification — the inference engine must compare types structurally (equality of type structures) when resolving constraints.

### Decision

Adopt **local bidirectional type inference** as Orthon's inference strategy with the following design:

1. **Local inference (function-body only):** Type inference is confined to function bodies. Function parameters and return types require explicit annotations. No cross-module inference — public API surfaces are fully annotated.
2. **Bidirectional inference flow:**
   - **Bottom-up:** The type of an expression is inferred from its subexpressions (e.g., `42` infers as `Int`).
   - **Top-down:** The expected type from context constrains expression inference (e.g., `let x: Float = 42` infers `42` as `Float`).
3. **Generic type argument inference:** At call sites, generic type arguments are inferred from argument types and expected return type. Turbofish-style disambiguation (`::<T>`) is available for ambiguous cases.
4. **Type annotations at public API boundaries:** All public function parameters and return types must be annotated. Private/internal functions may use inference but annotation is recommended for clarity.
5. **Complete within function bodies:** Within a function body, the compiler infers every local expression, variable, and intermediate type. No explicit annotations required inside function bodies (except for generic turbofish disambiguation).
6. **Inferred types are inspectable:** The Schema Provider and Compiler Introspection API expose inferred types. An LLM or IDE can query the type of any expression without running the program.
7. **No ambiguous inference:** If inference produces an ambiguous type, the compiler reports a compile-time error, not an arbitrary default.
8. **Defer concrete `: Type` syntax to Phase 5.** The semantic model of inference is settled here; the concrete syntax for type annotations is determined in Phase 5 (Syntax).

### Consequences

**Positive:**
- Balances explicitness and ergonomics — annotations at API boundaries provide documentation; inference inside functions eliminates noise
- Bidirectional inference handles common patterns (literal types from context, generic instantiation from call site)
- No cross-module inference preserves Module Boundary Policy — consumers never need to run the inference engine to understand module APIs
- Inferred types are tooling-accessible for LLM and IDE consumption

**Negative:**
- Inference failure at function boundaries produces compiler errors that may be hard to diagnose for complex generic signatures
- The bidirectional inference algorithm is more complex than unidirectional inference
- Deferred `: Type` syntax means Phase 5 must settle the concrete annotation syntax — any delay in Phase 5 blocks finalisation of inference ergonomics

### Compliance

1. Every public function parameter and return type must have an explicit annotation (verified by the compiler).
2. Inference must be deterministic — the same expression in the same context must always infer the same type.
3. Ambiguous inference must produce a compile-time error, not a silent default.
4. The Schema Provider must expose inferred types for every expression in the compilation unit.
5. Type inference must use EQUALITY (EDR-017) semantics for type unification — structural comparison of type structures.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Global inference (OCaml/Haskell) | Complex error messages; refactoring can silently change inferred types across large codebases. Violates Explicitness at API surfaces. |
| Full annotation (Go/Java) | Excessive verbosity — type annotations add noise inside function bodies. Orthon values LLM readability which favours conciseness. |
| Gradual inference (TypeScript) | Loses compile-time guarantees in untyped regions. Contradicts Declarative With Static Guarantees. |
| Unidirectional bottom-up only | Cannot handle contextual inference patterns (expected return type constraining generic arguments). Too restrictive. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes `let x = items.map(fn (i) -> i * 2)` without annotating any intermediate type. The compiler infers everything. Annotations are only needed at module boundaries — where they serve as documentation. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Local bidirectional inference is internally consistent: bottom-up inference flows from leaves to root; top-down inference flows from context to leaves; the two meet at each expression node. No circularity — inference is well-founded on the expression tree. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Hypothesis: "Local bidirectional inference provides the best balance of ergonomics and explicitness." Compared to global inference (complex errors), full annotation (high verbosity), and gradual inference (lost guarantees), local bidirectional inference with explicit boundaries is the proven middle path (Rust, Kotlin). |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Type inference is a Core Language type-system service (Level 1). It depends on EQUALITY (EDR-017) for type unification. It is independent of the Syntax layer (Phase 5) — the semantic model is defined before concrete annotation syntax. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Apparent contradiction: bidirectional inference seems tied to a specific algorithm. Separation: the *semantic specification* (local, bidirectional, no cross-module) is strategy-independent; the *inference algorithm* (Hindley-Milner, unification-based, constraint-based) is a Strategy choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "Infer types inside functions; require annotations at API boundaries." Local inference has proven stability in Rust, Kotlin, and C#. The explicit boundary requirement prevents the most common inference-fragility scenarios. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | An LLM generating code within a function body can omit type annotations for local variables — the compiler infers them. At function boundaries, the LLM must supply explicit types — the Schema Provider makes the expected types queryable. The rule is simple: "inside = inferred, boundary = annotated." |

**Gates not applied:** None — all seven gates are required for a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/TYPE_INFERENCE.md` — Full specification
- `what/concepts/EQUALITY.md` (EDR-017) — Type unification via equality semantics
- `what/concepts/TRAITS.md` (EDR-019) — Trait bounds influence inference constraints
- `what/concepts/GENERICS.md` (EDR-024) — Generic type argument inference
- `how/concepts/research/essential/TYPE_INFERENCE.md` — Research analysis
- `how/DESIGN_PRINCIPLES.md` — Explicitness, Simplicity

### Supersedes

*None* — this is a new decision.
