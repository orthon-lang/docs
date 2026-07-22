# Iteration Loop

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language express repeated execution over a sequence of values?

Two fundamentally different loop models exist:

1. **C-style for loop** (`for (init; condition; increment)`) — a three-clause construct combining initialization, a boolean guard, and a per-iteration step. General-purpose: can iterate over arrays, count up, count down, walk pointers, or encode arbitrary state machines. But this generality is also its weakness: the three clauses are independent and can interact in unexpected ways. The compiler cannot verify correctness (e.g., off-by-one, infinite loops, missed edge cases).

2. **For-each iteration** (`for item in sequence`) — a single-purpose construct: consume values from an iterable source. The programmer describes *what* to iterate over; the language handles *how*. Simpler, safer, and more declarative, but cannot express all loop patterns (skip by two, reverse iteration, index-based access) without library support.

The core problem: **should the language include a general-purpose loop (C-style `for`) alongside the safer for-each, or should it commit to for-each-only and provide library abstractions for non-standard iteration patterns?**

A related question: **what counts as "iterable"?** Arrays, ranges, sequences, generators, async streams — each is a different kind of iteration with different semantics (eager vs lazy, synchronous vs asynchronous, finite vs infinite).

## Principles

1. **One loop construct** — There should be exactly one loop construct for consuming values from a sequence. No separate syntax for index-based vs. element-based iteration.
2. **Sequence-based** — The loop construct operates on sequences (values produced over time), not on indices. The programmer says *what* to iterate over, not *how* to produce each index.
3. **Exhaustive by default** — The loop processes every value in the sequence. Skipping, filtering, or early termination are explicit operations on the sequence, not implicit in the loop syntax.
4. **Composable with sequence operations** — Common iteration patterns (map, filter, take, skip, zip) should be expressible as sequence transformations before the loop, not as loop-internal control flow.
5. **No hidden allocation** — The loop should not force eager collection of lazy sequences. Iteration over a generator or range should stream values without allocating an intermediate buffer.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Iteration Policy | Defines which types are iterable and how iteration is resolved (protocol vs. trait) |
| Loop Model Policy | Determines whether the language includes a C-style for loop, for-each only, or both |
| Sequence Policy | Governs whether sequences are lazy or eager by default, and whether index-based access is available on sequences |
| Early Termination Policy | Controls how `break` and `continue` interact with structured iteration |

## Model (What)

The language has exactly one loop construct: `for item in sequence`. There is no C-style `for (;;)`. All iteration patterns are expressed through sequence transformations.

```
# Basic iteration — consumes every element
for item in items:
    process(item)

# Index-based iteration — express via index range, not loop syntax
for i in 0..len(array):
    process(array[i])

# Filter-map before the loop
for valid in items.filter(is_valid).map(transform):
    use(valid)

# Early termination
for item in items:
    if is_target(item):
        handle(item)
        break     # exits the loop

# Skipping elements — express via sequence operations
for item in items.skip(2).take(5):
    process(item)

# Enumerate — pair each element with its index
for (index, value) in items.enumerate():
    process(index, value)
```

Key features:
- **`for item in sequence`** — the only loop form. `sequence` is any iterable type (Sequence, Range, Generator, etc.).
- **No `for (;;)`** — index-based loops are expressed as `for i in 0..n:` (range literal) or `for i in array.indices:`.
- **No `while` with manual increment** — `while` exists as a separate construct for condition-based looping (not iteration), but the typical "while i < n: i += 1" pattern is replaced by `for i in 0..n:`.
- **`break` and `continue`** — available for early termination and skip-to-next. These are the only control-flow escapes inside `for`.
- **Destructuring in loop variable** — `for (a, b) in pairs:` decomposes each element automatically.

### Comparison with Java and Python

| Aspect | Java | Python | Orthon (proposed) |
|---|---|---|---|
| C-style for | `for (i=0; i<n; i++)` | Not available | Not available |
| For-each | `for (T item : items)` | `for item in items:` | `for item in sequence` |
| Index iteration | `for (i=0; i<n; i++) arr[i]` | `for i in range(n): arr[i]` | `for i in 0..n: array[i]` |
| While loop | `while (cond)` | `while cond:` | `while condition` |
| Skip/filter/take | Manual inside loop | `itertools.islice` | Sequence operations |

## Default Strategy

For-each is the only loop construct. Sequences support a rich set of lazy transformation operations (`.filter()`, `.map()`, `.take()`, `.skip()`, `.enumerate()`, `.zip()`, etc.) that compose before the loop. The compiler optimizes chain operations into fused loops where possible.

## Alternative Strategies

| Strategy | Description |
|---|---|
| C-style for included | Both `for (;;)` and `for-each` are available. C-style for handles complex iteration patterns directly, but creates two ways to iterate. |
| Index-based for-each | For-each loop includes an optional index variable: `for (i, item) in items`. Python's `enumerate` as syntax. |
| While as general iteration | No `for` construct; all iteration uses `while` with explicit iterator management. Rare and verbose. |
| Iterator protocol only | No built-in `for` loop; iteration is always explicit via `.next()` calls on iterators. Maximum orthogonality but ergonomic loss. |

## Open Questions

1. Should `0..n` (range) be a literal syntax or a standard library type?
2. How does `for` interact with async sequences — `for await item in async_stream:` or a separate construct?
3. Should `break` support a value (like Rust's `break value` for `loop`)?
4. How does iteration interact with ownership — does `for item in collection` consume or borrow the collection?
5. Should there be an `else` clause on `for` (Python-style, executed only if the loop completed without `break`)?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/concepts/research/GENERATORS.md`
- [ ] Other: `what/LIBRARY_BOUNDARY.md`
