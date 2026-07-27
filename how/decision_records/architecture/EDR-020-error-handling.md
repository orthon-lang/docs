# EDR-020: Error Handling — Result Type with Explicit Propagation

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon needs a principled mechanism for handling errors — operations that may fail. The language's design principles demand explicit fallibility (errors visible in function contracts), composable error handling, and compiler-enforced handling guarantees. No unchecked exceptions.

The research document at `how/concepts/research/essential/ERROR_HANDLING.md` established the conceptual model, tracing the evolution from return codes (C) through exceptions (Java, Python) to monadic result types (Rust, Swift, OCaml). Five steps of the Concept Design Review were completed:

1. **Step 1 (Idea/Problem):** Orthon needs a way for functions to declare fallibility, propagate errors without hidden control flow, and require the caller to handle them — without exceptions or unchecked fallibility.
2. **Step 2 (Minimal Solution):** A `Result<T, E>` monadic type with `Ok(T)` and `Error(E)` variants, `?` short-circuit propagation operator, combinators (`map`, `and_then`, `or_else`), and compiler-enforced exhaustive matching.
3. **Step 3 (Principle Check):** Aligns with Explicitness (errors declared in signatures, `?` is visible), Declarative With Static Guarantees (compiler enforces handling), Minimal Core (one concept replaces exceptions, error codes, and checked exceptions).
4. **Step 4 (Examples):** All canonical forms documented in `what/concepts/ERROR_HANDLING.md` with `orthon` code blocks, including `?` propagation, pattern matching, combinators, and relationship with `Option`.
5. **Step 5 (EDR):** This document.

The Decision Pipeline (processed in Phase 4, Wave 1) classified ERROR_HANDLING as **Language** per D-03: `Result<T,E>` is a type with compiler-level propagation mechanism (`?` operator). Not expressible via composition of primitives. Compiler must enforce handling.

---

### Decision

Adopt the **`Result<T, E>` monadic type model** for Orthon error handling:

1. **`Result<T, E>`** — A sum type with two variants: `Ok(T)` (success) and `Error(E)` (failure).
2. **`?` operator** — Short-circuit propagation. Inside a function returning `Result`, `expr?` unwraps `Ok` or returns `Error` immediately.
3. **Exhaustive handling** — The compiler enforces pattern matching on `Result` to cover both `Ok` and `Error`.
4. **No exceptions** — There is no `try-catch` mechanism. All fallibility is declared in the type system.
5. **Combinators** — `map`, `and_then`, `or_else`, `unwrap`, `unwrap_or`, `unwrap_or_else`, `is_ok`, `is_error`, `ok`, `error`.
6. **Relationship with `Option`** — `Result` and `Option` are distinct types with distinct combinators. Both support `map` and `and_then` for uniform composition.
7. **Relationship with ERROR_UNION** — Multiple error sources compose naturally via union types (e.g., `Result<Data, IOError | ParseError>`). The ERROR_UNION concept (Plan 04-02) defines the formal error-union semantics.

---

### Consequences

**Positive:**
- Explicit fallibility — every function that can fail declares `Result<T, E>` in its signature.
- Compiler-enforced handling — unhandled `Result` is a compile-time error.
- Composable error transformation via combinators (`map`, `and_then`, `or_else`).
- No hidden control flow — `?` is syntactically visible at every propagation site.
- No runtime exceptions — eliminates the entire class of uncaught-exception bugs.
- Clean separation: `Result` for fallible operations, `Option` for absent values.

**Negative:**
- Boilerplate for deeply nested fallible calls (mitigated by `?` operator).
- Error type polymorphism requires explicit union types or `dyn Error` for heterogeneous error sources (mitigated by ERROR_UNION).
- Learning curve for programmers accustomed to exceptions.
- Combinator chains can be less readable than sequential exception-style code for simple error handling.

---

### Compliance

