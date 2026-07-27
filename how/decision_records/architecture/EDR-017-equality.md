# EDR-017: Equality — Three-Operator Model (Value, Semantic, Identity)

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Equality is one of the most error-prone concepts in programming languages. Languages conflate reference equality with value equality (Java `==` on objects vs. primitives, Python `==` defaulting to identity for custom types), creating a persistent source of bugs. Orthon's design principles demand explicitness, consistency, and orthogonality — applying these to equality requires a clean separation of concerns.

The research document at `how/concepts/research/essential/EQUALITY.md` established the conceptual model. Five steps of the Concept Design Review were completed:

1. **Step 1 (Idea/Problem):** Programmers need predictable, unambiguous equality semantics. The conflation of reference and value equality is the #1 source of bugs in mainstream languages.
2. **Step 2 (Minimal Solution):** Three distinct operators (`===`, `==`, `is`) with three distinct semantics — structural value equality, user-defined semantic equality, and identity equality. Structural by default. Transitivity as an invariant.
3. **Step 3 (Principle Check):** Aligns with Explicitness (different operators have different syntax), Consistency (same operator = same semantics for all types), Data First (structural comparison by default), Named Before Symbolic (each operator has a named function equivalent).
4. **Step 4 (Examples):** All canonical forms documented in `what/concepts/EQUALITY.md` with `orthon` code blocks.
5. **Step 5 (EDR):** This document.

The Decision Pipeline (processed in Phase 4) classified EQUALITY as **Language** per D-03: the `===` operator requires compiler-generated structural comparison not expressible via composition of existing 9 primitives.

---

### Decision

Adopt the **three-operator equality model** for Orthon:

1. **`===`** — Value equality (structural). Compiler-generated field-by-field comparison. The default.
2. **`==`** — Semantic equality (user-defined). Falls back to `===` if not overridden via the `Eq` trait.
3. **`is`** — Identity equality. Two references point to the same object. Only meaningful for reference types.

Enforce the **Transitivity Invariant**: if `a == b` and `b == c` then `a == c` must hold. NaN equality is deferred to the Standard Library to avoid violating transitivity.

---

### Consequences

**Positive:**
- Eliminates the reference-vs-value confusion that plagues Java, Python, and JavaScript.
- Structural-by-default aligns with the Data-first philosophy — data is compared by its structure.
- Three distinct operators make programmer intent syntactically visible at every comparison site.
- Transitivity Invariant prevents the NaN paradox from leaking into the core language.
- Named function equivalents (`eq()`, `equal()`, `same()`) satisfy the Named Before Symbolic principle.

**Negative:**
- Three operators instead of one increases the learning surface for beginners (mitigated by consistent semantics — the same operator always means the same thing).
- NaN deferral means floating-point equality is not expressible via a core-language operator (mitigated by Standard Library `Float.isNaN()` and future `==~` approximate equality).
- `is` is only meaningful for reference types; using it on value types always returns `false`, which may surprise programmers from languages where `is` is the default comparison.

---

### Compliance

1. The `what/concepts/EQUALITY.md` specification defines the canonical semantics.
2. Every implementation must generate structural comparison for `===` on compound types.
3. The `Eq` trait (defining `==`) must be an explicit opt-in — no implicit `==` override.
4. The Transitivity Invariant must be enforced: the compiler may insert debug-mode runtime checks.
5. NaN equality is excluded from the core language operators.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Single `==` with context-dependent semantics (Java model) | Violates Consistency — same operator would mean different things for different types. Rejected at Step 3 (Principle Check). |
| Single `===` for all equality, with `is` for identity (JS model) | Would make `==` unavailable for user-defined semantic equality. Prevents domain-specific equivalence (e.g., `Person` equality by ID). |
| Python `__eq__` model (reference by default, `__eq__` opt-in) | Violates Data First — data should be compared structurally by default, not by identity. Structural-by-default is the safer default. |
| Structural and identity only, no user-defined `==` | Prevents domain modelling where business-logic equivalence differs from structural equality. Too restrictive for practical use. |
| Single keyword `eq`/`same` with explicit mode parameter | Violates Explicitness — the kind of comparison would be determined by a parameter, not syntactically visible at the call site. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I need to compare values, and I don't know which operator does what." Structural-by-default matches programmer intuition (data should compare by its content). Three distinct operators remove the guesswork. Directly serves VISION.md's Comfortable by Design pillar. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | All three operators have precise, non-overlapping definitions. No self-referential paradoxes — `===` cannot be defined in terms of `==` or `is`. The Transitivity Invariant prevents the NaN paradox. `==` falls back to `===` by default, creating a clean hierarchy without ambiguity. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "Three operators are minimal — removing any one makes some necessary comparison inexpressible." Removing `===` would make structural comparison dependent on trait implementation. Removing `==` would prevent domain-specific equivalence. Removing `is` would make identity checks impossible for shared types. Result: all three are necessary. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Equality operates at Level 2 (Language Patterns) in the Semantic Dependency Architecture — it composes Primitive Operations (Level 1) into higher-level comparison patterns. `===` generates field-by-field comparison using `pack`/`unpack` primitives. `==` delegates to the `Eq` trait (Standard Library, Level 3). `is` uses the `reference` primitive. No layer violations. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: structural comparison must know the type's structure (which seems strategy-dependent), yet equality semantics must be strategy-independent. Separation: the *semantic definition* of `===` is "field-by-field structural comparison" — the mechanism (compile-time generated comparison, runtime reflection, or trait-based dispatch) is a Strategy choice. All three strategies (Default, Embedded, High-Performance) can implement structural comparison. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "Three operators — value equality compares structure, semantic equality compares meaning, identity equality compares pointers." Each operator is explainable without "and", "but", "except" (NaN is excluded, not an exception). Remove-one-thing test: removing any operator makes some comparison pattern inexpressible. The model matches established patterns (Python `is` for identity, Rust `==` for trait-based, JavaScript `===` for strict). No conceptual debt. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: each operator is unambiguous — `===` always compares structure, `==` always delegates to the trait, `is` always compares identity. No context-dependent semantics. Schema round-trip: all three operators are expressible in the abstract grammar. Hallucination surface: zero — each operator has exactly one meaning. Self-correction: using `is` on a value type is statically detectable (warning: always returns false). |

**Gates not applied:** None — all seven gates are required for an architecture-level decision establishing a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/EQUALITY.md` — Full concept specification
- `what/SEMANTIC_MODEL.md` § Identity — Identity dimension
- `what/GLOSSARY.md` — Value Equality, Semantic Equality, Identity Equality
- `how/DESIGN_PRINCIPLES.md` — Explicitness, Consistency, Data First, Named Before Symbolic
- `how/gates/DECISION_VALIDATION.md` — Gate framework

### Supersedes

*None* — this is a new decision, not a replacement.
