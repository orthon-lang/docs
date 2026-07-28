# EDR-039: Algebraic Data Types Subsume Dedicated Enum Mechanism

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Type System)

**Supersedes:** EDR-040 (ENUM_ALTERNATIVES, folded into this decision)

---

### Context

Orthon needs a mechanism for modelling data that takes one of several known forms — "this OR that" (sum types) alongside "this AND that" (product types) — in a way that is type-safe and composable.

Two related but distinct questions arise:

1. **Algebraic Data Types (ADTs):** How does the language model data as a choice between named variants, where each variant carries its own fields? This covers sum types (e.g., a shape is Circle OR Rectangle OR Triangle) and product types (a Point has x AND y), plus recursive types (trees, lists).

2. **Enum Alternatives:** Does the language also need a dedicated enum construct for finite sets of named, payload-free values — or are ADTs (via sealed traits + pattern matching) sufficient?

With TRAITS (EDR-019) providing sealed trait hierarchies and PATTERN_MATCHING (EDR-025) providing exhaustive structural matching, the foundation for ADTs already exists. The question is whether an explicit ADT declaration form adds sufficient value over manual sealed trait + match patterns, and whether a separate enum construct introduces undesirable redundancy per the Manifesto's "One concept — one syntax" principle.

The research documents at `how/concepts/research/important/ALGEBRAIC_DATA_TYPES.md` and `how/concepts/research/important/ENUM_ALTERNATIVES.md` explore these questions in depth.

---

### Decision

**1. Algebraic Data Types are accepted as a Language feature.** ADTs are declared using the `type` keyword with pipe-separated variant syntax, building on sealed trait exhaustiveness from TRAITS (EDR-019) and pattern matching from PATTERN_MATCHING (EDR-025). The ADT declaration form provides:

- **Combined declaration:** variant names and their field types in a single `type Name = Variant(fields) | Variant(fields)` form.
- **Automatic discriminant:** the compiler generates the tag/discriminant that distinguishes variants at runtime.
- **Sealed variant set:** like sealed traits, the variant set is closed; pattern matching must be exhaustive.
- **Recursive types:** supported with compiler-enforced termination checks (size bounds, indirection via reference for unbounded recursion).
- **Generic ADTs:** `type Option<T> = Some(value: T) | None` — generics combine freely per EDR-024.
- **`@derive` compatibility:** structural derives (`Show`, `Eq`, `Clone`, `Hash`) apply to ADT declarations via the existing derive mechanism (EDR-029).

**2. Orthon has exactly one sum-type mechanism: Algebraic Data Types.** No separate enum construct is added. Dedicated enums (Java-style enum classes, C-style named integer constants) are subsumed by ADTs:

- Payload-free variants (`type Color = Red | Green | Blue`) serve the simple enum use case directly with compiler-enforced exhaustiveness.
- Data-carrying variants (`type Shape = Circle(radius: Float) | Rectangle(w: Float, h: Float)`) cover the full ADT use case.
- This satisfies "One concept — one syntax" (Manifesto §4): one sum-type mechanism instead of three overlapping ones (ADTs, enums, named constants with iota).

**3. Variant fields are named by default.** Positional fields are available as a shorthand for single-field variants. Named fields improve readability and enable copy-with-modify at construction.

```orthon
# ADT with named fields
type Shape = Circle(radius: Float)
           | Rectangle(width: Float, height: Float)

# Simple enum-style ADT (payload-free variants)
type Color = Red | Green | Blue

# Generic ADT
type Option<T> = Some(value: T) | None

# Recursive ADT
type Tree<T> = Empty
             | Node(value: T, left: Tree<T>, right: Tree<T>)
```

**4. Methods on ADT variants** follow the trait model: traits are defined separately, and `impl` blocks provide implementations. ADT variants do not carry inherent methods.

---

### Consequences

- **Positive:**
  - Single sum-type mechanism reduces language surface area.
  - Simple enum-style ADTs are as concise as dedicated enums (`type Color = Red | Green | Blue`).
  - Data-carrying ADTs cover Rust-style enum variants, Kotlin sealed classes, and TypeScript discriminated unions — all in one construct.
  - Existing TRAITS + PATTERN_MATCHING foundations are reused.
  - `@derive` mechanism handles boilerplate generation (Eq, Show, Clone) without new keywords.

- **Negative:**
  - No auto-increment mechanism (like Go's `iota`) — ADTs have no implicit integer values. Serialization to/from integers requires explicit mapping (StdLib concern).
  - Pattern matching on simple enum-style ADTs requires explicit `case` arms even for payload-free variants — no implicit switch-to-integer conversion.
  - ADT exhaustiveness depends on sealed trait semantics from TRAITS (EDR-019); if TRAITS changes, ADT semantics must be revisited.

---

### Compliance

Compliance will be verified by:

1. The ADT declaration form (`type Name = Variant(fields) | Variant(fields)`) must compile to a sealed trait hierarchy with named variant constructors.
2. Pattern matching on ADTs must produce a compile-time error if any variant is uncovered.
3. Recursive ADTs must pass compiler termination checking (size bounds or indirection enforcement).
4. Generic ADTs must satisfy the same trait-bounded generics rules as EDR-024.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Dedicated enum construct (Java/Kotlin-style) | Redundant with sealed trait + ADT mechanism. "One concept — one syntax" mandates a single sum-type mechanism. |
| Named constants with iota (Go-style) | Weaker type safety — constants are integer-typed, not distinct types. No exhaustiveness checking. Violates the Data First principle (data identity should be explicit). |
| Full ADTs + separate enums (Rust-style) | Overlapping mechanisms. Rust's `enum` is itself an ADT; Orthon's `type ... = ... | ...` covers the same ground. No benefit to two parallel constructs. |
| No ADTs — sealed traits only | ADT declaration syntax provides ergonomic benefit over manual sealed trait + variant type declarations. The `type` form with pipe syntax is more readable for sum types than N separate `type` + `trait` + `impl` declarations. |
| Positional fields only | Named fields improve readability for multi-field variants and enable copy-with-modify. Positional shorthand is available for single-field variants. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | ADTs solve a fundamental data-modelling problem. Every non-trivial program needs sum types. Enum use case is covered with zero additional syntax. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | One sum-type mechanism, one declaration form. No contradictions with existing TRAITS or PATTERN_MATCHING semantics. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Combined ADT + enum in a single mechanism is simpler than two separate mechanisms. The `type ... = ... | ...` form is intuitive. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | ADTs build on existing sealed trait (EDR-019) and pattern matching (EDR-025) foundations. No new architectural layer required. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | ADT semantics (sealed variants, exhaustiveness, recursive types) are defined independently of any memory layout strategy. Tagged union is the default layout but alternatives (niche optimisation, flat layout) are permitted. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Single sum-type mechanism is more maintainable than two overlapping ones. ADT pattern is proven across Rust, Haskell, OCaml, Swift, Kotlin. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | ADT declaration syntax is simple and regular. Pattern matching on ADTs follows the same rules as pattern matching on sealed traits. LLMs already demonstrate reliable ADT generation in similar languages (Rust enums, TypeScript discriminated unions). |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-039 for per-gate reasoning trail.
