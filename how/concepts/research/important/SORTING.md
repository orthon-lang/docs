# Sorting Stability

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).>
> **Last updated:** 2026-07-20
## Issue (Why)

When sorting data multiple times by different keys, should the relative order of equal elements be preserved?

Instability is invisible on first sorting but causes subtle correctness bugs on subsequent passes. Sorting a table by `date` then by `priority` should keep records ordered by date within the same priority level. An unstable sort cannot guarantee this.

The core problem: **sorting is not a one-shot operation** — it is often applied as a pipeline (key by key). Stability guarantees compose; instability forces the programmer to sort by all keys simultaneously.

## Principles

1. **Stability by default** — The default sort is stable. Stability is guaranteed for all sorting operations on ordered collections.
2. **Opt-in instability** — An explicit `unstable_sort` or `sort_unstable` variant exists for performance-critical use.
3. **Composability** — Multiple sort passes produce predictable results.
4. **Performance parity** — The stable sort algorithm must not be significantly slower than unstable for common case data (Timsort property).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Sort Algorithm Policy | Selects the default sorting algorithm (Timsort, merge sort, pattern-defeating quicksort) |
| Stability Policy | Governs whether stability is guaranteed or implementation-defined |

## Model (What)

The language provides a default stable sort that guarantees preservation of relative order for equal elements. An explicit `unstable` variant is also available for performance-conscious code.

```
// Stable by default
list.sort()
list.sort(key: .priority)

// Explicit unstable variant
list.sortUnstable()
list.sortUnstable(key: .priority)
```

The default algorithm is **Timsort** — a hybrid stable sorting algorithm derived from merge sort and insertion sort, designed to perform well on many kinds of real-world data (partially sorted arrays, small runs). Timsort is O(n log n) worst-case, O(n) best-case on already-sorted data.

## Default Strategy

Timsort is the default. Stability is guaranteed. The programmer must explicitly opt into instability.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Pattern-defeating quicksort (pdqsort) | Unstable, fast, branchless — suitable for primitive types. |
| Adaptive merge sort | Stable, O(n log n), memory overhead O(n). |
| Counting/Radix sort | Linear time for specific key domains. |
| Comparator-based sort | User provides comparison function; stability language-defined. |

## Open Questions

1. Should the sort algorithm be a configurable policy or hardcoded to Timsort?
2. Should arrays of primitives (integers, floats) get an automatic unstable sort for performance?
3. How to expose sort algorithm selection to the user without API bloat?
4. Should `sort_by(key)` be stable if `sort()` is stable?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
