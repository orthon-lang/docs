# Imperative Crutch: Multi-Key Sorting

## Problem

Sorting by multiple fields requires manually writing comparators with
comparison chains — code becomes verbose and fragile.

## Examples

### Sorting by multiple fields

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `cmp_to_key`, complex conditional constructs | `sorted(list, key=lambda x: (x.a, x.b))` |
| Java | Anonymous `Comparator` with manual compare chain (nightmare pre-Java 8) | `list.sort(Comparator.comparing(Obj::getA).thenComparing(Obj::getB))` |

## Implications for Orthon

- Tuples / structs as natural sort keys — lexicographic comparison is
  built into the type, no separate comparator needed.
- The standard library should provide declarative comparators:
  `.sort(by(x => x.a).then(x => x.b))`.
- Sort order (ascending / descending) is a modifier, not a separate
  comparator.
- Sort stability is guaranteed by default (see concept `SORTING.md`).

**See also:** concept `SORTING.md`, `DATA_MODEL.md`
