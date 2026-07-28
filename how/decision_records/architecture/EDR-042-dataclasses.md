# EDR-042: Dataclasses — Derive-Based Data Carriers

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module (Standard Library)

---

### Context

A large fraction of types in any codebase are passive data carriers — configuration objects, API response DTOs, database rows, value objects. In languages without syntactic support for this pattern, each such type requires many lines of mechanically identical code: constructors, accessors, equality, hashing, and string representation.

Orthon already has:
- **TRAITS (EDR-019)** for behavioural contracts.
- **AST_MACROS (EDR-029)** with `@derive` for generating trait implementations.
- **EQUALITY (EDR-017)** with structural `===` and semantic `==`.

The question: does Orthon need a dedicated `data class` keyword, or does the existing `@derive` mechanism already provide the dataclass pattern?

The research document at `how/concepts/research/important/DATACLASSES.md` explores this in depth.

---

### Decision

**1. Orthon has no dedicated `data class` or `record` keyword.** Dataclasses are a pattern expressed through the existing `@derive` mechanism (EDR-029). A "dataclass" is simply a type declaration annotated with `@derive(init, eq, repr, hash)`:

```orthon
@derive(init, eq, repr, hash)
type Point(x: Float, y: Float)
```

This generates:
- A constructor (`init`) accepting all fields as named parameters.
- Structural equality (`eq`) via compiler-generated field-by-field `===`.
- A string representation (`repr`) showing the type name and field values.
- A hash function (`hash`) for use in hash-based collections.

**2. Immutability is the default.** Fields in a dataclass-style type are immutable by default (a `fun` declaration kind field is read-only). Explicit `var` fields are available for mutable carriers but must be declared explicitly.

**3. Copy-with-modify is provided via a `with` expression (StdLib/compiler intrinsic).** The `with` expression creates a new instance with specified fields changed:

```orthon
p2 = Point(x: 3.0, y: 4.0)
p3 = p2 { y = 5.0 }    # copy with y changed
```

The `with` expression is a compiler-recognized intrinsic (not syntax sugar) — it generates a field-by-field copy with selective reassignment.

**4. Dataclass derives are StdLib.** The derive implementations for `init`, `eq`, `repr`, and `hash` are standard library macros registered in the derive registry. They are not built into the compiler.

---

### Consequences

- **Positive:**
  - No new keyword — dataclasses reuse the existing `@derive` mechanism.
  - Composability: users can derive any subset of `init`, `eq`, `repr`, `hash`.
  - Extensibility: custom derives work alongside standard dataclass derives.
  - Immutability by default aligns with Orthon's data-first philosophy.
  - `with` expression enables ergonomic copy-with-modify.

- **Negative:**
  - No single `@dataclass` annotation — requires listing individual derives (`@derive(init, eq, repr, hash)`).
  - The `with` expression adds a new syntactic form (compiler intrinsic), increasing parser complexity.
  - Derive macros must be available at compile time via the macro registry (EDR-029).

---

### Compliance

1. `@derive(init, eq, repr, hash)` must generate constructor, structural equality, string representation, and hash for any concrete type.
2. Generated constructor accepts named parameters matching field declarations, in declaration order.
3. Structural equality (`eq` derive) must use `===` (EDR-017) for recursive field comparison.
4. `with` expression must produce a new value — it must never mutate the original.
5. Derive implementations are StdLib macros — they are not compiler built-ins.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Dedicated `data class` keyword (Kotlin-style) | Violates Minimal Core — the `@derive` mechanism already provides the same capability. A new keyword would overlap with existing functionality. |
| Java records-style positional constructor | Named parameters at construction time are more readable and less error-prone. Positional is available as shorthand. |
| No `with` expression — manual construction only | Loses ergonomic benefit. Copy-with-modify is the most common data-carrier operation after construction. |
| Structural equality built-in for all types (no derive needed) | Already provided by `===` (EDR-017) for value equality. The `eq` derive generates the `==` (semantic equality) implementation, which defaults to `===` if not overridden. |
| Dataclass fields are implicitly properties | Adds unnecessary complexity for v0.1. Properties are a separate concern (deferred). Dataclass fields are direct accessors. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Dataclasses eliminate a major source of boilerplate — programmers write `@derive(init, eq, repr, hash)` instead of dozens of lines of mechanical code. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Dataclass derives are a straightforward application of the existing `@derive` mechanism. No new type rules or semantic contradictions. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | The `@derive` pattern is simpler than a dedicated keyword — one mechanism (derive) covers all boilerplate generation needs. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | StdLib classification places dataclass derives in the correct layer. The `with` expression is a limited compiler intrinsic (copy-with-modify), not a general syntactic extension. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Dataclass semantics (structural equality, copy-with-modify) are independent of any specific memory layout or allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | The derive-based approach is more maintainable than a dedicated keyword — new derive targets can be added without language changes. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `@derive(init, eq, repr, hash)` is a simple, regular annotation that LLMs can reliably generate. The pattern is familiar from Rust's `#[derive(...)]`. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-042 for per-gate reasoning trail.
