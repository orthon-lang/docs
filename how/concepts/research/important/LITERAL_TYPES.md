# Literal Types

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

TypeScript lets a specific concrete value — a string like `"GET"`, a number
like `42`, a boolean like `true` — be a type unto itself, inhabited only by
that exact value. These literal types compose with union types into
lightweight closed sets, e.g. `type Method = "GET" | "POST" | "PUT"`, as an
alternative to a dedicated enum construct.

This is a distinct question from [`ENUM_ALTERNATIVES.md`](ENUM_ALTERNATIVES.md).
That document compares three named strategies for modelling "a finite set of
named values" (full enum types, named constants with auto-increment, sum
types/algebraic data types) but never itself describes literal types as a
fourth option, nor explains the more fundamental "value as type" mechanism a
`"GET" | "POST" | "PUT"` union relies on. This file exists to name that
mechanism directly.

This is also directly related to
[`UNION_INTERSECTION_TYPES.md`](UNION_INTERSECTION_TYPES.md): that file's
Issue concerns combining *existing* types (`Int | String`); this file's
Issue concerns a single, specific *value* becoming its own type, which is
then composed via that file's union mechanism into a closed set.

The core problem: should Orthon let literal values act as their own types,
and if so, are literal types widened automatically (inferred as
`String`/`Int`/`Bool`) or preserved verbatim depending on binding context, as
in TypeScript's `let` (widens) vs. `const`/`as const` (preserves)
distinction?

## Examples

| Language | Literal type support | Widening behaviour | Compose into closed union | Typical use |
|---|---|---|---|---|
| **TypeScript** | String/number/boolean/enum-member literal types | `let` widens to the base type unless `as const` or an explicit annotation preserves the literal | Via `\|` into closed unions | HTTP methods, discriminated-union tags, config option strings |
| **Python** | `typing.Literal["GET", "POST"]` wraps specific values as a type explicitly | No automatic widening — Python's type system is optional/gradual | Via `Union[Literal["GET"], Literal["POST"]]` or the `Literal["GET", "POST"]` shorthand | The same string-enum stand-in for JSON-facing APIs |
| **Rust** | No literal types as first-class citizens usable in a signature | N/A | N/A — closest analogues are `const` generics (compile-time integer/bool values as type parameters) and literal patterns in `match` | Const generics for array sizes, not general closed-set modelling |
| **Kotlin/Java/C#** | No literal types | N/A | N/A | None — always falls back to enum or sealed class |
| **Swift** | No general literal types | N/A | N/A | Enums with raw values are the idiomatic mechanism instead |

A TypeScript literal-type union:

```typescript
type Method = "GET" | "POST" | "PUT";

function request(method: Method) {
    // method is narrowed to exactly one of the three literal strings
}
```

Widening contrast:

```typescript
let x = "GET";    // type string (widened)
const y = "GET";  // type "GET" (literal preserved)
```

## Implications for Orthon

1. **A fourth answer `ENUM_ALTERNATIVES.md` does not enumerate** — Ground
   this in `ENUM_ALTERNATIVES.md`'s "Option A: named constants" and its
   stated core problem ("does the language need a dedicated enum construct,
   or can named constants / sum types cover the same ground"). Literal
   types are a fourth genuinely distinct answer that `ENUM_ALTERNATIVES.md`'s
   Model section does not enumerate. Recommend `ENUM_ALTERNATIVES.md` be
   revisited during Concept Design Review to either fold literal-type
   unions in as an explicit additional option or reject them outright.
2. **Risk of three overlapping closed-set mechanisms** — Per the
   Manifesto's "One concept — one syntax," Orthon should be wary of
   shipping three overlapping closed-set mechanisms simultaneously
   (`ENUM_ALTERNATIVES.md`'s named constants, `ALGEBRAIC_DATA_TYPES.md`'s
   tagged sum types, and literal-type unions). Each solves "a fixed set of
   distinct values" with different ergonomics and different guarantees, and
   the minimal-core principle (`why/VISION.md` "Small core — all features
   decompose to a minimal primitive set") argues for picking one mechanism,
   not three.
3. **Widening rule must be one explicit, always-applicable rule** — If
   Orthon adopts literal types, the widening-vs-preserve question must have
   one explicit, always-applicable rule rather than TypeScript's
   context-dependent inference (mutable `let` widens, `const`/`as const`
   does not). A context-dependent widening rule is exactly the "hidden
   conversion" and context-dependent-rule class of problem
   `why/VISION.md`'s LLM Generability section warns against ("no hidden
   conversions to forget").
4. **Value depends on the union-combinator decision** — Literal types
   compose with `UNION_INTERSECTION_TYPES.md`'s union combinator to form
   closed sets (`"GET" | "POST"`). If Orthon rejects general structural
   unions (per that file's Implication 2), literal types alone provide
   little value, since their primary use case is exactly this composition;
   the two decisions should be made together, not independently.
5. **Weaker exhaustiveness than tagged sum types** — A literal-type
   approach to "closed set of distinct values" provides weaker
   exhaustiveness guarantees than `ALGEBRAIC_DATA_TYPES.md`'s tagged sum
   types — a string literal union has no notion of "all variants" beyond
   the union's syntactic membership, and adding a new arm silently changes
   the type rather than requiring an explicit new variant declaration.
   Orthon should weigh this against the ergonomic win (no declaration
   ceremony) before allowing literal-type unions into the core language.

## Open Questions

1. Should Orthon give literal values (string/int/bool) their own singleton
   type at all, or should every literal always widen immediately to its
   base type, closing off the TypeScript-style closed-string-union idiom
   entirely?
2. If literal types are adopted, should widening be governed by the binding
   form — Orthon already distinguishes `=` mutable vs. `:=` immutable
   bindings per `GRADUAL_TYPING.md` — rather than TypeScript's opt-in
   `as const`?
3. Does adopting literal types make `ENUM_ALTERNATIVES.md`'s Option A
   (named constants) redundant, or do the two serve genuinely different
   needs (external string/JSON-facing values vs. internal enumerations)?
4. Should literal types be restricted to primitive scalars (string/int/bool)
   only, matching TypeScript, or could Orthon extend the concept to other
   literal forms?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [ENUM_ALTERNATIVES.md](ENUM_ALTERNATIVES.md)
- [UNION_INTERSECTION_TYPES.md](UNION_INTERSECTION_TYPES.md)
- [ALGEBRAIC_DATA_TYPES.md](ALGEBRAIC_DATA_TYPES.md)
- [GRADUAL_TYPING.md](GRADUAL_TYPING.md)
