# Hypothesis: Mutable Iteration via `proc` Receiver-Type Dispatch

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Pending design convergence (CONCEPT_PIPELINE.md Stage 10, Type B) before EDR.
>
> **See also:** `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022),
> `what/concepts/TRAITS.md` (Open Q5), `what/SEMANTIC_MODEL.md` § Mutation.

## Context

`what/concepts/ITERATOR_PROTOCOL.md` currently expresses mutable iteration as a
separate method name `collection@iter_mut()` (Rust convention). The 2026-08-05
review raised that this conflicts with Orthon's `fun`/`proc`/`new` declaration
kinds and that two same-name methods (`fun iter` / `proc iter`) on one type
cannot coexist (return type is not part of method resolution).

## Accepted Constraint (2026-08-05)

`IntoIterator[T]` is implemented as an **immutable iterator** only:
`fun iter(self) -> Iterator[&T]`.

## Problem

How does a collection provide mutable iteration without a same-name
`fun`/`proc` method conflict and without a separate `iter_mut()` method name?

## Proposal

Mutable iteration is expressed via the **receiver type**, not the method name.
A collection type has two `IntoIterator` implementations:

```orthon
impl IntoIterator for &Collection      # shared borrow
    fun iter(self) -> Iterator[&T]     # element &T

impl IntoIterator for &mut Collection  # exclusive borrow
    fun iter(self) -> Iterator[&mut T] # element &mut T
```

`Iterator[T]::next()` is declared `proc` (it advances the cursor — mutates
`self`). This is the answer to TRAITS.md Open Q5: a trait may declare a `proc`
method; calls to it require exclusive access, which the borrow checker enforces.

## Open Questions

- Does receiver-type dispatch respect the coherence rule (at most one impl per
  type)? `&Collection` vs `&mut Collection` are distinct types — no violation.
- Is `iter_mut()` removed entirely, or kept as sugar for `&mut`-receiver dispatch?
- Interaction with ownership: does `for x in &mut collection` correctly borrow?

## Next Step

Design convergence → EDR amendment (refines/supersedes EDR-022) + update
`SEMANTIC_MODEL.md` § Mutation if needed.
