# Hypothesis: Java-Style Function Return Type (Prefix)

> **⚠️ DRAFT — This document is a preliminary syntax hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Phase 5 input — syntax hypothesis; not a concept under current review.
> Home: [`how/syntax/`](README.md) — established by EDR-087 (2026-08-15).
> Separate from argument syntax (see `FUNCTION_ARGUMENT_SYNTAX.md`).
>
> **See also:** `../../what/SYNTAX.md` (Phase 5 hub),
> `../../what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md` (EDR-081),
> `../../notes/code-block-semantics.md`.

> **Coupling (EDR-097, provisional).** [EDR-097](../decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md) introduces an optional name in the result slot of a signature (`-> name: Type`). The notation follows the decision of this hypothesis (the return-type position) and is provisional (EDR-097, R5); the notation for several results stays open.

## Problem

The current form places the return type after the whole signature:

```orthon
fun process(order_id: Int) require Database db, Logger log -> Receipt
```

With `require`/`using` clauses the return type **dangles at the end** of the
signature (EDR-081 example).

## Proposal

Java-style prefix return type:

```orthon
fun Receipt process(order_id: Int) require Database db, Logger log
```

This frees the signature tail for `require`/`using` and frees the `->` symbol.

## Tradeoffs

- **Pro:** clean tail; return type never in the way of trailing clauses.
- **Con:** contradicts the `->` return convention used across all accepted
  concepts and the closure type `(T) -> U` (`FUNCTIONS.md`); wide ripple.

## Open Questions

- Coupled with lambda syntax (`notes/code-block-semantics.md`) and the
  match-arm separator inconsistency (`=>` per EDR-025 vs `->` in iterator
  docs — cross-concept conflict).

## Next Step

Phase 5 syntax decision; coupled with `FUNCTION_ARGUMENT_SYNTAX.md`.
