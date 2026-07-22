# Dynamic Collections

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does the language represent a collection of multiple values — ordered, unordered, indexed, growable, or fixed?

Two contrasting design lineages exist:

1. **Fixed-size homogeneous arrays** (Java, C, Go) — an array has a fixed length determined at creation. All elements share the same type. To grow, the programmer must allocate a new array and copy elements. Predictable memory layout, fast indexing, type-safe. But rigid: every collection size must be known in advance or managed manually.

2. **Dynamic heterogeneous collections** (Python `list`, JavaScript `Array`, Ruby `Array`) — a list can grow and shrink dynamically. Elements can be of different types. Flexible and convenient, but sacrifices memory predictability, type safety, and sometimes performance. The same syntax (`list.append(x)`) works for any type.

The core problem: **should the language offer both fixed and dynamic collections as first-class concepts, or unify them under a single abstraction?**

A related problem: **should homogeneity be enforced at compile time (generic collections) or can a collection hold values of any type (heterogeneous)?** The answer affects the type system, the standard library, and the language's ergonomics for quick scripting versus large-scale programs.

## Principles

1. **Dynamic by default for general-purpose use** — The primary ordered collection (`List[T]`) is dynamically growable. Fixed-size collections (`Array[T, N]`) are available for performance-sensitive or ABI-stable contexts.
2. **Type-safe by default** — Collections are homogeneous by default (`List[Int]`, `Array[String, 5]`). Heterogeneous collections are available through explicit union types (`List[Int | String]`) or the `Any` type.
3. **Value semantics for fixed collections** — `Array[T, N]` has value semantics: copying produces an independent array. `List[T]` has reference semantics: copying shares the underlying data (use `.copy()` for an independent clone).
4. **Seamless interop** — `List[T]` and `Array[T, N]` share the same iteration and indexing protocols. Code that works on one works on the other (structural typing).
5. **No primitive array syntax** — Arrays are not a special syntactic form. They are library types with compiler support for literals.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Collection Policy | Defines which collection types are built-in vs. library-defined |
| Grow Policy | Determines whether collections are fixed-size, growable, or both |
| Element Type Policy | Governs whether collections enforce homogeneous typing at compile time |
| Memory Policy | Controls whether collections use value or reference semantics |
| Literal Policy | Defines literal syntax for collection construction |

## Model (What)

Two primary ordered collection types: `List[T]` (dynamic, growable) and `Array[T, N]` (fixed-size, value semantics). Both share the same iterator and index protocols. Homogeneous by default; heterogeneous via union types.

```
# List — dynamic, growable, reference semantics
fruits: List[String] = ["apple", "banana", "cherry"]
fruits.push("date")                    # grows dynamically
first = fruits[0]                      # indexed access

# Array — fixed-size, value semantics
coordinates: Array[Int, 3] = [10, 20, 30]
# coordinates.push(40)                 ← ERROR: Array has fixed size
cloned = coordinates                   # value copy — independent

# Homogeneous by default — type-safe
ints: List[Int] = [1, 2, 3]
# ints.push("hello")                   ← ERROR: expected Int

# Heterogeneous via union types
mixed: List[Int | String] = [1, "hello", 42]

# Common protocol — both support iteration and indexing
fun sum(values: Iterable[Int]) -> Int:
    total = 0
    for v in values:
        total += v
    return total

print(sum([1, 2, 3]))                  # List — OK
print(sum(Array[Int, 3]([4, 5, 6])))  # Array — OK (shared Iterable protocol)
```

Key features:
- **`List[T]`** — growable, reference semantics. Backed by a dynamic array (amortized O(1) append).
- **`Array[T, N]`** — fixed-size, value semantics. Stack-allocated for small N (configurable threshold), heap-allocated otherwise.
- **`Iterable[T]`** — shared trait for iteration. Both `List[T]` and `Array[T, N]` implement it.
- **Literal syntax** — `[value, ...]` produces a `List` by default. Array literals use `Array[T, N]([value, ...])`.
- **Homogeneous by default** — the type checker enforces that all literal elements have the same type (or a common supertype).

### Comparison with Java and Python

| Aspect | Java | Python | Orthon (proposed) |
|---|---|---|---|
| Primary collection | `ArrayList<T>` | `list` | `List[T]` |
| Fixed array | `T[]` (special syntax) | `array.array` | `Array[T, N]` (library type) |
| Heterogeneous | Not allowed (generic) | Always allowed | Via union types |
| Growable | `ArrayList` only | All lists | `List[T]` |
| Value semantics | No | No (reference) | `Array[T, N]` only |
| Literal syntax | `{1, 2, 3}` (not for arrays) | `[1, 2, 3]` | `[1, 2, 3]` for List |

## Default Strategy

`List[T]` is the default collection type. Array literals produce a `List`. Fixed-size `Array[T, N]` is available explicitly. Type inference infers the element type from literal elements; if elements have different types, the inferred type is a union.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Arrays-only | No dynamic collections in the language. All collections are fixed-size; dynamic resizing is a library concern (Rust's `Vec` is a library, not a language type). |
| Heterogeneous-everywhere | No type enforcement on collection elements. Any collection can hold any type (Python `list`, JavaScript `Array`). Flexible but sacrifices type safety. |
| Immutable collections only | All collections are immutable by default. Mutating operations return a new collection (Clojure, functional languages). Eliminates reference-sharing bugs. |
| No built-in collections | No collection types in the language. All collections are library-defined (minimal language core). |

## Open Questions

1. Should `List[T]` have a syntax shorthand for fixed-capacity pre-allocation (e.g., `List[Int](capacity: 100)`)?
2. Should slices / views (non-owning references into a collection) be a separate type or part of `List` / `Array`?
3. How does `Array[T, N]` interact with the Execution Program model — can N be a runtime value or must it be compile-time constant?
4. Should there be dedicated syntax for tuple-like fixed-size heterogeneous collections (e.g., `(Int, String, Bool)`)?
5. How do dynamic collections interact with the ownership model — does `List[T]` own its elements or borrow them?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/concepts/research/DATA_MODEL.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/LIBRARY_BOUNDARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/concepts/research/ALLOCATION.md`
- [ ] `how/concepts/research/OWNERSHIP.md`
- [ ] Other: `how/strategies/DEFAULT_STRATEGY.md`
