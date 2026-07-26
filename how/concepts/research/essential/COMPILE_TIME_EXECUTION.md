# Compile-Time Execution (Unified Comptime)

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

Should generics, duck-typed polymorphism, reflection, and metaprogramming each
get their own dedicated language mechanism, or should a single compile-time
execution model serve all four needs at once?

Zig answers this question by collapsing all four into one homogeneous
mechanism: `comptime`. Any function parameter can be declared `comptime`,
including parameters whose declared type is `type` itself — so
`fn max(comptime T: type, a: T, b: T) T` is an ordinary function definition,
not special generic syntax. There is no `<T>` bracket syntax, no separate
generic-parameter list, and no declared trait/interface bounding what `T`
must support. Constraint checking is deferred entirely to instantiation: the
compiler substitutes the concrete type and simply compiles the function body
as written. If the body calls `.eq()` on a `T` value and the substituted type
has no `eq` method, the resulting compile error points at the actual call
site inside the generic function body, not at a mismatch against a
separately declared contract.

The same `comptime` keyword also drives Zig's reflection story — `@typeInfo(T)`,
`@field(value, name)`, and `@hasDecl(T, name)` are ordinary function calls
evaluated at compile time, not a separate reflection API — and its
metaprogramming story: code that generates code is just ordinary
`if`/`for`/`while` control flow running at compile time. There is no macro
language, no AST manipulation, no annotation processor, and no
`@derive`-style declarative code-generation system layered on top.

This repository already has four documents proposing solutions for one
slice of this problem space each:

- [`GENERICS.md`](GENERICS.md) — trait-bound, monomorphised generics (a
  declared `trait`/`impl` contract checked before instantiation).
