# Lazy Sequence Generators

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-lazy-sequences.md`

## Issue (Why)

Manually implementing an iterator requires writing a stateful class or
object, explicitly defining iteration methods — too much boilerplate
for a simple idea. This forces programmers to write infrastructure code
instead of business logic.

Orthon should eliminate manual iterator-implementation boilerplate by
providing generators (`yield` keyword) and lazy sequences as a core
language feature.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Class with `__iter__` and `__next__`, manual state management | `def gen(): i=0; while True: yield i; i+=1` |
| Java | Anonymous class implementing `Iterator` | `Stream.iterate(0, i -> i+1)` |

## Hypothesis

Orthon can eliminate manual iterator-implementation boilerplate by
providing generators (`yield` keyword) and lazy sequences as a core
language feature — not a library add-on — enabling infinite sequences,
composition without intermediate allocation, and declarative stream
operations.

## Implications for Orthon

- Generators (functions with `yield`) are a basic language element, not
  a library feature.
- Lazy sequences must be lazy by default; materialization must be explicit.
- Stream-like operations (map, filter, take, drop) operate on lazy
  sequences without allocating intermediate collections.
- Infinite sequences are a valid concept the language should not
  artificially block.
- Generator composition — chaining without losing laziness.
- Related concepts: `ITERATOR_PROTOCOL.md`, `EXECUTION_PROGRAM.md`

## Open Questions

- Should `yield` be an expression or a statement?
- How to signal generator completion vs error?
- Relationship with Execution Program model — can a generator be
  executed in a different strategy?

## Decision History

Initial hypothesis derived from `imperative-crutch-lazy-sequences.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/ITERATOR_PROTOCOL.md`
- [ ] `essential/EXECUTION_PROGRAM.md`
- [ ] `important/GENERATORS.md`
- [ ] `what/GLOSSARY.md`
