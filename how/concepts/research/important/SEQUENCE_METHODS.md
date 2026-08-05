# Hypothesis: Sequence Methods Apply Directly to Ranges

> **✅ ACCEPTED via [EDR-083](../../../decision_records/architecture/EDR-083-range.md)
> (2026-08-05).** Merged into the RANGE concept — see
> [`what/concepts/RANGE.md`](../../../../what/concepts/RANGE.md) § Iteration and Combinators.
> This document is the reasoning trail; the accepted specification lives in the RANGE concept.
>
> **See also:** `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022),
> `RANGE_STEP.md`, `RANGE_SLICE.md`.

## Context

The 2026-08-05 review: with ranges as data values (`(1..10)`), iterator
combinators (`map`, `filter`, etc.) should apply directly to the range value,
without an explicit `.iter()` step.

## Problem

Can a range enter a combinator chain directly, or only via `for` iteration?

## Proposal (adopted, EDR-083)

`(1..10)` is a sequence value. Iterator combinators apply directly:

```orthon
(1..10).map(|x| x * 2)
(1..10).filter(|x| x % 2 == 0)
(1..10).step(2).collect()
```

Ranges implement `IntoIterator` and enter combinator chains without `.iter()`.
Chains stay lazy — combinators are lazy by default.

## Resolved Questions (2026-08-05, EDR-083)

- A `Range` is a **value type** (a compact descriptor), not itself a `Sequence`
  representation; it is iterable via `IntoIterator[Int]`.
- `Range` is interchangeable with `Iterator` in all combinator entry points that
  accept `IntoIterator`; it does not need to be an `Iterator` itself.

## Next Step

Merged into the RANGE concept (EDR-083) — 2026-08-05.
