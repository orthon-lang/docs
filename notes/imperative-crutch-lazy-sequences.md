# Imperative Crutch: Lazy Sequences / Iterators

## Problem

Manually implementing an iterator requires writing a stateful class/object,
explicitly defining `__iter__` / `__next__` or implementing the `Iterator`
interface — too much boilerplate for a simple idea.

## Examples

### Lazy generation of an infinite sequence

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Class with `__iter__` and `__next__`, manual state management | `def gen(): i=0; while True: yield i; i+=1` |
| Java | Anonymous class implementing `Iterator` | `Stream.iterate(0, i -> i+1)` / `IntStream.iterate(0, i -> i+1)` |

## Implications for Orthon

- Generators (functions with `yield`) are a basic language element, not a
  library feature.
- Lazy sequences must be lazy by default; materialization must be explicit.
- Stream-like operations (map, filter, take, drop) operate on lazy sequences
  without allocating intermediate collections.
- Infinite sequences are a valid concept that the language should not
  artificially block.
- Generator composition — chaining without losing laziness.

**See also:** concepts `GENERATORS.md`, `EXECUTION_IMAGE.md`, `EXECUTION_PROGRAM.md`
