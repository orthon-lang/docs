# EDR-024: Generics — Trait-Bounded Parametric Polymorphism

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon needs parametric polymorphism — the ability to write functions and types that operate uniformly on values of different types while preserving type safety and performance. Research in `how/concepts/research/essential/GENERICS.md` analysed the landscape:

- **Type erasure** (Java, C#) — type parameters erased at runtime; runtime casts, no reification
- **Templates** (C++) — monomorphisation with duck-typed constraints; full performance but code bloat and cryptic errors
- **Trait-bounded generics** (Rust) — monomorphisation with explicit trait bounds; type-safe, performant, clear errors
- **Typeclasses** (Haskell) — ad-hoc polymorphism with implicit dictionary passing; powerful but complex

EDR-019 established Orthon's trait system — nominal traits with explicit `impl`, static dispatch by default, the orphan rule, associated types, and default method implementations. Generics build directly on this foundation: trait bounds on type parameters are the mechanism by which generic code expresses behavioural constraints.

Key design tensions:

1. **Mononomorphisation vs. boxing** — The default dispatch strategy determines code size and compilation time trade-offs.
2. **Variance** — Generic type parameters may be covariant, contravariant, or invariant, affecting subtyping of compound types.
3. **`where` clauses** — The syntax for expressing complex trait bounds must be ergonomic.
4. **Associated type resolution** — How associated types are resolved in generic contexts.
5. **Cross-reference with COMPILE_TIME_EXECUTION (Plan 04-03)** — Compile-time evaluation may enable generic computation over type-level values.

### Decision

Adopt **trait-bounded parametric polymorphism** as Orthon's generics system with the following design:

1. **Trait bounds on type parameters:** Generic functions and types declare constraints via `where T: TraitA + TraitB` syntax. Multiple bounds compose via `+`.
2. **Static dispatch by default (monomorphisation):** Each generic instantiation produces separate compiled code. Dynamic dispatch via `dyn Trait` is the opt-in alternative.
3. **Variance:**
   - Generic type parameters are **invariant by default** — `List[T]` has no subtype relationship to `List[U]` even if `T` is a subtype of `U`.
   - **Covariant** parameters are declared in the trait via `type Output` (return-position associated types).
   - **Contravariant** parameters are declared in the trait via `fn accept(self, item: T)` (argument-position parameters).
   - Explicit variance annotations on type parameter declarations are deferred as an opt-in extension (v0.2+).
4. **`where` clauses:** Complex bounds use `where T: TraitA + TraitB, U: TraitC` syntax after the parameter list. Simple single bounds may use inline `[T: Trait]` shorthand for brevity.
5. **Associated type resolution:** Associated types are resolved during monomorphisation. The compiler substitutes the concrete type's associated type for the trait's declared associated type.
6. **Monomorphisation strategy:** Default is per-instantiation monomorphisation. Compilation-unit-level monomorphisation is an optimisation choice, not a semantic change.
7. **No type erasure:** Generic type information is preserved through compilation — no erasure, no runtime type casts.

### Consequences

**Positive:**
- Full type safety — type errors are caught at compile time, never at runtime
- Optimal performance — monomorphisation eliminates indirect call overhead and enables inlining
- Explicit contracts — trait bounds make behavioural requirements visible in the signature
- Backward compatible with the trait system (EDR-019)

**Negative:**
- Monomorphisation may increase binary size for many instantiations of the same generic
- Compile times increase due to code duplication and trait bound resolution
- Variance rules add complexity — invariant-by-default is safe but may require explicit type annotations in some contexts

### Compliance

1. Every generic function or type must have resolvable trait bounds — no unbounded type parameters.
2. Monomorphisation must not change program semantics — behaviour is identical regardless of instantiation count.
3. The Schema Provider must expose generic type parameters and their bounds for LLM querying.
4. Variance must be deterministically computable from trait method signatures.
5. Cross-reference with COMPILE_TIME_EXECUTION (Plan 04-03) must be maintained — if `comptime` generics are adopted, their interaction with runtime generics must be specified.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Type erasure (Java-style) | Loses type safety — runtime casts, no reification. Contradicts Declarative With Static Guarantees. |
| Duck-typed templates (C++-style) | No explicit constraints — cryptic error messages, accidental interface conformance. Violates Explicitness. |
| Dynamic dispatch only (boxed generics) | Performance penalty for all generic usage. Static dispatch by default is the proven modern pattern (Rust). |
| Higher-kinded types (HKT) | Significant complexity increase; deferrable to v0.2+. Current trait system supports common patterns without HKT. |
| Negative bounds (`where T: !Clone`) | Opens coherence questions; deferred to v0.2+ after real-world experience with positive bounds. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes `fn first[T: Iterator](items: T)` and the compiler generates specialised code for each concrete type. The programmer describes behavioural constraints via trait bounds; the compiler handles code generation. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Trait-bounded generics are internally consistent: a type parameter `T` is constrained by a set of traits; every operation on `T` is justified by those traits. Monomorphisation produces identical semantics for every instantiation. Variance rules are well-defined and computable. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Hypothesis: "Trait bounds + monomorphisation is the simplest generics model that provides type safety and performance." Compared to erasure (unsafe), duck-typed templates (unclear errors), and HKT (overly complex), trait-bounded generics with `where` clauses provide the best balance. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Generics sit at the same level as traits in the Semantic Dependency Architecture. Trait bounds depend on the trait system (Level 1). Monomorphisation is a compiler code-generation strategy (Level 3). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Apparent contradiction: generics seem tied to monomorphisation, which is an implementation strategy. Separation in space: the *semantic model* (type parameterization with trait constraints) is strategy-independent; the *dispatch mechanism* (monomorphisation vs. boxing vs. dictionary passing) is a Strategy choice. The default is monomorphisation; alternatives are permitted as Strategy profiles. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "Generic functions constrained by traits, specialised at compile time." Variance-by-default invariant is safe — conservative and predictable. Monomorphisation has proven production stability in Rust, C++, and Swift. No unresolved complexity debt. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | An LLM can reliably produce generic code with trait bounds: the syntax is `fn name[T: Trait](t: T)` or `fn name[T](t: T) where T: Trait`. Hallucination surface is low — trait names are known entities. The compiler catches bound violations. Associated type resolution is deterministic. |

**Gates not applied:** None — all seven gates are required for a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/GENERICS.md` — Full specification
- `what/concepts/TRAITS.md` (EDR-019) — Trait system foundation
- `what/concepts/TYPE_INFERENCE.md` (EDR-027) — Generic type argument inference
- `how/concepts/research/essential/GENERICS.md` — Research analysis
- `how/strategies/DEFAULT_STRATEGY.md` — Monomorphisation strategy
- `how/DESIGN_PRINCIPLES.md` — Orthogonality, Simplicity, Explicitness

### Supersedes

*None* — this is a new decision.
