# Hypothesis: `then` Keyword in `if ... then ... else`

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Pending design convergence before EDR.
>
> **See also:** `what/SEMANTIC_MODEL.md` § Evaluation,
> `what/GLOSSARY.md` (no `if`/`then` entry — documentation gap),
> `notes/iterator-protocol-discussion.md` (2026-08-05).

## Context

`if cond then expr else expr` is used throughout accepted concept docs
(`SEMANTIC_MODEL.md`, `ERROR_HANDLING.md`, `COMPILE_TIME_EXECUTION.md`,
`COMPOSABLE_COLLECTION_OPS.md`), but there is **no dedicated `if`/`then`
concept and no GLOSSARY entry** for either keyword.

## Problem

Is `then` a keyword? Is `if` a language construct (Level 1/2) or sugar over
`match`/`when`? How does it compose with expression-oriented evaluation?

## Proposal

- `if cond then expr else expr` is the conditional form; `then` is a keyword.
- `if` is expression-oriented (produces a value, per Evaluation).
- Add GLOSSARY entries for `if` and `then`; add a Conditional concept if the
  pipeline classifies `if` as a language construct.

## Open Questions

- Is `then` required, or is `if cond: expr else: expr` sufficient (dropping
  `then`)?
- Does `if` desugar to `match` on `Bool`, or is it primitive control flow?

## Next Step

Design convergence → decision on `if`/`then` → GLOSSARY entries + EDR
(per routing B4).
