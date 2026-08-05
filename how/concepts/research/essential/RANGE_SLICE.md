# Hypothesis: Range / Slice as a Concept Separate from Iterator

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Pending design convergence before EDR.
>
> **See also:** `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022),
> `what/concepts/SPAN.md`, `what/concepts/ITERATION_LOOP.md` (Open Q1),
> `RANGE_STEP.md`, `SEQUENCE_METHODS.md`.

## Context

The 2026-08-05 review (B3): Range/Slice should have a semantic definition
**separate from the Iterator trait**, not be buried inside ITERATOR_PROTOCOL.
Ranges (`0..10`) and slices (`span[1..3]`) already share the `..` operator.

## Problem

How are ranges and slices defined as first-class values, and how do they relate
to (but stay distinct from) the Iterator protocol?

## Proposal

- `Range` is a value type: `a..b` (exclusive end), `a..=b` (inclusive end).
- `range(a, b)` is the named canonical form (Named Before Symbolic); both forms
  are equivalent.
- A **slice** = a Range applied to a Span: `span[1..3]`, `span[range(1, 3)]`.
  Slicing never copies (`SPAN.md`).
- A **strided** range (with step) is non-contiguous → produces an `Iterator`
  or materialised collection, **not** a `Span`.

## Open Questions

- Is the `Range` literal built-in or a StdLib constructor? (ITERATION_LOOP Open Q1)
- Interaction with `SPAN.md`: `StrSpan`, multi-dimensional indexing.
- Does `range(a, b)` / `a..b` also serve as an indexing expression everywhere,
  or only in `span[...]`?

## Next Step

Design convergence → separate EDR for Range/Slice (per routing B3).
