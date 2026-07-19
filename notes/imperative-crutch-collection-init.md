# Imperative Crutch: Collection Initialization

## Problem

Creating a collection and then manually adding each element is verbose,
hard to read, and encourages mutation where a literal would suffice.

## Examples

### 1. List with elements

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `lst = []; lst.append(1); lst.append(2)` | `lst = [1, 2]` |
| Java | `list = new ArrayList<>(); list.add(1); list.add(2);` | `List.of(1, 2)` / `Arrays.asList(1, 2)` |

### 2. Map with key-value pairs

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `d = {}; d['a'] = 1; d['b'] = 2` | `d = {'a': 1, 'b': 2}` |
| Java | `map = new HashMap<>(); map.put("a", 1); map.put("b", 2);` | `Map.of("a", 1, "b", 2)` |

## Implications for Orthon

- Collection literals (list, map, set) are a basic syntax element, not a
  library feature.
- Compact syntax for immutable collections by default.
- If a mutable collection is needed, it requires explicit notation (e.g.,
  `MutableList` or a `mut` qualifier).
- Avoid Java's situation where `Map.of()` is limited to 10 pairs — built-in
  support for arbitrary size.

**See also:** concept `DATA_MODEL.md`
