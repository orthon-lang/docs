# Hypothesis: Step as a Method Outside the Range Literal

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Pending design convergence before EDR.
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
2. `0..10` is a **data value**; method calls on it use parens: `(0..10).step(2)`.

## Proposal

```orthon
for i in (0..10).step(2)   # 0, 2, 4, 6, 8
```

`.step(n)` is a method on `Range` returning a strided range value.

## Open Questions

- Does `.step()` return a `Range` (value) or directly an `Iterator`?
- Semantics of `step(0)` (error?) and negative steps (descending direction).
- Does the strided result still implement `IntoIterator` for `for`?

## Next Step

Design convergence → merge into the Range/Slice hypothesis (B3) or a
separate EDR.
