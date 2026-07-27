# EDR-018: Null Safety — Option Type Without Null Sentinel

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Null pointer errors are the most common class of runtime crashes in mainstream languages. Tony Hoare called null references his "billion-dollar mistake." The core design tension: how does a language represent the absence of a value without introducing null pointer errors?

Existing approaches fall into two camps:
- **Nullable annotations** (C# `T?`, Kotlin `T?`) — compiler-checked at call sites, but `null` remains as a valid sentinel value that can escape the type system.
- **Option/Maybe types** (Rust `Option<T>`, Haskell `Maybe<T>`) — absence is a first-class concept in the type system. No special `null` value.

The research document at `how/concepts/research/essential/NULL_SAFETY.md` established the conceptual model. Five steps of the Concept Design Review were completed:

1. **Step 1 (Idea/Problem):** Null pointer errors. Every reference of type `T` can silently be `null`, turning every dereference into a potential runtime crash.
2. **Step 2 (Minimal Solution):** No `null` sentinel. `Option<T>` as a sum type. Syntactic convenience via `?.`, `??`, `!`. Compiler-enforced exhaustive matching.
3. **Step 3 (Principle Check):** Aligns with Declarative With Static Guarantees (absence is statically tracked), Explicitness (forced unwrap `!` is visible), Minimal Core (one concept replaces an entire bug class).
4. **Step 4 (Examples):** All canonical forms documented in `what/concepts/NULL_SAFETY.md` with `orthon` code blocks.
5. **Step 5 (EDR):** This document.

The Decision Pipeline (processed in Phase 4) classified NULL_SAFETY as **Language** per D-03: the `Option<T>` type adds `?` semantics and compiler-enforced exhaustiveness not decomposable to existing primitives.

---

### Decision

Adopt the **Option type model** for Orthon null safety:

1. **No `null` sentinel** — There is no `null` value in the language.
2. **`Option<T>`** — The canonical representation of optional values. Sum type with variants `Some(T)` and `None`.
3. **`?.` (elvis)** — Optional chaining: short-circuits to `None` on `None`.
4. **`??` (fallback)** — Unwrap or default value.
5. **`!` (forced unwrap)** — Panics on `None`. Syntactically visible.
6. **Exhaustiveness** — Pattern matching on `Option` must cover both `Some` and `None`.

`Option<T>` and `Result<T, E>` are distinct: `?` works on `Result` (failure propagation), `?.` works on `Option` (absence propagation).

---

### Consequences

**Positive:**
- Eliminates the entire class of null pointer dereference errors.
- Compiler-enforced exhaustiveness catches missing cases at compile time.
- `Option` and `Result` are distinct, preventing confusion between "absent value" and "operation failed."
- `?.` and `??` provide ergonomic syntax for common patterns without sacrificing safety.
- `!` makes forced unwrap syntactically visible — no silent panics.
- Named function equivalents (`opt.or(default)`, `opt.map(fn)`, etc.) satisfy the Named Before Symbolic principle.

**Negative:**
- `Option` chaining via `?.` adds syntactic surface beyond a simple dereference.
- Interoperation with non-Orthon code (C FFI) may still need a `null` sentinel at the boundary, requiring explicit conversion.
- Forced unwrap `!` can still panic at runtime, though the explicitness requirement reduces accidental use.
- Programmers from null-safe languages (Kotlin, C#) must learn the `Option` pattern rather than `T?` syntax.

---

### Compliance

1. The `what/concepts/NULL_SAFETY.md` specification defines the canonical semantics.
2. No implementation may introduce a `null` sentinel value.
3. Every pattern match on `Option<T>` must be checked for exhaustiveness at compile time.
4. The `!` operator must produce a runtime panic on `None` — undefined behaviour is not permitted.
5. `?.` must short-circuit to `None` without evaluating the right-hand side when the left is `None`.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Nullable types (C#/Kotlin `T?`) | `null` sentinel still exists as a value that can escape the type system. Violates Declarative With Static Guarantees — the compiler cannot guarantee absence of null at every point. |
| Java `Optional<T>` (library type) | Not enforced by the compiler. Values can still be `null`. Violates Declarative With Static Guarantees — manual discipline replaces compiler enforcement. |
| Python `None` + `typing.Optional` | No compiler enforcement. `None` is a valid value of any type at runtime. Violates Declarative With Static Guarantees. |
| Null Object pattern (no language support) | Ad-hoc per-type sentinel instances. No uniform interface. No compiler enforcement. |
| Implicitly non-null by default with `T?` annotation | Closer to Orthon's model, but preserves `null` as a sentinel. The `null` value itself is the root cause — removing it entirely is cleaner. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I dereferenced a null and the program crashed." Every working programmer has encountered this. The solution directly serves VISION.md's Comfortable by Design pillar — the programmer never needs to ask "can this be null?" if the type says so. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | `Option<T>` as a sum type has a precise definition: `Option<T> = Some(T) | None`. The operators `?.`, `??`, `!` each have a single, deterministic behaviour. No self-referential paradoxes. Distinction from `Result<T, E>` is clear and documented. No context-dependent semantics — `?.` always means optional chaining. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The Option type model is minimal — removing any component makes null safety incomplete." Removing `?.` forces explicit match for every access (poor ergonomics). Removing `??` forces explicit match for default values. Removing `!` removes the escape hatch (programmers forced into unsafe workarounds). Removing exhaustiveness checking removes the safety guarantee. Result: all components are necessary. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | `Option` operates at Level 2 (Language Patterns) — it composes primitive operations (`pack`/`unpack` for sum types, `function` for combinators, `call` for invocation) into a higher-level pattern. `?.` is syntactic sugar over match + short-circuit. `!` is syntactic sugar over match + panic. Exhaustiveness checking is a compiler feature that does not introduce new layer dependencies. No layer violations. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: `Option` is a sum type that requires runtime representation (tagged union), yet null safety must be strategy-independent. Separation: the *semantic definition* of `Option<T>` is "a value that is either `Some(T)` or `None`" — the concrete representation (tagged union, nullable pointer with sentinel, or separate boolean + data pointer) is a Strategy choice. A GC-backed strategy can use a null pointer with NaN-boxing; an arena strategy can use a discriminant + union. Behaviour (exhaustiveness, short-circuit) is identical. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "Values that may be absent are wrapped in `Option` — the compiler ensures you handle both cases." Each operator is explainable simply (`?.` = "if absent, stay absent"; `??` = "if absent, use default"; `!` = "I know it's present"). Remove-one-thing test: removing `Option` would reintroduce null pointers. The model matches established patterns (Rust `Option`, Haskell `Maybe`). No conceptual debt. The evolution path is clear: `Option` can gain more combinators in the Standard Library without changing the core language. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: `Option<T>` is unambiguous — `Some(T)` wraps a value, `None` marks absence. `?.` always short-circuits, `??` always provides a default, `!` always panics on `None`. No context-dependent semantics. Schema round-trip: fully expressible in the type system — `Option<T>` is a generic sum type. Hallucination surface: low — `?.` and `??` follow established patterns (Kotlin, TypeScript, Rust). Self-correction: missing match arms are statically detectable. A common LLM mistake (forgetting to handle `None`) is caught by the compiler. |

**Gates not applied:** None — all seven gates are required for an architecture-level decision establishing a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/NULL_SAFETY.md` — Full concept specification
- `what/SEMANTIC_MODEL.md` — Ownership and Lifetime dimensions (Option interacts with both)
- `what/GLOSSARY.md` — Option Type
- `how/concepts/research/ERROR_HANDLING.md` — `Result<T, E>` (distinct from `Option`, same combinators)
- `how/DESIGN_PRINCIPLES.md` — Declarative With Static Guarantees, Explicitness, Minimal Core

### Supersedes

*None* — this is a new decision, not a replacement.
