# Declarative Multi-Key Sort

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-sorting.md`

## Issue (Why)

Sorting by multiple fields requires manually writing comparators with
comparison chains — code becomes verbose and fragile. Java pre-Java 8
required anonymous `Comparator` classes with hand-written comparison
logic for even simple multi-field sorts.

Orthon should eliminate manual comparator chains by providing built-in
lexicographic comparison for tuples and records.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `cmp_to_key`, complex conditional constructs | `sorted(list, key=lambda x: (x.a, x.b))` |
| Java | Anonymous `Comparator` with manual compare chain | `Comparator.comparing(Obj::getA).thenComparing(Obj::getB)` |

## Hypothesis

Orthon can eliminate manual comparator chains by providing built-in
lexicographic comparison for tuples and records, a declarative
`.sort(by: ...)` API with chaining, and guaranteed stable sort by
default.

## Implications for Orthon

- Tuples / structs as natural sort keys — lexicographic comparison is
  built into the type, no separate comparator needed.
- The standard library should provide declarative comparators.
- Sort order (ascending / descending) is a modifier, not a separate
  comparator.
- Sort stability is guaranteed by default.
- Related concepts: `important/SORTING.md`, `essential/DATA_MODEL.md`

## Open Questions

- Should reverse/descending be a function or a wrapper type?
- How to handle partial ordering (NaN, None)?

## Decision History

Initial hypothesis derived from `imperative-crutch-sorting.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `important/SORTING.md`
- [ ] `essential/DATA_MODEL.md`
- [ ] `what/GLOSSARY.md`
