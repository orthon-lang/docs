# EDR-023: Error Union — Zig-Style Tag-Only Error Union for Fallible Functions

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Phase 4 (EDR-020) established `Result<T, E>` as Orthon's error handling mechanism — a monadic type with explicit `?` propagation, pattern matching for exhaustive handling, and no exceptions. However, `Result<T, E>` with a user-declared generic `E` leaves an unresolved design tension: for the common case where errors are simple tags (file not found, access denied, parse failure) without payload data, forcing the programmer to declare, maintain, and convert between explicit error enums adds ceremony without corresponding semantic value.

Research in `how/concepts/research/essential/ERROR_UNION.md` analysed Zig's Error Union model: a distinct type former `!T` where the error side is an inferred, structurally-widening set of unit-like error tags. The Error Union is not sugar for `Result<T, E>` — it is a distinct kind of type with different properties:

1. **Inferred error sets** — The compiler infers the minimal error set from every fallible call inside the function body. The signature `!T` asks the compiler to discover the error set, not for the programmer to declare it.
2. **Structural widening** — A narrower error set coerces implicitly into any superset. No explicit `From` or `Into` conversion required.
3. **Tag-only errors** — Error values are unit tags without payload data. No associated data, no fields — just a name.
4. **`anyerror` escape hatch** — A universal error supertype for boundaries where precise error tracking is impractical.

This creates a mutual-exclusivity tension with `Result<T, E>`'s payload-bearing model. The two serve different use cases:
- Error Union — simple, tag-only errors; cheap, inferred, structurally-widening error sets
- `Result<T, E>` — payload-bearing errors; explicit, declared, convertible via `From`

The language must decide: adopt one, adopt both as coexisting representations, or build a bridge that unifies them.

### Decision

Adopt **Error Union (`!T`) as the primary error representation** for Orthon, coexisting with `Result<T, E>` for payload-bearing errors, with the following design:

1. **Primary mechanism:** Error Union `!T` where the error set is inferred by the compiler from the function body. The function signature `!T` indicates fallibility with an inferred error set.
2. **Tag-only errors:** Error values are unit tags (e.g., `error.FileNotFound`, `error.AccessDenied`) with no payload data. For payload-bearing errors, use `Result<T, E>` explicitly.
3. **Inferred error sets:** The compiler discovers every fallible call in the function body and computes the union of all error tags. The error set is part of the type but inferred, not written. Explicit error set declaration is available as an opt-in for documentation purposes.
4. **Structural widening:** A narrower error set coerces implicitly into a wider one. This is a closed, decidable compile-time check — structural subset comparison of tag sets.
5. **Propagation:** The existing `?` operator (from EDR-020) handles both Error Union and `Result<T, E>` propagation, with the same short-circuit behaviour. No `try`/`catch` keywords are added — one propagation operator serves both representations.
6. **`anyerror` escape hatch:** A universal error supertype available for plugin boundaries, FFI, and other scenarios where precise error set tracking is impractical. Its use is tracked and documented.
7. **Explicit `Result<T, E>`:** Preserved for payload-bearing errors. When an error needs associated data (e.g., a parse error with line/column details), use `Result<T, ErrorPayload>` explicitly.

### Consequences

**Positive:**
- Eliminates error-enum boilerplate for the common case — the compiler infers what would otherwise require manual enum declaration, variant addition, and `From` impls
- Error sets stay accurate as implementation changes — no manual sync between function body and error type declaration
- Structural widening eliminates the combinator chain for error type conversion across call boundaries
- Coexistence with `Result<T, E>` preserves payload-bearing error capabilities
- One propagation operator (`?`) serves both representations — no duplication in the language

**Negative:**
- Inferred error sets are not visible in the function signature — an LLM reading a `!T` signature cannot determine which specific errors are possible without querying compiler-derived tooling (Schema Provider)
- Inferred error sets may differ across compilations if implementation changes — the set is derived from the code, not declared
- `anyerror` is a coercion escape hatch that could be overused; its adoption requires governance
- The coexistence of two error representations (`!T` and `Result<T, E>`) adds language surface area

### Compliance

1. Every function returning `!T` must have a deterministically computed error set — the same body must produce the same error set in every compilation.
2. The Schema Provider must expose inferred error sets for LLM querying.
3. Error Union and `Result<T, E>` must share the same propagation operator (`?`).
4. `anyerror` usage must be tracked and documented; a governance rule limits its use to defined boundary scenarios.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| `Result<T, E>` only (EDR-020 as sole mechanism) | Forces error-enum declaration for all fallible functions — disproportional ceremony for tag-only errors |
| Error Union only (no `Result<T, E>`) | Tag-only errors cannot carry payload data — too restrictive for parse errors, validation errors, etc. |
| Unified `!T` with payload-bearing errors | Combines inferred sets with payload types — creates complex semantics for when a set has some tags with payload and some without |
| `try`/`catch` as dedicated propagation syntax | Duplicates `?` operator; violates "one concept, one syntax" per Manifesto |
| No inferred error sets (always explicit) | Loses the primary ergonomic benefit of Error Union — set inference is the defining feature |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes `!T` and the compiler infers error tags from every fallible call. No manual error-enum declaration, no `From` impls, no conversion boilerplate. The common case (tag-only errors) becomes frictionless. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Error Union is a distinct type former with precise semantics: inferred tag set + structural widening + tag-only values. No self-referential paradoxes. The `?` operator propagates both `!T` and `Result<T, E>` — unified propagation semantics. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Hypothesis: "Error Union reduces cognitive load vs. explicit error enums for tag-only errors." Tested by comparing code examples — the Error Union variant is consistently shorter and the error set is always correct by construction. The implicit widening rule adds one concept (structural coercion) but eliminates an entire class of boilerplate (conversion impls). |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Error Union sits at the same architectural level as `Result<T, E>` — both are Core Language types (Level 0). The `?` operator is a Core Language propagation mechanism (Level 1). The Schema Provider exposure aligns with the LLM Toolchain layer. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Apparent contradiction: inferred error sets seem to tie error handling to a specific compilation strategy. Separation: the *semantic model* (tag set, structural widening, `?` propagation) is strategy-independent; the *inference algorithm* (how the compiler discovers and unions error tags across the call graph) is a Strategy choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "Inferred error tags that widen structurally." The concept is stable — Zig has proven this model works in production since 2016. The `anyerror` escape hatch provides a clear deprecation path should the inferred-set model ever become untenable. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | An LLM can reliably produce `!T` for any fallible function — the rule is simple: "if the function can fail with a tag-only error, return `!T`." The inferred set is not in the signature, but the Schema Provider exposes it for tooling. The `?` operator is already established from EDR-020. |

**Gates not applied:** None — all seven gates are required for a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/ERROR_UNION.md` — Full specification
- `what/concepts/ERROR_HANDLING.md` (EDR-020) — `Result<T, E>` coexistence
- `how/concepts/research/essential/ERROR_UNION.md` — Research analysis
- `how/DESIGN_PRINCIPLES.md` — Explicitness, Minimal Core
- `what/GLOSSARY.md` — Error Union, Error Set, Error Tag

### Supersedes

*None* — this is a new decision complementing EDR-020.
