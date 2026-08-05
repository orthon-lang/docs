# Hypothesis: Step as a Method Outside the Range Literal

> **✅ ACCEPTED via [EDR-083](../../../decision_records/architecture/EDR-083-range.md)
> (2026-08-05).** Merged into the RANGE concept — see
> [`what/concepts/RANGE.md`](../../../../what/concepts/RANGE.md) § Step.
> This document is the reasoning trail; the accepted specification lives in the RANGE concept.
>
> **See also:** `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022),
> `what/concepts/ITERATION_LOOP.md` (EDR-053), `RANGE_SLICE.md`.

## Problem

Range step syntax is inconsistent across accepted docs:
- `0..10:step(2)` (ITERATOR_PROTOCOL) — `:` conflicts with type annotation.
- `0..10.step(2)` (ITERATION_LOOP) — reads as if `step` binds to `10`.

Proposed alternates fail: `0..3..10` is parse-ambiguous (associativity);
`0.2.10` collides with float literals (`0.2`).

## Accepted Constraints (2026-08-05)

1. Step lives **outside** the range literal — `:` is never used in ranges.
2. A range is a **data value**; method calls on it use parens: `(1..10).step(2)`.

## Proposal (adopted, EDR-083)

```orthon
for i in (1..10).step(2)   # 1, 3, 5, 7, 9
```

`.step(n)` is a method on `Range` returning a strided range value.

## Resolved Questions (2026-08-05, EDR-083)

- `.step()` returns a `Range` (strided value), **not** an `Iterator` — it remains a data value.
- `step(0)` is a compile-time error.
- Negative steps iterate in descending direction (`(1..5).step(-1)` → 5, 4, 3, 2, 1).
- The strided result still implements `IntoIterator`, so `for` and combinators accept it.
- Under the inclusive-inclusive norm the canonical spelling is `(1..10).step(2)`, not `0..10`.

## Next Step

Merged into the RANGE concept (EDR-083) — 2026-08-05.
