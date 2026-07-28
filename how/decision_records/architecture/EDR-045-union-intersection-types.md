# EDR-045: Union and Intersection Types — Structural Type Combinators

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Type System)

---

### Context

Union types (`A | B`) and intersection types (`A & B`) are structural, set-theoretic type combinators that operate over arbitrary types — including anonymous object shapes — with no tag, discriminant, or prior variant declaration required.

This is a distinct question from Algebraic Data Types (EDR-039). ADTs require declaring a named type up front with named variants and support exhaustive `match` against an explicit tag. Structural union types require no named-type declaration, no tag, and apply to any two existing types, including types the union's author does not own.

The core tension: Orthon already has ADTs (EDR-039) as the canonical "this OR that" mechanism. Introducing structural union types alongside creates a second "one of several types" mechanism, potentially violating "One concept — one syntax." However, structural union types serve genuinely different use cases — ad-hoc combination at point of use, open to types the user does not own — that ADTs cannot cover without declaration ceremony.

Intersection types (`A & B`) merge the structural shape of two types into a new anonymous type with all members of both — a purely compile-time type combinator with no runtime object created. This is distinct from composition (runtime/instance-level) and from declaring a new product type.

The research document at `how/concepts/research/important/UNION_INTERSECTION_TYPES.md` explores this in depth.

---

### Decision

**1. Union types (`A | B`) are accepted as a Language feature.** The `|` combinator creates an anonymous, untagged union of two or more types:

```orthon
type ID = String | Int

fun print_id(id: String | Int)
    match id:
        case s: String => print(s)
        case i: Int    => print(i.to_string())
```

Union types:
- Are **untagged** — no discriminant is stored at runtime. The runtime representation is the value itself (no boxing, no tag).
- Support **narrowing** via `match` or `is` checks — the compiler tracks which member of the union is active.
- Are **structural** — any two types can be combined at the point of use without prior declaration.
- Are **closed** in the sense that the union's members are explicitly listed — adding a member changes the type.
- Cannot produce exhaustive pattern matching in the same sense as sealed ADTs — the compiler cannot enumerate "all possible types" in an untagged union.

**2. Intersection types (`A & B`) are NOT accepted for v0.1.** The intersection combinator `A & B` is redundant with Orthon's existing product type mechanism — a named record type carrying all fields of both types covers the same structural merging need more explicitly. Intersection types are deferred to post-v0.1 evaluation.

**3. Union types are restricted to concrete named types or literal types (EDR-043).** Anonymous structural shapes (TypeScript's `{name: string} | {age: number}`) are not supported in v0.1 — union members must be named types. This restriction preserves exhaustiveness when combined with structural widening and improves LLM generability.

**4. Narrowing over union types follows the same flow-sensitive rules as TYPE_LEVEL_NULL_SAFETY (EDR-028).** After a `match` or `is` check, the compiler narrows the value to the matched member. Narrowing requires effectively-immutable binding (no reassignment in the narrowed scope).

---

### Consequences

- **Positive:**
  - Enables ad-hoc union composition at point of use — no ADT declaration ceremony for simple cases like `String | Int`.
  - Composes with LITERAL_TYPES (EDR-043) for closed string/number unions.
  - Structural union of named types is simpler than TypeScript's fully open structural model.
  - Intersection types rejected avoids redundancy with product types — keeping the type system simpler.

- **Negative:**
  - Overlaps with ADTs — `String | Int` and `type ID = StringId(value: String) | IntId(value: Int)` express similar concepts with different guarantees.
  - No exhaustiveness guarantee — untagged union types cannot be exhaustively checked by the compiler.
  - Narrowing requires the same effectively-immutable precondition as TYPE_LEVEL_NULL_SAFETY (EDR-028).
  - Restricted to named types — anonymous structural unions are not supported.

---

### Compliance

1. The `|` combinator must produce an anonymous union type — no named declaration required.
2. Union members must be named types or literal types — no anonymous structural shapes.
3. Narrowing via `match` or `is` must follow the same flow-sensitive rules as EDR-028.
4. Runtime representation of a union value is the value itself — no tag or discriminant.
5. Pattern matching on union types does NOT require exhaustiveness (unlike ADTs).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| No structural union types — ADTs only | Misses the ergonomic benefit of ad-hoc composition. `String | Int` is meaningfully lighter than declaring a named sum type. |
| Full TypeScript-style structural unions (any types, anonymous shapes) | Exhaustiveness becomes impossible to guarantee. Open-ended structural algebra multiplies compiler complexity and LLM reasoning cost. Restricted to named types balances expressiveness with simplicity. |
| Intersection types accepted | Redundant with product types. Declaring a named record is more explicit than `A & B` and provides a documented type name. |
| Union types only for StdLib types | Artificial restriction — the usefulness of union types is in composing user-defined types. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Union types solve a real ergonomic problem — ad-hoc composition of alternative types without declaration ceremony. Essential for JSON-like APIs, configuration, and heterogeneous collections. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Clear rules: named type members only, no tag at runtime, narrowing follows EDR-028 rules. No contradictions with ADTs (EDR-039) — ADTs are for declared variants with exhaustiveness; unions are for ad-hoc composition. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | The `A | B` combinator is the simplest possible union form. Named-type-only restriction limits complexity. Intersection types excluded reduces surface area. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on the existing type system infrastructure — narrowing reuses EDR-028 rules, literal types provide the members (EDR-043). No new architectural layer. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Union type semantics (structural, untagged, narrowing via match/is) are independent of any memory layout or allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Named-type-only restriction limits long-term complexity. Union types are a well-understood, stable feature (TypeScript, Scala 3). |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The `A | B` syntax is unambiguous and widely understood. Named-type union members are easy for LLMs to reason about. The no-exhaustiveness rule is simple and clear. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-045 for per-gate reasoning trail.
