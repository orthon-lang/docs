# Make Illegal States Unrepresentable

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process. A concept is registered only after acceptance via EDR
> (Architecture category).
>
> **Last updated:** 2026-07-29

## Issue (Why)

Yaron Minsky's aphorism — *"make illegal states unrepresentable"* — captures a
design philosophy in which the type system prevents invalid program states from
being expressed at all. A program that cannot represent an invalid state cannot
enter that state at runtime: the compiler rejects it before execution.

This is not a single language feature. It is a **cross-cutting pattern**
realised through multiple type-system mechanisms working together:

| Mechanism | What it prevents | Orthon EDR |
|-----------|-----------------|------------|
| **Literal types** | Typos in string/number constants (`"opne"` instead of `"open"`) | EDR-043 |
| **Algebraic data types** | Missing variant handling in `match`; uninitialised or partially-initialised data | EDR-039 |
| **Null safety (`Option<T>`)** | Null-pointer dereference; forgetting to handle absence | EDR-018, EDR-028 |
| **Exhaustive pattern matching** | Incomplete case analysis; unhandled variants at compile time | EDR-039 |
| **Immutable-by-default bindings** | Accidental mutation of shared data | EDR-043 (widening rule) |
| **Union types** | Functions accepting values outside a declared closed set | EDR-045 |

For a language targeting LLM code generation, this pattern is especially
valuable. An LLM sampling tokens will eventually produce a typo, a missing
case, or an invalid state. If that state cannot be represented in the type
system, it cannot compile — and therefore cannot ship. The type system serves as
a **safety net for nondeterministic generation**.

### Prior Art

**BAML** (boundaryml.com) elevates this to an explicit design philosophy:

```baml
class Ticket {
  status: "open" | "closed",   // "opne" won't compile
}
```

BAML's compiler rejects invalid literal values at the type level. The
philosophy is stated directly in BAML's documentation: an agent sampling tokens
will eventually produce an invalid state, and the type system must catch it.

**TypeScript** achieves the same effect through literal types combined with
union types:

```typescript
type Status = "open" | "closed";
function update(status: Status) { /* ... */ }
update("opne");  // Error: Argument of type '"opne"' is not assignable
```

**Rust** achieves it through enums with exhaustive `match`:

```rust
enum Status { Open, Closed }
fn update(status: Status) { /* ... */ }
// Impossible to call update with an undefined variant
```

**Scala 3** achieves it through enums and opaque types with explicit
constructors that validate at the boundary.

### Relationship to Orthon Design Principles

This pattern is a **consequence** of several existing Orthon design
principles, not a replacement for them:

| Principle | How it contributes |
|-----------|-------------------|
| **Explicitness** | Invalid states are rejected at the syntax level — no hidden conversions, no implicit defaults |
| **Declarative With Static Guarantees** | The type system proves absence of invalid states at compile time |
| **Semantic Purity** | Each type means exactly one thing; a `"GET" | "POST"` union means exactly those two values |
| **Intent Over Implementation** | The programmer declares *what* is valid; the compiler enforces it |
| **LLM Generability Gate** | A type system that rejects invalid states is a measurable, automatable quality gate for generated code |

The question for Orthon is whether this pattern deserves explicit recognition
**as a named cross-cutting concern** — not as a new design principle (which
would require a Tier 1 EDR to modify `DESIGN_PRINCIPLES.md`), but as a
documented pattern that connects multiple accepted mechanisms and guides future
design decisions toward type-level safety.

## Implications for Orthon

1. **Literal types are the most direct mechanism.** EDR-043 already
   establishes that `"GET" | "POST"` is a closed set and `"GETT"` will not
   compile. This is the entry point for the pattern — the simplest, most
   visible application.

2. **ADTs provide the stronger form.** EDR-039's tagged sum types with
   exhaustive `match` provide compile-time proof that all variants are
   handled. Literal type unions are ergonomic for external-facing
   boundaries; ADTs are the correct choice for internal domain modelling
   where variant identity must be declared.

3. **The pattern should guide future concept evaluation.** When evaluating
   a new concept for Orthon, one question should be: *"Does this concept
   make more illegal states representable, or fewer?"* A concept that
   opens new holes in the type system should carry a heavier burden of
   proof.

4. **LLM Generability Gate synergy.** The gate already validates that an
   LLM can reliably produce correct code. This pattern provides a concrete
   mechanism: if the type system rejects invalid states, the LLM's
   incorrect output fails at compile time rather than at runtime. This is
   faster feedback and cheaper to detect.

5. **Not every invalid state can be eliminated.** Some properties
   (e.g., "this list is sorted", "this integer is a valid port number
   between 1 and 65535") require dependent types or runtime checks.
   Orthon should distinguish between **representable invariants**
   (enforced by the type system) and **verified invariants** (enforced by
   contracts or runtime assertions).

## Open Questions

1. Should "Make Illegal States Unrepresentable" be added as an explicit
   sub-principle under "Declarative With Static Guarantees" in
   `DESIGN_PRINCIPLES.md`? This would require a Tier 1 EDR (Architecture
   category) since `DESIGN_PRINCIPLES.md` is locked. The alternative is to
   keep it as a documented cross-cutting pattern (this file) without
   modifying the locked principles document.

2. How should the pattern interact with Orthon's `CONTRACTS.md` concept?
   Contracts express invariants that the type system cannot capture
   (e.g., "this integer is positive"). The pattern's boundary — what the
   type system can prove vs. what requires runtime verification — should
   be clearly documented.

3. Should Orthon provide a `newtype` or opaque type mechanism to make the
   pattern applicable to single-value wrappers? E.g., `type Port = Int`
   where only values 1..65535 are constructible. This is related to
   `SMART_CONSTRUCTORS.md` (if it exists in research).

4. Does the pattern interact with `REFINEMENT_TYPES.md` (if it exists)?
   Refinement types extend the pattern to value-range constraints
   (`Int where _ > 0`), bridging the gap between type-level and
   contract-level invariants.

## Decision History

Initial research — no decisions recorded yet.

### Cross-References

- [EDR-039 — Algebraic Data Types](../../../how/decision_records/architecture/EDR-039-algebraic-data-types.md)
- [EDR-043 — Literal Types](../../../how/decision_records/architecture/EDR-043-literal-types.md)
- [EDR-045 — Union and Intersection Types](../../../how/decision_records/architecture/EDR-045-union-intersection-types.md)
- [EDR-018 — Null Safety (Option Type)](../../../how/decision_records/architecture/EDR-018-null-safety.md)
- [EDR-028 — Type-Level Null Safety](../../../how/decision_records/architecture/EDR-028-type-level-null-safety.md)
- [DESIGN_PRINCIPLES.md](../../../how/DESIGN_PRINCIPLES.md) — "Declarative With Static Guarantees", "Explicitness"
- [LITERAL_TYPES.md](../important/LITERAL_TYPES.md) — research document for literal types
- [baml-concepts-orthon-gap-analysis.md](../../../notes/baml-concepts-orthon-gap-analysis.md) — gap analysis identifying this as a P0 concern
