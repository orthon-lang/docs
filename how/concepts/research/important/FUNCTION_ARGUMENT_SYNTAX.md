# Hypothesis: Java-Style Function Argument Syntax (Type Before Name)

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Post-acceptance hypothesis from the ITERATOR_PROTOCOL review (2026-08-05).
> Syntax question — Phase 5 scope. Separate from return syntax
> (see `FUNCTION_RETURN_SYNTAX.md`).
>
> **See also:** `what/SYNTAX.md` (Phase 5 placeholder),
> `what/SEMANTIC_MODEL.md` § Mutation (`fun`/`proc`/`new` examples).

## Problem

Argument style: `(a: Int, b: Int)` (name-first) vs `(Int a, Int b)`
(type-first, Java/C style). Which is canonical for Orthon?

## Proposal

Evaluate Java/C-style `(Int a, Int b)` for type-first readability, consistent
with a prefix return type (`FUNCTION_RETURN_SYNTAX.md`).

## Tradeoffs

- **Type-first** `(Int a, Int b)`: reads like a declaration; familiar from
  Java/C/C++.
- **Name-first** `(a: Int, b: Int)`: reads like a binding; familiar from
  Kotlin/Swift/Rust/Python. Orthon currently uses this everywhere.

## Open Questions

- Named and optional parameters interaction (`NAMED_AND_OPTIONAL_PARAMETERS.md`).
- Signature readability with `require`/`using` clauses present.

## Next Step

Phase 5 syntax decision; coupled with `FUNCTION_RETURN_SYNTAX.md`.
