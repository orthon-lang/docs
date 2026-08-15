# Type Annotation Syntax

> **Status:** Phase 5 input — language comparison; not a concept under current review.
> Records the `:` type-annotation syntax hypothesis deferred to Phase 5 (Syntax)
> by [`DECLARATION_BY_ASSIGNMENT.md`](../../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)
> (EDR-074). It is a factual language comparison; no binding verdict is made here.
>
> **Last updated:** 2026-08-15

## Issue (Why)

What concrete syntax should Orthon use for explicit type annotations: `x: Type`, `Type x`, or something else?

The declaration model ([`DECLARATION_BY_ASSIGNMENT.md`](../../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)) leaves the annotation syntax to Phase 5 (Syntax). This document records the `:` hypothesis and the alternatives, so the Phase 5 decision has a single reference point.

## The `:` tradition

In mathematics and type theory, `x : T` reads "x has type T" (x belongs to the set/type T). Hindley-Milner typed languages preserved this convention (ML, Haskell, OCaml, F#), and it was later adopted by TypeScript, Swift, Kotlin, Rust, and Python (annotations).

```orthon
count: Int = 42        # "count has type Int"
```

The colon reads as "has type" or "is of type".

## The `Type x` alternative

`int x` comes from C, which inherited it from Algol 60's `integer x` declarations. Historically it served the compiler — the type is visible first, simplifying storage allocation. It is an engineering convenience that became habit.

Note: Fortran is not part of this lineage — Fortran used implicit typing by first letter (I–N), not `Type name`.

```text
# Other languages, for comparison only — not Orthon
integer x    # Algol 60
int x        # C
```

## Colon overload

Outside the type-annotation position, `:` may appear in other constructs (dictionaries, and so on). Ambiguity is resolved by context: in a position where a type is expected, `:` is read as an annotation.

For Orthon, the range conflict is already closed: `0..10:step(2)` was rejected because `:` conflicts with type annotations — ranges never use `:` (EDR-083, see [`RANGE_STEP.md`](../important/RANGE_STEP.md)). Remaining overload considerations (for example, dictionaries, if Orthon adopts a dictionary literal) belong to Phase 5.

## Conclusion (hypothesis, no verdict)

`x: Type` keeps the name first, which suits long identifiers and complex types, and aligns with an inference-first design (see [`TYPE_INFERENCE.md`](../essential/TYPE_INFERENCE.md)). Adopting `Type x` would break uniformity with the ML-derived ecosystem. The final choice is a Phase 5 (Syntax) decision.

## Cross-References

- [`DECLARATION_BY_ASSIGNMENT.md`](../../../what/concepts/DECLARATION_BY_ASSIGNMENT.md) — concept (EDR-074), Phase 5 boundary.
- [`RANGE_STEP.md`](../important/RANGE_STEP.md) — `:` vs range conflict (EDR-083).
- [`TYPE_INFERENCE.md`](../essential/TYPE_INFERENCE.md) — inference strategy couples to annotation syntax.
- [`SHADOWING_SYNTAX.md`](SHADOWING_SYNTAX.md) — companion Phase 5 syntax hypothesis.
