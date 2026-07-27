# Sorting

## Issue (Why)

When sorting data multiple times by different keys, should the relative order of equal elements be preserved? Instability is invisible on first sorting but causes subtle correctness bugs on subsequent passes. The core problem: sorting is not a one-shot operation — it is often applied as a pipeline.

## Principles

1. **Stability by default** — The default sort is stable for all sorting operations on ordered collections.
2. **Opt-in instability** — An explicit `sort_unstable` variant exists for performance-critical use.
3. **Composability** — Multiple sort passes produce predictable results.
4. **Performance parity** — The stable sort algorithm must not be significantly slower than unstable for common cases.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Sort Algorithm Policy | Selects the default sorting algorithm |
| Stability Policy | Governs whether stability is guaranteed or implementation-defined |

## Model (What)

The language provides a default stable sort that guarantees preservation of relative order for equal elements. An explicit `unstable` variant is also available:

```orthon
# Stable by default
list.sort()
list.sort(key: .priority)

# Explicit unstable variant
list.sort_unstable()
list.sort_unstable(key: .priority)
```

The default algorithm is **Timsort** — a hybrid stable sorting algorithm O(n log n) worst-case, O(n) best-case on partially sorted data.

## Default Strategy

Timsort is the default. Stability is guaranteed. The programmer must explicitly opt into instability.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Pattern-defeating quicksort (pdqsort) | Unstable, fast. Suitable for primitive types. |
| Adaptive merge sort | Stable, O(n log n), memory overhead O(n). |
| Counting/Radix sort | Linear time for specific key domains. |

## Open Questions

1. Should the sort algorithm be a configurable policy or hardcoded to Timsort?
2. Should arrays of primitives get an automatic unstable sort for performance?
3. How to expose sort algorithm selection without API bloat?

## Decision History

- **EDR-066:** Sorting accepted as StdLib — sort algorithms are library implementations. The stability guarantee is a policy choice (default stable) but sorting as a concept does not require new language semantics.
- **Classification per D-03:** StdLib. Sort algorithms are implementations of the `sorted` method on collection types. Cross-reference Algorithm Policy (IMPLEMENTATION_POLICIES.md).

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
