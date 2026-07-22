# Enum Alternatives

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language model a finite set of named, distinct values?

Three approaches with very different trade-offs exist:

1. **Full enum types (Java, Rust, C++, C#, Swift)** — a first-class type with a fixed set of named variants. Java enums are classes with methods and fields; Rust enums are tagged unions (algebraic data types) that can carry data per variant; Swift enums fall somewhere between. Powerful, but heavy: enums have their own syntax, their own type rules, and often interact poorly with other constructs (exhaustive matching requirements, serialization, evolution over time).

2. **Named constants (Go, C)** — a set of typed or untyped integer constants, often with an auto-increment mechanism (Go's `iota`). No special enum type — just constants. Go uses `const` + `iota`:

   ```go
   const (
       StatusActive = iota  // 0
       StatusInactive       // 1
       StatusBanned         // 2
   )
   ```

   Lightweight and simple, but no type safety beyond the integer type. No exhaustiveness checking. No methods on the "enum" type without conventions.

3. **Sum types / algebraic data types** — a type whose values are exactly one of a fixed set of variants, each optionally carrying data. These are more general than enums and overlap with pattern matching. Rust enums are actually sum types.

The core problem: **does the language need a dedicated enum construct, or can named constants (Go) / sum types (Rust) / pattern matching cover the same ground with fewer special cases?**

## Principles

1. **One mechanism** — There should be exactly one way to model a set of distinct named values. Not both enums and constants and sum types with overlapping semantics.

2. **Type safety** — Assigning a value of one enum type to a variable of another should be a compile-time error. Plain integer constants (C-style) do not satisfy this.

3. **Exhaustiveness** — When switching over an enum-like value, the compiler should warn about unhandled variants.

4. **Lightweight syntax** — Defining a set of named values should not require boilerplate. One line per variant, no method declarations for basic usage.

5. **Evolvable** — Adding a new variant should be detectable (compile error at match sites) but not require changes to unrelated code.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Enumeration Policy | Determines whether the language has dedicated enums, uses constants, or relies on sum types |
| Constant Policy | Governs how named constants are declared and typed |
| Pattern Matching Policy | Controls exhaustiveness checking and destructuring |
| Type Safety Policy | Determines whether enum-like values are distinct types or aliases for integers |

## Model (What)

The model describes options rather than prescribing a single answer:

**Option A: Named constants with auto-increment (Go-style)**
```orthon
const Status {
    Active     // = 0
    Inactive   // = 1
    Banned     // = 2
}
```
Lightweight. No methods. No exhaustiveness (unless the compiler specially tracks `const` groups). Simple and predictable.

**Option B: Sum types with data (Rust-style)**
```orthon
type Status {
    Active
    Inactive(since: Date)
    Banned(reason: String, until: Date)
}
```
More expressive. Exhaustiveness checking via pattern matching. Carries data per variant. More complex syntax and semantics.

**Option C: Simple constants (Go-iota-style)**
```orthon
const StatusActive = 0
const StatusInactive = 1
const StatusBanned = 2
```
Minimal mechanism. No type safety beyond the underlying integer type. No exhaustiveness.

## Default Strategy

The default strategy is **Option A: named constants with auto-increment**, similar to Go's `iota` but with a block syntax that groups related constants. The compiler treats the group as a distinct type for exhaustiveness checking.

## Alternative Strategies

1. **Full algebraic enums (Rust/Swift-style)** — Sum types with pattern matching and data per variant. Appropriate if Orthon already has algebraic data types and pattern matching.

2. **No special enum mechanism at all** — Use only plain integer constants. Simplest mechanism, weakest guarantees. Appropriate for a minimal core language where more structured approaches are library-level conventions.

3. **String constants instead of integers** — Use string constants for better debuggability and serialization. Discouraged for performance-sensitive code.

## Open Questions

1. Does Orthon already have pattern matching and sum types? If yes, dedicated enums may be redundant.
2. Should const groups be monomorphic (all same type) or can they mix types?
3. How does the iota-style auto-increment interact with serialization — should values be stable across compiler versions?
4. Should the language support both sum types (for data-carrying variants) and plain enums (for marker variants)?

## Cross-References

- See `PATTERN_MATCHING.md` for exhaustiveness checking and destructuring
- See `ALGEBRAIC_DATA_TYPES.md` for sum types as an alternative to enums
- See `DATA_MODEL.md` for the type hierarchy
- See [`../../why/ZEN.md`](../../why/ZEN.md) for the principle of one concept per syntax
