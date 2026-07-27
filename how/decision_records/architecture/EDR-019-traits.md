# EDR-019: Traits — Nominal Trait System for Polymorphic Behaviour

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon needs a mechanism for expressing shared behaviour across different types — polymorphism — without the fragility of class inheritance. The language's design principles demand behaviour separate from data, explicit satisfaction, static dispatch by default, and coherence guarantees.

The research document at `how/concepts/research/essential/TRAITS.md` established the conceptual model, drawing from Rust's trait system (the proven synthesis of Java interfaces and Haskell typeclasses). Five steps of the Concept Design Review were completed:

1. **Step 1 (Idea/Problem):** Orthon needs polymorphic behaviour — different types fulfilling a shared behavioural contract — without class inheritance, fragile base classes, or ad-hoc polymorphism without coherence.
2. **Step 2 (Minimal Solution):** A nominal trait system with explicit `impl` blocks, static dispatch by default, `dyn Trait` for dynamic dispatch, coherence via orphan rule, associated types, default method implementations, and `where` clauses for trait bounds.
3. **Step 3 (Principle Check):** Aligns with Orthogonality (behaviour separate from data, traits compose via `where T: A + B`), Composition Over Inheritance, Minimal Core (traits replace interfaces, abstract classes, and mixins), Explicitness (explicit `impl`, visible `dyn`).
4. **Step 4 (Examples):** All canonical forms documented in `what/concepts/TRAITS.md` with `orthon` code blocks, including trait declaration, `impl`, static/dynamic dispatch, associated types, default methods, Template Method pattern, and blanket implementations.
5. **Step 5 (EDR):** This document.

The Decision Pipeline (processed in Phase 4) classified TRAITS as **Language** per D-03: the nominal trait system adds interface semantics, trait bound resolution, coherence checking, and dispatch selection not decomposable to existing primitives.

---

### Decision

Adopt the **nominal trait system** for Orthon polymorphism:

1. **Trait declaration** via `trait Name { ... }` — method signatures, associated types, default implementations.
2. **Explicit implementation** via `impl Trait for Type { ... }` — a type must explicitly declare trait conformance.
3. **Static dispatch by default** — generic functions with `where T: Trait` bounds are monomorphised.
4. **Dynamic dispatch opt-in** via `dyn Trait` — vtable-based, syntactically visible.
5. **Orphan rule** — an implementation must be in the same module as either the trait or the type.
6. **Associated types** — `type Item` within a trait declaration.
7. **No inheritance** — trait bounds via `where` clauses replace hierarchy.
8. **Default implementations** — methods with bodies in trait declarations.
9. **Blanket implementations** — `impl<T> Trait for T where T: OtherTrait`.

---

### Consequences

**Positive:**
- Behaviour separate from data — clean architectural separation.
- Static dispatch eliminates indirect call overhead and enables inlining.
- Coherence via orphan rule prevents conflicting implementations.
- No class hierarchy — trait bounds via `where` clauses are composition, not inheritance.
- Default methods enable the Template Method pattern without abstract classes.
- Associated types model type families without additional generic parameters.
- Explicit `impl` makes trait satisfaction syntactically visible.
- Matches proven patterns (Rust traits, Haskell typeclasses with orphan rule).

**Negative:**
- Explicit `impl` requires more boilerplate than Go's structural interfaces (mitigated by IDE generation support).
- The orphan rule prevents some cross-cutting patterns (e.g., implementing `Serialize` for a foreign type in a downstream module). Mitigation: the Newtype pattern or explicit wrapper types.
- Static dispatch can increase binary size through monomorphisation (mitigated by `dyn Trait` opt-in).
- No trait inheritance means some relationships require more verbose bounds (mitigated by `where` clause composition).

---

### Compliance

