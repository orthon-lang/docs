# Hypothesis: Range / Slice as a Concept Separate from Iterator

> **✅ ACCEPTED via [EDR-083](../../../decision_records/architecture/EDR-083-range.md)
> and [EDR-084](../../../decision_records/architecture/EDR-084-slice.md) (2026-08-05).**
> Split into two accepted concepts — see
> [`what/concepts/RANGE.md`](../../../../what/concepts/RANGE.md) and
> [`what/concepts/SLICE.md`](../../../../what/concepts/SLICE.md).
> This document is the reasoning trail.
>
> **See also:** `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022),
> `what/concepts/SPAN.md`, `what/concepts/ITERATION_LOOP.md` (Open Q1),
> `RANGE_STEP.md`, `SEQUENCE_METHODS.md`.

## Context

The 2026-08-05 review (B3): Range/Slice should have a semantic definition
**separate from the Iterator trait**, not be buried inside ITERATOR_PROTOCOL.
Ranges (`1..10`) and slices (`span[1..3]`) already share the `..` operator.

## Problem

How are ranges and slices defined as first-class values, and how do they relate
to (but stay distinct from) the Iterator protocol?

## Proposal (adopted, EDR-083 + EDR-084)

- `Range` is a value type: `a..b` — **inclusive-inclusive** (`1..N` is N elements).
  `..=` is eliminated (single spelling).
- `range(a, b)` is the named canonical form (Named Before Symbolic); both forms
  are equivalent.
- A **slice** = a Range applied to a random-access composite: `span[1..3]`,
  `items[1..k]`. Slicing never copies (`SPAN.md`); contiguous slices are `Span` views.
- A **strided** range (with step) is non-contiguous → produces an `Iterator`
  or materialised collection, **not** a `Span`.

## Resolved Questions (2026-08-05)

- The range literal is **Language**; the `Range` type and `range(a, b)` named
  form are **StdLib** (LIBRARY_BOUNDARY per EDR-082; EDR-083).
- Interaction with `SPAN.md`: `StrSpan` and multi-dimensional indexing are
  deferred (SPAN Open Q1/Q3; FFI Milestone 8).
- `range(a, b)` / `a..b` serves as an indexing expression wherever the
  `Indexable`-like `@get` protocol applies (INDEXING EDR-082; SLICE EDR-084).

## Next Step

Split into two accepted concepts: RANGE (EDR-083) + SLICE (EDR-084) — 2026-08-05.