1. The `what/concepts/ERROR_HANDLING.md` specification defines the canonical semantics.
2. Every implementation must enforce exhaustive matching on `Result` at compile time.
3. The `?` operator is the canonical propagation mechanism — no implicit propagation.
4. No `try-catch`, `throw`, or unchecked exceptions in the language.
5. Combinators are standard library methods on `Result<T, E>` (default implementations).
6. Functions that can fail must declare `Result<T, E>` in their return type — no implicit fallibility.
7. ERROR_UNION integration follows the formal definition in Plan 04-02.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Checked exceptions (Java) | Hidden control flow — `try-catch` is not visible at the call site. No composability — exceptions cannot be transformed or chained. Violates Explicitness. |
| Unchecked exceptions (Python, C#, Java runtime) | No compiler enforcement — errors can escape unhandled. Violates Declarative With Static Guarantees. |
| Error codes + `errno` (C) | Easy to ignore — no type safety, no composability, no compiler enforcement. Violates Safety. |
| Algebraic effects | Powerful but complex — requires effect system, handlers, and runtime support. Overkill for error handling. Violates Minimal Core. |
| Go-style multi-return `(T, error)` | Verbose at every call site. No composability — no `map`/`and_then` for error transformation. Easy to forget to check. |
| Optional-only (no error type) | Absence and failure are semantically distinct — `Option` cannot carry diagnostic information. Violates Explicitness. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to call a function that might fail and handle the error without crashing." Every programmer encounters this. The solution serves VISION.md's Comfortable by Design pillar — `Result<T, E>` makes error handling visible and composable. Code example from ERROR_HANDLING.md (`divide(a, b)`) shows the pain point (division by zero) and the solution (type-safe error handling). |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | All constructs have precise, non-overlapping definitions. `Result<T, E>` = sum type with `Ok`/`Error` variants. `?` = short-circuit propagation (match + early return). Combinators = transformation functions on `Result`. No self-referential paradoxes — `Result` does not handle itself. The `?` operator is only valid inside functions returning `Result`. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The Result model is minimal — removing any component makes error handling incomplete." Removing `?` would force explicit matches at every call site (Go problem). Removing the type parameter `E` would lose diagnostic information (null/optional problem). Removing combinators would force nested match blocks. Removing exhaustiveness checking would allow unhandled errors. Result: all components are necessary. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | `Result<T, E>` operates at the type-system level — it is a generic sum type. The `?` operator composes Primitive Operations (match + return) into a concise propagation form. No layer violations — `Result` depends on sum types (`pack`/`unpack`) and pattern matching (both Level 1 primitives). Combinators are Level 2 (Language Patterns in StdLib). Error type unions integrate with the type system per EDR-012's Semantic Dependency Architecture. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: `?` propagation requires control-flow manipulation (seems strategy-dependent), yet error semantics must be strategy-independent. Separation: the *semantic definition* of `?` is "match + early return" — the mechanism is strategy-independent. Stack unwinding (Default), setjmp/longjmp (Embedded), or CPS transformation (High Performance) are all valid Strategy choices. Error type layout (tagged union vs. pointer) is a Strategy choice. Semantics (which variant is active, what happens on propagation) are identical. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "`Result<T, E>` is a type-safe way to say an operation might fail — `?` propagates the error, and combinators transform it." Each component is explainable without "and", "but", "except". Remove-one-thing test: removing `Result` would force the language back to exceptions or error codes. The model matches established patterns (Rust, Swift, OCaml). Evolution path: ERROR_UNION (Plan 04-02) extends `Result` with union error types; new combinator methods can be added to the Standard Library. No conceptual debt. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: `Result<T, E>`, `Ok(T)`, `Error(E)`, and `?` each have a single, unambiguous meaning. No context-dependent syntax. Schema round-trip: fully expressible in the type system — `Result` is a generic sum type. Hallucination surface: low — the pattern matches Rust, which LLMs generate reliably. Self-correction: missing error handling is statically detectable (unhandled `Result`), wrong combinator use produces type errors, missing `Error` variant in pattern match is a compile error. Common LLM mistakes (forgetting `?`, using `unwrap` when `?` is appropriate) are statically detectable. |

**Gates not applied:** None — all seven gates are required for an architecture-level decision establishing a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/ERROR_HANDLING.md` — Full concept specification
- `what/concepts/NULL_SAFETY.md` — `Option<T>` for absence; `Result<T, E>` for failure. Distinct but complementary.
- `what/SEMANTIC_MODEL.md` — Ownership and Mutation (error values carry ownership semantics)
- `what/GLOSSARY.md` — Result Type, Error Propagation
- `how/concepts/research/essential/ERROR_HANDLING.md` — Original research document
- ERROR_UNION (Plan 04-02) — Union types for multi-source error composition. Cross-referenced in EDR-020 as a related concept for error type composition. The `Result<T, E>` model supports error types that are themselves union types, enabling multiple error sources (e.g., `IOError | ParseError`) to be composed naturally. ERROR_UNION defines the formal semantics for error type polymorphism.
- `how/concepts/research/essential/TRAITS.md` — Traits provide the method dispatch for combinators
- `how/concepts/research/FUNCTIONS.md` — Closures used in combinator arguments
- `how/DESIGN_PRINCIPLES.md` — Explicitness, Minimal Core, Declarative With Static Guarantees

### Supersedes

*None* — this is a new decision, not a replacement.
