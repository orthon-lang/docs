# Type Annotation Syntax

> **Status:** Phase 5 input — syntax hypothesis; not a concept under current review.
> Home: [`how/syntax/`](README.md) — established by EDR-087 (2026-08-15).
> Records the `:` type-annotation syntax hypothesis deferred to Phase 5 (Syntax)
> by [`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)
> (EDR-074). It is a factual language comparison; no binding verdict is made here.
>
> **Last updated:** 2026-08-15

## Issue (Why)

What concrete syntax should Orthon use for explicit type annotations: `x: Type`, `Type x`, or something else?

The declaration model ([`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)) leaves the annotation syntax to Phase 5 (Syntax). This document records the `:` hypothesis and the alternatives, so the Phase 5 decision has a single reference point.

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

For Orthon, the range conflict is already closed: `0..10:step(2)` was rejected because `:` conflicts with type annotations — ranges never use `:` (EDR-083, see [`RANGE_STEP.md`](../concepts/research/important/RANGE_STEP.md)). Remaining overload considerations (for example, dictionaries, if Orthon adopts a dictionary literal) belong to Phase 5.

## Decision Pipeline Run

> **Pre-filter result (2026-08-15).** Type annotations already passed the
> Decision Pipeline via EDR-027 / EDR-074; this run tests the *syntax
> hypothesis* only. Per [`DECISION_PIPELINE.md`](../process/DECISION_PIPELINE.md)
> and the "Syntax change" row of `DECISION_VALIDATION.md` § Gate Selection.
> This is provenance, not a binding verdict.

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Choosing the surface syntax for explicit type annotations (`x: Type` vs `Type x`). The semantics are settled (EDR-027: annotations at API boundaries; EDR-074: type from initializer); only the surface form is open. |
| Q2 | Is this a language problem or a library problem? | **Language** — syntax of the language. Not a new feature decision: annotations are already accepted (EDR-027, EDR-074). |
| Q3 | Can it be solved with existing primitives? | N/A — annotations exist as semantics; decomposition (`identifier` + `scope`) is settled in EDR-027. Neither candidate changes primitives. |
| Q4 | Does it violate any Design Principle? | No for `x: Type` — inference-first (name first), Consistency over legacy (ML-derived ecosystem), one concept per syntax (`:` defended by EDR-083 for ranges, EDR-086 for generics bounds), semantics before optimisation (`Type x`'s historical motivation is a compiler convenience). |
| Q5 | Adds new semantics (vs sugar)? | No — surface form of an already-defined binding+type semantic. The `:` reservation is already handled (EDR-083, EDR-086); remaining overload (e.g. dictionaries) is Phase 5 scope, noted in this document. |
| Q6 | Expressible through composition? | Yes — `identifier` (type name) + `scope` (binding), per EDR-027's decomposition. |
| Q7 | Syntactic sugar over primitives? | Yes — both candidates are sugar over the same binding-with-type semantic; hence a Phase 5 (Syntax) decision, not a pipeline decision. |
| Q8 | Optimisation, not semantics? | No — syntax, not an optimisation. (`Type x`'s storage-first origin is an optimisation-era rationale.) |
| Q9 | Backward compatibility? | N/A — pre-v1.0. Forward: `x: Type` reserves `:` (partly already via EDR-083/EDR-086). |
| Q10 | Worth adding at all? | Feature: **yes**, already accepted (EDR-027/EDR-074). Syntax choice: **deferred to Phase 5** — no pipeline blocker; the hypothesis supports `x: Type` consistently with the project's trajectory. |

**Verdict:** Pipeline **PASS** for the underlying feature (already accepted).
No blockers for the syntax hypothesis; it is a Phase 5 (Syntax) decision, not
a Tier 1–2 decision requiring a new EDR. Acceptance path:
[`SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md).

## Conclusion (hypothesis, no verdict)

`x: Type` keeps the name first, which suits long identifiers and complex types, and aligns with an inference-first design (see [`TYPE_INFERENCE.md`](../concepts/research/essential/TYPE_INFERENCE.md)). Adopting `Type x` would break uniformity with the ML-derived ecosystem. The final choice is a Phase 5 (Syntax) decision.

## Cross-References

- [`README.md`](README.md) — syntax hypothesis inbox (decision queue).
- [`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md) — concept (EDR-074), Phase 5 boundary.
- [`RANGE_STEP.md`](../concepts/research/important/RANGE_STEP.md) — `:` vs range conflict (EDR-083).
- [`TYPE_INFERENCE.md`](../concepts/research/essential/TYPE_INFERENCE.md) — inference strategy couples to annotation syntax.
- [`SHADOWING_SYNTAX.md`](SHADOWING_SYNTAX.md) — companion Phase 5 syntax hypothesis.
