# Hypothesis: Sequence Methods Apply Directly to Ranges

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Pending design convergence before EDR.
>
> **See also:** `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022),
> `RANGE_STEP.md`, `RANGE_SLICE.md`.

## Context

The 2026-08-05 review: with ranges as data values (`(0..10)`), iterator
combinators (`map`, `filter`, etc.) should apply directly to the range value,
without an explicit `.iter()` step.

## Problem

Can a range enter a combinator chain directly, or only via `for` iteration?

## Proposal

`(0..10)` is a sequence value. Iterator combinators apply directly:

```orthon
(0..10).map(|x| x * 2)
(0..10).filter(|x| x % 2 == 0)
(0..10).step(2).collect()
```

Ranges implement `IntoIterator` and enter combinator chains without `.iter()`.
Chains stay lazy — combinators are lazy by default.

## Open Questions

- Is a `Range` itself a sequence, or only iterable via `IntoIterator`?
- Does this make `Range` interchangeable with `Iterator` in all combinator
  entry points, or only through the `IntoIterator`-based ones?

## Next Step

Design convergence → part of the Range/Slice hypothesis scope (B3).
