# Composable Collection Operations

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-collections-loops.md`

## Issue (Why)

Manual index-based loops, empty accumulator lists, and explicit flags for
search force the programmer to describe *how* to iterate instead of *what*
to obtain. This pattern duplicates business logic with iteration mechanics
and defers type errors to runtime.

Orthon should eliminate index-based and accumulator-based loops as a
primary pattern by providing declarative, composable collection operations.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `for i in range(len(lst)):` | `for i, item in enumerate(lst):` |
| Python | Empty list + loop + `append` | `[x*2 for x in data if x>0]` |
| Java | Loop with `add()` inside | `stream().filter(...).map(...).collect(...)` |
| Java | Loop + `break` + `found` flag | `stream().filter(cond).findFirst().orElse(null)` |

## Hypothesis

Orthon can make the explicit `for` loop a rarely needed escape hatch by
providing declarative, composable collection operations (map, filter,
reduce, find, enumerate) as built-in language constructs — either via a
standard-library protocol or comprehension syntax.

## Implications for Orthon

- The language should provide composable high-level collection operations
  as part of the standard library, not force handwritten loops.
- Indexed iteration is a special case, not the primary pattern.
- Transformation chains should be declarative and lazy where possible.
- The compiler should check types at compile time, not at runtime.
- Related concepts: `ITERATOR_PROTOCOL.md`, `essential/DATA_MODEL.md`

## Open Questions

- Should comprehensions be syntactic sugar for the protocol or a separate
  feature?
- How to handle early exit (break/return) in declarative chains?

## Decision History

Initial hypothesis derived from `imperative-crutch-collections-loops.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/ITERATOR_PROTOCOL.md`
- [ ] `essential/DATA_MODEL.md`
- [ ] `what/GLOSSARY.md`
