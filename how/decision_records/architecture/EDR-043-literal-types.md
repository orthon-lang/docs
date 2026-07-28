# EDR-043: Literal Types — Values as Types

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Type System)

---

### Context

In some type systems, a specific concrete value — a string like `"GET"`, a number like `42`, a boolean like `true` — can be a type unto itself, inhabited only by that exact value. These **literal types** compose with union types to form closed sets: `type Method = "GET" | "POST" | "PUT"` as an alternative to a named ADT or enum.

With EDR-039 establishing Algebraic Data Types as Orthon's single sum-type mechanism, and the enum question resolved (no separate enum construct), literal types offer a complementary approach: they provide the same "closed set of distinct values" capability without requiring a named type declaration.

The key tension: **literal types overlap with simple ADTs** (`type Method = GET | POST | PUT` vs. `type Method = "GET" | "POST" | "PUT"`). Both solve "a fixed set of distinct values." Per the Manifesto's "One concept — one syntax," Orthon should not ship two overlapping closed-set mechanisms.

The research document at `how/concepts/research/important/LITERAL_TYPES.md` explores this in depth, including the widening rule (whether `let x = "GET"` infers as `"GET"` the literal type or `String` the base type).

---

### Decision

**1. Literal types are accepted as a Language feature.** String, integer, boolean, and floating-point literals produce singleton types inhabited only by that value. Literal types compose with union types (EDR-045) to form closed sets.

```orthon
# Literal type in annotation
let method: "GET" = "GET"

# Closed union of literal types
type Method = "GET" | "POST" | "PUT"

type Port = 80 | 443 | 8080

type Flag = true | false
```

**2. Widening is explicit.** A binding's type is the literal type if:
- It is an immutable binding (`let`) with a literal initializer → preserves the literal type.
- It is a mutable binding (`var`) with a literal initializer → widens to the base type.

This is simpler than TypeScript's context-dependent widening. The rule is: **immutable bindings preserve literal types; mutable bindings widen to base types.** This is one explicit, always-applicable rule.

```orthon
let x = "GET"    # type: "GET" (literal preserved)
var y = "GET"    # type: String (widened — mutable)
```

**3. Literal types are restricted to primitive scalars:** `String`, `Int`, `Float`, `Bool`. No compound literal types (no `[1, 2]` as a type) in v0.1.

**4. Literal types are input to type-level computation (EDR-046).** The primitive `keyof` intrinsic produces a union of literal property-name types from a record type. Conditional type intrinsics can match against literal types.

**5. Literal types coexist with ADTs (EDR-039).** The programmer chooses:
- **ADT** for named, declared variants with compiler-enforced variant identity.
- **Literal type union** for string/number literal sets that map to external formats (JSON, HTTP methods, protocol constants).

Both benefit from pattern matching exhaustiveness, but ADTs provide stronger guarantees (variant identity is declared, not value-dependent). The choice is ergonomic — ADTs are preferred for internal domain modelling; literal type unions are preferred for external-facing API boundaries.

---

### Consequences

- **Positive:**
  - Enables TypeScript-style closed string unions for API boundaries and protocol constants.
  - Simple widening rule (immutable vs. mutable) is LLM-generable.
  - Composes with UNION_INTERSECTION_TYPES (EDR-045) for closed-set modelling.
  - Composes with TYPE_LEVEL_COMPUTATION (EDR-046) for `keyof` and conditional type matching.
  - Coexists with ADTs — programmer chooses the right tool for the context.

- **Negative:**
  - Adds complexity to the type system — literal types must be tracked, narrowed, and widened.
  - Weaker exhaustiveness than ADTs — a literal type union has no compiler-enforced "all variants" beyond syntactic membership. Adding a new arm silently changes the type.
  - Overlaps with simple ADTs for payload-free variants (e.g., `type Method = GET | POST` vs. `type Method = "GET" | "POST"`).
  - Widening rule adds a semantic distinction between `let` and `var` beyond mutability.

---

### Compliance

1. Every literal in immutable binding position must produce a singleton literal type.
2. Every literal in mutable binding position must widen to its base type.
3. Literal types must narrow correctly in pattern matching (e.g., `match method { case "GET" => ... }` narrows `method` to `"GET"` in the arm).
4. Literal type unions must participate in type-level computation intrinsics (`keyof`, conditional types).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| No literal types — ADTs only for closed sets | Misses the ergonomic benefit for external-facing string/number sets. ADTs require declaration ceremony; literal unions are inline. |
| TypeScript-style context-dependent widening (let widens, const preserves, `as const` opt-in) | Violates the LLM Generability Gate — context-dependent rules are exactly the "hidden conversion" class of problem Orthon avoids. One explicit rule is simpler. |
| Literal types for strings only | Integers and booleans are equally useful as literal types (port numbers, flags, discriminated union discriminants). |
| Full TypeScript-style literal types (compound, template literal) | Adds unnecessary complexity for v0.1. Primitive scalar literal types cover the essential use cases. Template literal types and compound literal types are deferred. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Literal types solve a real ergonomic need — modelling closed sets of string/number values without ADT declaration ceremony. Essential for API boundaries and protocol constants. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | One explicit widening rule eliminates context-dependent ambiguity. No contradictions with existing type system (ADTs, pattern matching). |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | The concept is simple: a value is its own type. The widening rule is a single, always-applicable rule. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on the existing type system — no new architectural layer. Literal types are a natural extension of the type inference mechanism (EDR-027). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Literal type semantics (singleton types, widening, narrowing) are independent of any memory layout or allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Literal types are a well-understood feature (TypeScript, Scala 3, Python typing). The restricted scalar-only scope limits long-term complexity. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The single widening rule (`let` preserves, `var` widens) is LLM-generable. Literal type union syntax (`"GET" | "POST"`) is unambiguous. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-043 for per-gate reasoning trail.
