# Type-Level Computation

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

TypeScript's Conditional Types (`T extends U ? X : Y`), Mapped Types
(`{ [K in keyof T]: ... }`), Template Literal Types, `keyof`, `typeof` (used
in type position), Indexed Access Types (`T[K]`), and `infer`, plus the
Utility Types built entirely from them (`Partial`, `Pick`, `Omit`, `Record`),
individually look like seven unrelated features. Together they form one
underlying idea: a small, pure, functional programming language that
operates on types instead of values, evaluated entirely at compile time —
with its own conditionals (`extends ? :`), its own mapping/iteration (mapped
types), its own pattern matching and binding (`infer`), and its own string
manipulation (template literal types).

This is a distinct question from [`GENERICS.md`](GENERICS.md). That document
is about parametric polymorphism — writing one function/type that works
across multiple types via trait-bounded type parameters
(`func lookup<K: Hash, V>`). This concept is about *computing a new type as
a function of an existing type's shape* — not "does this type satisfy a
bound" but "derive a new type from another type."

This is also a distinct question from [`GRADUAL_TYPING.md`](GRADUAL_TYPING.md).
That document concerns whether/when type annotations are required and how
typed/untyped boundaries interact, while this concept assumes full static
typing is already in play and concerns the expressive power of the type
language itself once inside it — a separate axis entirely.

TypeScript's type-level language is widely documented as Turing-complete —
community type-level programs have implemented string calculators, JSON
parsers, and even a Game of Life simulation purely inside the type checker.
This is a direct, sharp tension against Orthon's stated minimal-core and
LLM-generability goals, and it must be named explicitly here rather than
glossed over.

The core problem: does Orthon want a Turing-complete (or at least highly
expressive) compile-time type-computation language, a fixed closed set of
built-in type-level utilities (`Partial`/`Pick`/`Omit`-equivalents as
compiler intrinsics with no user-defined type-level functions), or no
type-level computation mechanism at all beyond ordinary generics?

## Examples

| Language | Type-level computation support | Turing-complete? | Typical use |
|---|---|---|---|
| **TypeScript** | Full set — conditional types, mapped types, template literal types, keyof/typeof/indexed access, infer — plus Utility Types built from them | Yes — informally demonstrated by community type-level programs | Deriving DTO/form types from a source type such as `Partial<User>`, string-keyed lookups, validating string literal shapes |
| **Haskell** | Type families, both open and closed, directly parallel conditional/mapped types at the type level | GHC extensions make this Turing-complete | Advanced library-internal type-level programming such as `servant` and `singletons` |
| **Scala 3** | Match types, e.g. `type Elem[X] = X match { case Array[t] => t }`, approximate conditional types | Not typically pushed to full Turing-completeness in practice | Library-internal type transformations |
| **Rust** | Const generics, associated types, and trait-based type-level computation (e.g. the `typenum` crate) approximate some of this, but require explicit trait machinery per computation with no first-class mapped/conditional type syntax | Not Turing-complete in the same lightweight sense, though const generics plus traits can encode significant computation | Compile-time array-size arithmetic |
| **Kotlin/Java/C#/Go** | No type-level computation | No | None — always falls back to runtime reflection or code-generation tooling such as annotation processors or source generators for the same DTO-derivation use case TypeScript solves at the type level |

A conditional type:

```typescript
type IsString<T> = T extends string ? true : false;
```

A mapped type:

```typescript
type Partial<T> = { [K in keyof T]?: T[K] };
```

A template literal type:

```typescript
type Route = `/users/${string}`;
```

The remaining sub-features, in brief:

- **`keyof`** — produces a union of a type's own property-name literal
  types, e.g. `keyof User` yields `"id" | "name" | "email"`.
- **`typeof` (type position)** — lifts the inferred type of a value binding
  back into a type expression, e.g. `type T = typeof someValue`.
- **Indexed Access Types (`T[K]`)** — looks up the type of property `K` on
  type `T`, e.g. `User["email"]` yields `string`.
- **`infer`** — binds a sub-pattern of a type inside a conditional type's
  `extends` clause, enabling destructuring such as extracting a function's
  return type or an array's element type.
- **Utility Types** (`Partial`, `Pick`, `Omit`, `Record`) — standard-library
  types built entirely out of mapped types, conditional types, and `keyof`,
  demonstrating that the "seven features" are really one composable
  substrate.

