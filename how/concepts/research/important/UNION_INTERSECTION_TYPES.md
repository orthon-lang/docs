# Union & Intersection Types

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

TypeScript's `A | B` (union) and `A & B` (intersection) are structural,
set-theoretic type combinators that operate over arbitrary types — including
anonymous object shapes — with no tag, discriminant, or prior variant
declaration required. Any two types can be combined at the point of use,
simply by writing the combinator between them.

This is a distinct question from [`ALGEBRAIC_DATA_TYPES.md`](ALGEBRAIC_DATA_TYPES.md).
That document's sum types require declaring a named type up front with named
variants, and support exhaustive `match` against an explicit tag baked into
the type's own memory layout. This concept requires no named-type
declaration, no tag, and applies to any two existing types, including types
the union's author does not own.

Intersection types (`A & B`) merge the structural shape of two types into a
new anonymous type with all members of both — a type-level-only operation
with no runtime object created.

Orthon's own research corpus already assumes a union combinator exists
without defining it: [`DYNAMIC_COLLECTIONS.md`](DYNAMIC_COLLECTIONS.md)
describes heterogeneous collections "via explicit union types
(`List[Int | String]`)" and uses `Int | String` directly in a code sample,
but never specifies whether this is a structural combinator over arbitrary
types or shorthand for an already-declared sum type. That gap is exactly
what this file exists to close.

The core problem: should Orthon have a structural union/intersection type
combinator distinct from `ALGEBRAIC_DATA_TYPES.md`'s nominal tagged sum
types, or should tagged sum types remain the sole "one of several"
mechanism, with untagged structural unions rejected as a duplicate concept
per the Manifesto's "One concept — one syntax"?

## Examples

| Language | Union mechanism | Intersection mechanism | Tag/discriminant required | Narrowing mechanism |
|---|---|---|---|---|
| **TypeScript** | `A \| B` — structural union | `A & B` — structural intersection/merge | No — though the "discriminated union" pattern optionally adds one manually | `typeof`/`instanceof`/`in` checks and control-flow analysis |
| **Scala 3** | Native `A \| B`, structural | Native `A & B`, structural | No | Pattern matching / type tests, mirroring TypeScript |
| **Python** | `A \| B` via PEP 604 / `typing.Union[A, B]`, structural | No intersection type support — `Protocol` multiple inheritance is the closest workaround | No | `isinstance` checks |
| **Rust** | No native union types; sum types (`enum`) require named variants instead | No intersection type — trait bounds `T: A + B` are the closest analogue for "must satisfy A and B," but that is a generic constraint on a type parameter, not a value-level type combinator | Yes — sum types carry an explicit discriminant | Exhaustive `match` on the enum's declared variants |
| **Kotlin** | No union types — sealed classes are the idiomatic alternative | Intersection types exist only implicitly via generic bounds `<T> where T : A, T : B` | Sealed classes carry an implicit tag | Exhaustive `when` over the sealed hierarchy |
| **Java/C#** | Neither union nor intersection types exist as language features | Neither union nor intersection types exist as language features | N/A | N/A |

A TypeScript union type:

```typescript
type ID = string | number;

function printId(id: ID) {
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    } else {
        console.log(id.toFixed(2));
    }
}
```

A TypeScript intersection type:

```typescript
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;
```

No runtime object or class is created by either declaration — both are
purely compile-time type combinators. `ID` and `Person` exist only to the
type checker; nothing is allocated, instantiated, or tagged at runtime.

## Implications for Orthon

1. **`DYNAMIC_COLLECTIONS.md`'s undefined `Int | String` usage** — Orthon
   must decide whether `Int | String` in `DYNAMIC_COLLECTIONS.md`'s
   heterogeneous-collection example invokes a real structural union
   combinator or is informal shorthand for an already-declared sum type.
   This file recommends the underlying semantics be made explicit during
   Concept Design Review before `DYNAMIC_COLLECTIONS.md`'s example is
   treated as settled syntax.
2. **Risk to "One concept — one syntax"** — Structural unions of arbitrary
   types (including anonymous object shapes) risk violating the Manifesto's
   "One concept — one syntax." Orthon already has `ALGEBRAIC_DATA_TYPES.md`'s
   tagged sum types as the canonical "this OR that" mechanism, and
   introducing a second, structurally-typed union combinator alongside it
   creates two ways to express "one of several types," which the Language
   Design Gate's Orthogonality check should scrutinize directly.
3. **Exhaustiveness becomes harder to guarantee** — If Orthon adopts
   structural unions, exhaustiveness checking — a core promise of
   `ALGEBRAIC_DATA_TYPES.md` ("the compiler enforces this") — becomes harder
   to guarantee: an untagged `Int | String` union relies on runtime
   narrowing (`SMART_CAST.md`-style `is` checks) rather than
   pattern-matching a declared tag, and the compiler cannot warn "new
   variant added, match not updated" the way it can for nominal ADTs.
4. **Intersection types conflict with the composition story in kind, not
   degree** — Intersection types conflict with Orthon's existing
   composition story (`COMPOSITION_OVER_INHERITANCE.md`) in kind, not
   degree: those describe runtime/instance-level composition of behaviour,
   while `A & B` is a purely type-level combinator merging structural shapes
   with no runtime object. If Orthon's data model is Data First /
   value-semantics (`DATA_MODEL.md`), a structural intersection combinator
   may be redundant with simply declaring a new product type/record
   carrying all fields of both.
5. **LLM Generability cost of an open-ended algebra** — Per the LLM
   Generability pillar (`why/VISION.md` "Small surface, consistent rules"),
   an open-ended structural union/intersection algebra over arbitrary types
   multiplies the number of shapes an LLM must reason about at a call site —
   every function accepting `A | B` requires the LLM to correctly
   branch/narrow both cases, and the compiler cannot exhaustiveness-check
   unless the union is closed and tagged. Recommend either omitting general
   structural unions from the core, or restricting them to a narrow set of
   built-in cases (e.g. `T | None` for optionality, already covered by
   `NULL_SAFETY.md`'s `Option[T]`), leaving the general "one of several
   named forms" need to `ALGEBRAIC_DATA_TYPES.md` exclusively.

## Open Questions

1. Is `Int | String` in `DYNAMIC_COLLECTIONS.md`'s heterogeneous-collection
   example meant to invoke a real structural union type, or a stand-in for
   "some ADT," and should that document be updated once this is decided?
2. Should Orthon support structural union types at all, or should
   `ALGEBRAIC_DATA_TYPES.md`'s nominal tagged sum types remain the sole
   "sum" mechanism, with untagged unions rejected as redundant?
3. If structural unions are adopted, should they be restricted to closed
   sets of concrete named types (improving exhaustiveness), or fully open
   to any structural shape (TypeScript's model)?
4. Does Orthon need intersection types at all, or does declaring a new named
   type composing existing types' fields (already supported via
   records/product types) fully cover the same need without a separate `&`
   combinator?
5. If narrowing over unions is supported, how does it interact with
   `SMART_CAST.md`'s existing narrowing rules — does an untagged union
   require the same "effectively-immutable" precondition?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [ALGEBRAIC_DATA_TYPES.md](ALGEBRAIC_DATA_TYPES.md)
- [STRUCTURAL_TYPING.md](STRUCTURAL_TYPING.md)
- [SMART_CAST.md](SMART_CAST.md)
- [DYNAMIC_COLLECTIONS.md](DYNAMIC_COLLECTIONS.md)
- [NULL_SAFETY.md](NULL_SAFETY.md)
- [COMPOSITION_OVER_INHERITANCE.md](COMPOSITION_OVER_INHERITANCE.md)
