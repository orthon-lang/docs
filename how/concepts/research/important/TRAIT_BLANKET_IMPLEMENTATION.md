# Hypothesis: Trait Blanket Implementation

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Extracted from the `BOUNDS_IN_ANGLE_BRACKETS` review (2026-08-07). Blanket
> implementations are accepted at a basic level (EDR-019,
> `what/concepts/TRAITS.md`), but their syntax and interaction with bounds are
> under-specified. The `where`-elimination hypothesis flags blanket-impl syntax
> as its weakest point, so the topic deserves a dedicated analysis rather than
> being settled as a side effect.
> Syntax question — Phase 5 scope.
>
> **See also:** `../../../../what/concepts/TRAITS.md` (EDR-019),
> `../../../decision_records/architecture/EDR-019-traits.md`,
> `../../../../what/concepts/GENERICS.md`, `../../syntax/BOUNDS_IN_ANGLE_BRACKETS.md`,
> `../../../../what/GLOSSARY.md` § Trait Bound, `../../../DESIGN_PRINCIPLES.md`
> (Parsimony, DRY, Uniformity, Explicitness).

## Problem

A blanket implementation implements a trait for **every** type that satisfies a
given bound, in one declaration, instead of per-type:

- EDR-019 (item 9) accepts blanket implementations as a trait construct, and
  `what/concepts/TRAITS.md` documents a single `where`-clause form — but the
  design space (syntax, multi-bound forms, interaction with the orphan rule,
  negative bounds, LLM generability) is otherwise open.
- The `BOUNDS_IN_ANGLE_BRACKETS` hypothesis proposes moving bounds into `<>`
  and eliminating `where`; blanket impls are its weakest case, so the decision
  deserves its own analysis rather than being settled as a side effect.

## What It Is / Examples

The canonical example is Rust's `ToString` blanket impl — every `Display` type
gains `to_string()`:

```rust
impl<T: Display> ToString for T {}
```

Orthon's accepted form (via `where` clause, EDR-019 / TRAITS.md):

```orthon
impl<T> Printable for T where T as Display
    fun format(self) -> String
        return self.to_display()
```

The bound-first alternative proposed by `BOUNDS_IN_ANGLE_BRACKETS.md`, with the
bound moved into `<>`:

```orthon
impl<Display as T> Printable for T
```

## Implications for Orthon

- **Syntax choice.** Keep the `where` clause, adopt the bound-first `<>` form,
  or support both. This is the crux of the hypothesis: it couples blanket impl
  syntax to the broader `where`-elimination decision.
- **Multi-bound forms.** A blanket can carry several bounds on one parameter:
  `impl<Hash + Eq as T> Printable for T` (bound-first) vs.
  `impl<T> Printable for T where T as Hash + Eq` (current).
- **Orphan rule.** Per EDR-019, a blanket impl is subject to the orphan rule:
  it must live in the same module as the trait being implemented (or the type —
  but for a blanket the type is generic, so the trait's module is the realistic
  home). A blanket in a foreign module is an orphan.
- **Coherence / overlap.** Blanket impls and concrete impls must not overlap;
  the compiler must reject conflicting implementations for the same
  (trait, type) pair. Overlap rules need specification.
- **Negative bounds.** Blanket impls cannot express "any `T` that is NOT `X`"
  automatically; the complement requires a marker trait (`<NonFixedSize as T>`)
  that types explicitly implement. Automatic type-level negation is out of scope
  for v0.1 (no such task), but must be recorded as a boundary.
- **LLM generability.** The accepted `where` form is a familiar, readable shape;
  the bound-first form reads against the param-first intuition and needs an LLM
  generability check (Gate 5) before acceptance.
- **Policy footprint.** Dispatch Policy (static monomorphisation for blanket
  instances) and Coherence Policy (orphan rule, overlap rule).

## Tradeoffs

- **Pro (keep `where`):** familiar and readable; matches the accepted TRAITS.md
  example; low LLM generability risk.
- **Pro (bound-first `<>`):** one home for parameters and bounds; removes the
  second home for the same concept (DRY); uniform with functions, types,
  classes and methods.
- **Con (bound-first):** `impl<Display as T> Printable for T` reads less
  naturally than `where`; a syntax-regression risk for the most common trait
  pattern.

## Open Questions

1. Can blanket-impl bounds always move into `<>`, including multi-bound forms
   (`impl<Hash + Eq as T> Printable for T`)?
2. Is an opt-in marker trait sufficient permanently for negative bounds, or
   will Orthon ever need automatic complement (which `where`-style negation
   would require)?
3. How do blanket impls and concrete impls coexist without coherence
   violations — what overlap rule does Orthon adopt?
4. Where must a blanket impl live under the orphan rule — always the trait's
   module, or also the type's module when the bound narrows the type set?
5. Is the bound-first form generable by LLMs reliably (Gate 5 verification)?

## Next Step

Phase 5 syntax decision, coupled with `BOUNDS_IN_ANGLE_BRACKETS.md`. If the
bound-first direction is adopted, EDR-019 / TRAITS.md blanket-impl syntax is
revised accordingly. If blanket impls ever gain automatic negative bounds, a
`where`-style mechanism or a new construct is required.
