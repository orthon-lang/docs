# Imperative Crutch: Collections and Loops

## Problem

Manual index-based loops, empty accumulator lists, explicit flags for
search — the code describes *how* to iterate instead of *what* to obtain.

## Examples

### 1. Indexed iteration

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `for i in range(len(lst)): print(i, lst[i])` | `for i, item in enumerate(lst): print(i, item)` |
| Java | `for (int i=0; i<list.size(); i++) { ... }` | Enhanced for, or `IntStream.range(0, list.size())` |

### 2. Filtering and transformation

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Empty list + loop + `append` + `if` | `[x*2 for x in data if x>0]` |
| Java | Loop with `add()` inside | `stream().filter(...).map(...).collect(...)` |

### 3. Finding first match

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Loop with `break`, manual flag | `next((x for x in data if cond(x)), None)` |
| Java | Loop + `if` + `break` + `found` check | `stream().filter(cond).findFirst().orElse(null)` |

## Implications for Orthon

- The language should provide composable high-level collection operations
  (map, filter, reduce, find, enumerate) as part of the standard library,
  rather than forcing handwritten loops.
- Indexed iteration is a special case, not the primary pattern.
- Transformation chains should be declarative and lazy where possible.
- The compiler / runtime should check types at compile time, not at runtime.

**See also:** concepts `GENERATORS.md`, `UNPACKING.md`, `SORTING.md`