## Implications for Orthon

1. **Direct tension with Minimal Core** — A Turing-complete type-level
   language is, definitionally, an entire second programming language
   nested inside the type system, with its own semantics, its own
   recursion, and its own failure modes (type-level infinite loops causing
   compiler hangs or stack overflows are a documented TypeScript pain
   point). This is about as far from "minimal core"
   (`why/VISION.md`'s "Small core — all features decompose to a minimal
   primitive set" and the Manifesto's "Minimal core, maximum
   expressiveness") as a feature can get, and must be named as a genuine,
   serious conflict rather than smoothed over.
2. **Direct tension with LLM Generability** — Recursive conditional/mapped
   type definitions require simulating compile-time execution to know what
   a type resolves to; neither an LLM nor a human can determine a derived
   type's shape by local inspection alone, which is exactly the "hidden
   conversion" and "context-dependent rule" class of problem the
   LLM-readiness pillar (`why/VISION.md`'s "Small surface, consistent
   rules" and "Explicit semantics reduce hallucination") exists to
   eliminate.
3. **A closed set of intrinsics fits the philosophy better than a general
   language** — Per the Manifesto's "Extensibility over built-in magic" and
   "Composition over exceptions," if Orthon wants the ergonomic wins
   (deriving a `Partial<T>`/`Pick<T, K>`-equivalent from an existing record
   type without hand-writing a parallel type), the closed-set-of-built-in-
   utilities approach is more consistent with Orthon's philosophy than a
   general, user-extensible type-level language: ship `Partial`, `Pick`,
   `Omit`, `Record` (or equivalents) as fixed compiler intrinsics or stdlib
   types with fixed, documented semantics, rather than exposing
   `keyof`/mapped-type/conditional-type primitives that let users compose
   their own arbitrary type-level programs.
4. **A materially different, additive need from `GENERICS.md`** —
   `GENERICS.md`'s existing trait-bounded parametric polymorphism
   (`func lookup<K: Hash, V>`) already covers "write code that works across
   multiple types" — type-level computation instead answers "derive a
   brand-new type from an existing type's shape," a materially different
   and additive need. Adopting it should be evaluated as an explicit,
   separate extension to `GENERICS.md`'s model, not implied by it.
5. **Macros/derive mechanisms may cover the same need without a second
   computational layer** — If Orthon adopts only the closed-set intrinsic
   approach from Implication 3, it should still study whether that need is
   better served entirely outside the type system — e.g. compile-time
   macros/derive mechanisms already anticipated by
   `ALGEBRAIC_DATA_TYPES.md`'s `@derive` and `STRUCTURAL_TYPING.md`'s
   `@derive(Show, Eq, Clone)` and `METAOBJECTS.md`'s macro/annotation
   research — since a derive-macro model produces the same DTO-shaping
   ergonomics without introducing a second, compile-time-only computational
   language layered on top of the value-level one.

## Open Questions

1. Does Orthon want any type-level computation beyond ordinary generics, or
   does `GENERICS.md`'s trait-bounded model plus `METAOBJECTS.md`'s
   `@derive` mechanism fully cover the legitimate "derive a type from
   another type" use cases TypeScript's Utility Types solve?
2. If some type-level computation is wanted, should it be restricted to a
   small, fixed, non-recursive set of compiler intrinsics (no user-defined
   type-level functions, no `infer`, no recursive conditional types),
   eliminating the Turing-completeness and its associated compiler-hang
   failure mode?
3. How would Orthon's compiler bound type-level computation to guarantee
   termination if any user-extensible mechanism is allowed — a
   recursion-depth limit, as some TypeScript tooling imposes informally?
4. Should the "derive new type from existing type's shape" need be solved
   entirely through macros/derive annotations (`METAOBJECTS.md`) instead of
   a parallel type-level expression language, keeping exactly one mechanism
   for "generate code/types from other code/types" per the Manifesto's "one
   concept, one syntax"?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [GENERICS.md](GENERICS.md)
- [GRADUAL_TYPING.md](GRADUAL_TYPING.md)
- [ALGEBRAIC_DATA_TYPES.md](ALGEBRAIC_DATA_TYPES.md)
- [STRUCTURAL_TYPING.md](STRUCTURAL_TYPING.md)
- [METAOBJECTS.md](METAOBJECTS.md)
