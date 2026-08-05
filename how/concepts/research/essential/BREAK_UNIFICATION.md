# Hypothesis: `break` ≡ `break <unit>` — Unify Bare and Valued Break

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Routing A3: design convergence → EDR after the Decision Pipeline.
>
> **See also:** `what/concepts/ITERATION_LOOP.md` (EDR-053, Open Q2),
> `what/concepts/UNION_INTERSECTION_TYPES.md` (EDR-028),
> `notes/iterator-protocol-discussion.md` (2026-08-05).

## Problem

`break` (bare) and `break value` are two forms of one construct — a controlled
overload. ITERATION_LOOP Open Q2 asks whether `break value` should extend to
`for`/`while`. A bare `loop:` used as an expression has an ill-defined value.

## Proposal

Unify: every `break` carries a value; bare `break` ≡ `break Void` (unit type).

```orthon
let result = loop:
    let item = queue.receive()
    if is_valid(item) then break item   # type T
    # a bare `break` would yield Void
```

- `loop:` with bare `break` → `Void`
- `loop:` with `break item` → `T`
- Mixed bare + valued breaks → either a **type error** (Rust behaviour) or
  `Void | T` via union types (EDR-028).

## Tradeoffs

- Removes the `break` overload; the loop-as-expression type is well-defined.
- Mixed-break typing needs an explicit decision: error vs union.

## Next Step

Decision Pipeline → EDR (after pipeline, per routing A3).