1. The `what/concepts/TRAITS.md` specification defines the canonical semantics.
2. Every implementation must enforce the orphan rule at compile time.
3. Static dispatch (`monomorphisation`) is the default for trait-bounded generics.
4. `dyn Trait` must use vtable-based dispatch — no implicit static dispatch for `dyn`.
5. Trait bounds are expressed via `where` clauses — no trait inheritance.
6. Blanket implementations are subject to the orphan rule.
7. Pattern matching and method dispatch must respect trait bounds — no dynamic dispatch where static bounds are declared.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Full class inheritance (Java/C++) | Fragile base class problem, deep hierarchies, tight coupling. Violates Orthogonality — behaviour and data are coupled. Violates Composition Over Inheritance. |
| Structural interfaces (Go) | Implicit satisfaction can mask accidental interface conformance. Violates Explicitness — a type may satisfy an interface without the type author intending it. Refactoring is harder because adding a method to an interface silently changes which types satisfy it. |
| Typeclasses with unrestricted orphans (Haskell) | Orphan instances create incoherence — two instances of the same typeclass for the same type can be in scope simultaneously, producing unpredictable behaviour. Violates Deterministic Behaviour. |
| Protocols with inheritance (Swift) | Protocol inheritance creates hierarchy, reintroducing the problems that traits avoid. Increases complexity without significant benefit over `where` clause composition. |
| Concepts (C++20) | Syntactically heavy, tightly coupled to the template system. Poor LLM generability — concept definitions are verbose and have subtle rules. |
| No polymorphism (manual dispatch) | Impractical for any non-trivial language. Standard library abstractions (Iterator, Collection, Eq, Ord) require polymorphism. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to write one function that works with many types that share a behaviour." Every programmer encounters this need. The solution directly serves VISION.md's Comfortable by Design and Architectural Integrity pillars — traits provide a principled, non-fragile way to express polymorphism without class hierarchies. Code example from TRAITS.md (`fn print_all[T: Printable](items)`) shows the pain point (duplicated code without polymorphism) and the solution (one generic function). |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | All trait constructs have precise, non-overlapping definitions. Trait = method signatures + associated types + defaults. Implementation = concrete method bodies for a specific type. Dispatch = static (monomorphisation) or dynamic (vtable). Orphan rule = implementation must be in the same module as trait or type. No self-referential paradoxes — a trait does not implement itself (blanket `impl<T> Trait for T where T: OtherTrait` requires an explicit `where` clause). |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The trait system is minimal — removing any component makes polymorphism incomplete." Removing explicit `impl` would require structural typing (Go model, rejected). Removing static dispatch would force vtable overhead on all generic code. Removing the orphan rule would allow incoherent implementations. Removing associated types would force traits to use additional generic parameters. Removing default methods would require separate utility functions for Template Method. Result: all components are necessary. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Traits operate at Level 2 (Language Patterns) in the Semantic Dependency Architecture — they compose Primitive Operations (Level 1: `function` for method declarations, `call` for invocation, `scope` for trait blocks, `identifier` for associated types) into higher-level polymorphism patterns. Trait bounds on generics operate at the type-system level, consistent with EDR-012's Semantic Dependency Architecture. No layer violations — traits do not depend on Standard Library (Level 3). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: traits require dispatch mechanisms (vtable for `dyn`, monomorphisation for static) that seem strategy-dependent, yet trait semantics must be strategy-independent. Separation: the *semantic definition* of a trait is "a behavioural contract with method signatures" — dispatch mechanism is a Strategy choice. Static dispatch can use monomorphisation (Default strategy), generics specialization (High Performance strategy), or compile-time code generation (Embedded strategy). Dynamic dispatch via `dyn Trait` can use vtables, fat pointers, or closure-based dispatch. Behaviour (which method runs for which type) is identical. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "A trait declares what behaviour a type provides — `impl` connects the type to the trait, and `where` clauses require it." Each component is explainable without "and", "but", "except" (the orphan rule is a single sentence: "you must own the trait or the type"). Remove-one-thing test: removing traits would force the language back to class inheritance or structural typing. The model matches established patterns (Rust traits). Evolution path: new traits can be added to the Standard Library; traits can gain more capabilities (specialisation, negative bounds) without changing the core model. No conceptual debt. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: `trait`, `impl`, `where T: Trait`, and `dyn Trait` each have a single, unambiguous meaning. No context-dependent syntax. Schema round-trip: fully expressible in the type system — traits are generic constraints on types, implementations are named blocks. Hallucination surface: low — the pattern matches Rust traits, which LLMs generate reliably. Self-correction: missing trait implementations are statically detectable (type not satisfying required bounds), incorrect dispatch mode is detectable (`dyn Trait` used where `impl Trait` is expected or vice versa). The orphan rule violations are statically detectable. A common LLM mistake (forgetting to implement a required trait) is caught by the compiler. |

**Gates not applied:** None — all seven gates are required for an architecture-level decision establishing a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/TRAITS.md` — Full concept specification
- `what/SEMANTIC_MODEL.md` — Identity, Ownership, and Mutation dimensions (trait methods interact with all three)
- `what/GLOSSARY.md` — Trait, Trait Bound, Orphan Rule
- `how/concepts/research/COMPOSITION_OVER_INHERITANCE.md` — Traits as the composition mechanism
- `how/concepts/research/FUNCTIONS.md` — Functions as the unit of computation; traits provide polymorphic functions
- `how/DESIGN_PRINCIPLES.md` — Orthogonality, Composition Over Inheritance, Minimal Core, Explicitness

### Supersedes

*None* — this is a new decision, not a replacement.