- [`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md) — structural trait
  satisfaction, still against a declared `trait` contract, just without a
  mandatory explicit `impl`.
- [`METAOBJECTS.md`](METAOBJECTS.md) — traits, annotations, and macros as
  three distinct mechanisms for cross-cutting behaviour and code generation.
- [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md) — reified
  generics restricted to inline functions, plus `@derive` macros, plus
  sealed-types-and-pattern-matching, as three further distinct mechanisms.

Each of those four documents already proposes a solution for one slice of
this problem space, but none of them evaluates the alternative of collapsing
all of generics, duck-typed constraint checking, reflection, and
metaprogramming into the single mechanism Zig demonstrates: one execution
phase (compile time), one syntax (ordinary function/control-flow syntax),
one data kind that is new relative to runtime (`type` as a first-class
value), and zero additional declarative sublanguages — no trait-bound syntax
needed, no macro syntax needed, no annotation syntax needed.

The core problem statement: should Orthon adopt Zig's single unified
compile-time execution mechanism instead of (or alongside) the four separate
mechanisms already proposed elsewhere in this research corpus? And if the
unified approach is preferred, does it strictly subsume the other four
documents' proposals, or does it leave gaps — coherence guarantees,
structural-satisfaction ergonomics, IDE/LLM discoverability of "what a
generic function actually requires" — that the four separate mechanisms
currently handle better?

## Examples

| Language | Generic constraint mechanism | Constraint checked against | Reflection mechanism | Metaprogramming mechanism | Number of distinct sublanguages |
|---|---|---|---|---|---|
| **Zig** | `comptime T: type` parameter | No declared constraint — checked against the function body's actual usage at instantiation time | `@typeInfo`/`@field`/`@hasDecl` as ordinary compile-time function calls | Ordinary `if`/`for`/`while` executed at compile time | **One** — `comptime` is the only sublanguage; identical syntax to runtime code |
| **Rust** | Trait bounds (`where T: Ord`) | A declared trait | `std::any::TypeId` — limited, mostly opt-in | Procedural macros with their own token-stream/AST API (`syn`, `quote`) | **Three** — trait syntax, a separate macro-definition API, and ordinary code |
| **Java** | Generics via type erasure, `extends`/`super` bounds | A declared interface | `java.lang.reflect` runtime API | Annotation processors with their own API | **Four** — generics syntax, reflection API, annotation syntax, and ordinary code |
| **Kotlin** | Generic constraints via `where` | A declared interface | `KClass` reflection, plus `reified` type parameters in `inline` functions as a special case | Compiler plugins (`kotlinx.serialization`) with their own API | **Four** — generic syntax, reflection API, `reified`/`inline` special case, and compiler plugin API |
| **C++** | Templates, duck-typed structurally like Zig pre-C++20; C++20 `concepts` reintroduce a declared-constraint sublanguage | Structural (pre-C++20) or a declared `concept` (C++20+) | None first-class until the C++26 reflection proposal — itself a new sublanguage | Template metaprogramming historically uses the type system itself as an ad-hoc Turing-complete sublanguage, distinct from ordinary runtime code | **Three or four**, depending on `concepts`/reflection adoption |

### Zig sample — duck-typed `comptime` generic

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}
```

No declared bound on `T`. If the substituted type does not support `>`, the
compile error surfaces at the `a > b` call site inside `max`'s body.

### Rust sample — declared trait-bound generic

```rust
fn max<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}
```

[`GENERICS.md`](GENERICS.md)'s Default Strategy already chose the Rust-style
trait-bound approach for Orthon, so this file's purpose is to make the road
not taken — and its trade-offs — explicit for the eventual Concept Design
Review, not to silently override that existing choice.

## Implications for Orthon

1. **Unification vs. four independent convergences.** A single unified
   compile-time execution mechanism directly serves the Manifesto's "One
   concept — one syntax" and Minimal Core principles better, in the
   abstract, than four separate mechanisms for four related needs. But
   [`GENERICS.md`](GENERICS.md), [`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md),
   [`METAOBJECTS.md`](METAOBJECTS.md), and
   [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md) were each
   drafted independently and have already converged on a *different*
   answer — declared trait bounds, monomorphisation, and a small number of
   well-scoped sublanguages. Adopting the unified `comptime` model now would
   mean revising four existing DRAFT documents' Default Strategy sections at
   Concept Design Review time, not merely adding a fifth.

2. **Discoverability trade-off.** The core trade-off Zig accepts, and the
   four existing documents implicitly reject, is discoverability. With
   declared trait bounds ([`GENERICS.md`](GENERICS.md)'s model), a reader —
   human or LLM — can see a generic function's full contract without
   reading its body. With Zig-style duck-typed `comptime`, the contract is
   implicit in the function body and only surfaces as a compile error at a
   specific call site. This cuts directly against the LLM Generability
   pillar's "Small surface, consistent rules" framing in
   [`why/VISION.md`](../../../why/VISION.md), since an LLM generating a call
   to a `comptime`-generic function cannot determine its requirements
   without reading — and correctly interpreting — the function body, rather
   than a declared signature.

3. **Reflection-collapse is a genuinely simpler answer.** The
   reflection-collapse half of `comptime` (`@typeInfo`, `@field`, `@hasDecl`
   as ordinary compile-time function calls rather than a separate reflection
   API) is a genuinely orthogonal-and-simpler answer to the same problem
   [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md) already
   frames — "should Orthon have any runtime reflection, or should all
   metaprogramming needs be served by compile-time mechanisms." It is worth
   weighing against that document's `reified`-generics-in-inline-functions
   special case, since a single `comptime` phase applicable to any function
   eliminates the "only inside inline functions" restriction.

4. **Macro-collapse is a different, simpler answer than Homoiconicity's two
   options.** The macro-collapse half of `comptime` (ordinary
   `if`/`for`/`while` at compile time, no macro language, no AST
   manipulation) is a different, arguably simpler answer than
   [`HOMOICONICITY.md`](HOMOICONICITY.md)'s two modelled options — full
   Lisp-style code-as-data macros, or Orthon's Schema-Provider-for-LLM-
   queries approach. `comptime` needs neither code-as-data nor a schema
   query API for code generation, because it just runs the same language
   twice — once at compile time, once at runtime. This file's Open
   Questions should be cross-checked against
   [`HOMOICONICITY.md`](HOMOICONICITY.md)'s Decision History once that
   document reaches Concept Design Review.

5. **If the existing four-mechanism approach is re-affirmed, it must be a
   deliberate rejection, not an omission.** If Orthon's Concept Design
   Review ultimately re-affirms the existing four-mechanism approach —
   declared trait bounds for generics, a scoped reflection subset, traits
   for behaviour, `@derive` for common derivations — this file's role is to
   ensure that decision is made *with* the unified alternative on the table
   and explicitly rejected for stated reasons (most likely: discoverability
   and IDE/LLM tooling support for "what does this generic function
   require," which declared contracts provide and duck-typed `comptime`
   does not) — not simply never considered.

## Open Questions

1. Should Orthon's generics use declared trait bounds (as
   [`GENERICS.md`](GENERICS.md)'s Default Strategy currently specifies), or
   duck-typed `comptime`-style constraint checking at instantiation time, or
   a hybrid where trait bounds are optional annotations that narrow — but do
   not replace — instantiation-time duck-typed checking?

2. If a unified compile-time execution mechanism is adopted, does it
   strictly subsume [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md)'s
   `reified`-generics-in-inline-functions proposal, or does that proposal
   solve a narrower, still-useful problem (reflection inside non-generic
   inline functions) that a general `comptime` parameter does not address?

3. How would LLM-facing tooling (the Schema Provider, `LLM_NATIVE_TOOLCHAIN.md`)
   present a `comptime`-generic function's actual requirements to an LLM if
   those requirements are implicit in the function body rather than
   declared in the signature — would this require the compiler to
   synthesize and expose an inferred constraint set, effectively re-deriving
   a declared-trait-like contract for tooling purposes even if the source
   syntax doesn't require one?

4. Does allowing `type` as an ordinary first-class parameter value (rather
   than a special generic-parameter syntax) interact safely with Orthon's
   Data Model and Data Modifier abstractions in `DATA_MODEL.md`, or does it
   require treating `type` as a distinct kind of value with its own rules?

5. Should compile-time-only code (functions or branches that only ever
   execute with `comptime` arguments) be visibly marked in Orthon's syntax,
   the way Zig requires the `comptime` keyword at each call site, to
   preserve the Explicit Semantics pillar's requirement that a reader can
   tell from local syntax whether code runs at compile time or runtime?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [GENERICS.md](GENERICS.md)
- [STRUCTURAL_TYPING.md](STRUCTURAL_TYPING.md)
- [METAOBJECTS.md](METAOBJECTS.md)
- [REFLECTION_ALTERNATIVES.md](REFLECTION_ALTERNATIVES.md)
- [HOMOICONICITY.md](HOMOICONICITY.md)
