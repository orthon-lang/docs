# Persistent Data Structures

> **⚠️ HYPOTHESIS — This document is a research hypothesis, not a design decision.**
> Orthon currently has mutable `List` (DYNAMIC_COLLECTIONS.md) and conditionally
> immutable `Tuple` (DATA_MODEL.md). Whether fully immutable/persistent collections
> are needed — and at what level — has not been researched. This document frames
> the question.
>
> **Last updated:** 2026-07-26

## Problem

Orthon's current collection model has two built-in ordered types:

- **`List[T]`** — mutable, growable, reference semantics (DYNAMIC_COLLECTIONS.md)
- **`Tuple`** — conditionally immutable (immutable if nested elements are immutable)

This covers the common cases but leaves a gap: **what if a programmer wants
guaranteed immutability of a collection as a whole — including its contents —
without relying on the discipline of not calling `.push()` or `.set()`?**

Scenarios where this matters:

1. **Hash keys / set membership** — Mutable collections cannot safely be used
   as keys in hash maps or members of sets, because mutation would invalidate
   the hash. Immutable collections can.
2. **Concurrent access** — An immutable collection can be shared across threads
   without locks, copying, or ownership tracking. The compiler can prove safety
   statically.
3. **Caching / memoisation** — Functions that take mutable collections cannot
   safely cache results by input identity. Immutable collections can be used
   as cache keys.
4. **Undo history / versioning** — Keeping snapshots of state over time is
   expensive with mutable collections (deep copy each snapshot). Persistent
   data structures share structure across versions, making snapshots cheap.
5. **API contracts** — A function that accepts a `List[T]` cannot statically
   guarantee it won't mutate the argument. An immutable collection type
   provides this guarantee at the type level.

The core question: **does Orthon need a dedicated immutable collection type,
or can the existing `Tuple` + copy-on-write + programmer discipline cover
all these cases?**

## Level

**Stdlib with core type system support** — or purely **stdlib**, depending on
how deep the guarantees need to be.

| Level | What it means | Trade-off |
|-------|---------------|-----------|
| **Core** — special syntax, built-in type | `ImmutableList[T]` as a first-class representation like `Tuple` | Heavier language surface; every new collection type requires language changes |
| **Stdlib with core hooks** — library type with compiler optimisations | `PersistentList[T]` defined in the standard library, but the compiler knows about it for optimisation (like Rust's `Vec` vs. slice types) | Balances expressiveness with language minimality |
| **Pure stdlib** — library type, no special compiler treatment | Any immutable collection is a user-defined type; no syntactic sugar or compiler optimisation | Smallest language core, but no optimisations (structural sharing unlikely to be efficient without compiler support) |

**Initial hypothesis: stdlib with core hooks.** The core defines an
`Immutable` trait/marker that library types implement. The compiler can use
this marker for optimisations (e.g., eliding copies, allowing hash-key usage).
The concrete collection types live in the standard library.

## Proposal (Hypothesis)

Add an `Immutable` marker trait (or equivalent) that a collection type can
implement. The compiler guarantees that values of types implementing
`Immutable` cannot be mutated through any operation. Standard library
provides persistent collection types built on this trait:

```orthon
// Marker trait — no methods, purely a guarantee
trait Immutable

// Persistent list — structural sharing on "modification"
type PersistentList[T] is Immutable

// "Modification" returns a new collection, shares structure with old
mut new_list = list.append(42)
// list is unchanged
// new_list shares most of its structure with list
```

Key properties:
- **Structural sharing** — "Modifying" a persistent collection creates a new
  collection that shares internal nodes with the original. Time: O(log n) or
  O(1) amortised. Space: O(1) additional per operation.
- **Value semantics** — A `PersistentList[T]` is compared structurally, not
  by identity. Two lists with the same elements are equal.
- **Thread-safe by construction** — Because no mutation is possible, sharing
  across threads is always safe. No locks, no ownership tracking.
- **Usable as hash keys** — Because the value cannot change, it can safely
  participate in hash maps, sets, and caches.
- **Interop with mutable `List`** — Conversion functions: `list.to_persistent()`
  and `persistent_list.to_mutable()`. Conversion to persistent may share
  structure with the original mutable list (if the mutable list is not
  referenced elsewhere, the persistent version can take ownership of its
  buffer).

### Relationship with existing types

| Type | Mutation | Copy semantics | Sharing across versions |
|------|----------|---------------|------------------------|
| `List[T]` | Mutable (`push`, `set`, etc.) | Reference (CoW on shared mutation) | No — mutation is in-place |
| `Tuple` | Conditionally immutable | Value (deep copy on assignment) | No — each tuple is independent |
| `PersistentList[T]` (proposed) | Immutable (returns new version) | Value (structural sharing) | Yes — versions share internal nodes |

### Comparison with COW (COPY_ON_WRITE.md)

Copy-on-write and persistent data structures are often confused. They solve
different problems:

| Aspect | Copy-on-Write (COW) | Persistent Data Structures |
|--------|---------------------|---------------------------|
| **When does sharing break?** | On mutation — the mutating reference gets its own copy | Never — structure is always shared; "modification" creates a new root |
| **Concurrent safety** | Needs synchronisation for shared references | Safe by construction — no mutation |
| **Versioning** | Only one "active" version (the mutating reference) | All versions are equally valid — full persistence |
| **Memory** | Only one copy of unmodified data | Internal nodes shared across all versions |
| **Typical use** | Lazy cloning in otherwise mutable systems | Purely functional data structures |

## Trade-offs

### In favour

- **Safety guarantee** — Immutability is enforced by the type system, not by convention
- **Concurrent safety** — No locks, no data races, no ownership complexity
- **Hash-key safe** — Can be used in maps, sets, caches without defensive copying
- **Cheap snapshots** — Versioning, undo, and time-travel debugging become practical
- **LLM generability** — Immutable data is easier for an LLM to reason about (no aliasing, no hidden mutation)
- **Alignment with trends** — Immutable-by-default is a clear modern trend (see trends summary)

### Against

- **Cognitive overhead** — Two collection families (mutable and persistent) instead of one. The programmer must choose.
- **Performance for common cases** — Persistent data structures have higher constant factors than mutable arrays. A simple loop appending to a persistent list is O(n log n) vs O(n) amortised for a mutable list.
- **Implementation complexity** — Efficient persistent data structures (HAMT, RRB-trees) are complex to implement and may need compiler support for good performance.
- **Memory overhead** — Internal node sharing adds per-element overhead compared to a flat array.
- **Not compatible with in-place algorithms** — Sorting, shuffling, and other mutating algorithms cannot work on persistent collections.

## Related Concepts

- **`MUTABILITY.md`** (essential) — Immutability-by-default principle. Persistent collections are a natural extension: if bindings are immutable by default, should collections also be?
- **`COPY_ON_WRITE.md`** (important) — Related but distinct mechanism (see comparison table above). COW is the current strategy for managing shared mutable access to `List[T]`.
- **`DYNAMIC_COLLECTIONS.md`** (deferrable) — Defines `List[T]` and `Array[T, N]`. A persistent collection would be a third ordered collection type.
- **`DATA_MODEL.md`** (essential) — Defines `Tuple` as a representation. A persistent list is structurally different from a tuple (variable-size, structural sharing).
- **`VALUE_SEMANTICS.md`** (essential) — Persistent collections have value semantics with structural sharing, which is a new point in the design space (not pure copy, not reference).
- **`EQUALITY.md`** (essential) — Structural equality for persistent collections should work the same as for other value types.
- **`FOUNDATIONAL_ABSTRACTIONS.md`** (essential) — How do persistent collections relate to Data, Data Modifiers, and Representations?

## Alternatives

| Alternative | Description | When to use |
|-------------|-------------|-------------|
| **No persistent collections** | Rely on `Tuple` for fixed-size immutable data and `List[T]` + CoW for dynamic data. Hash keys use `Tuple`. Concurrent safety uses `shared` + synchronisation. | Minimal language; most cases already covered |
| **Persistent only** | All collections are persistent by default. Mutable collections are an explicit opt-in (Clojure model). | Maximum safety; functional-first style |
| **Freeze operation** | A `list.freeze()` method returns an immutable view that cannot be mutated. The underlying list can still be mutated through other references. | Simple API; but frozen view can be invalidated by mutation through another reference — weak guarantee |
| **Copy-on-write everywhere** | All collections use COW, including after explicit `freeze()`. This is the current approach in COPY_ON_WRITE.md. | Balances mutability and sharing; but doesn't give hash-key safety or thread safety |

## Open Questions

1. **Need validation** — Do the listed scenarios (hash keys, concurrent access,
   caching, snapshots) actually arise in practice for Orthon's target audience?
   Or is `Tuple` + CoW sufficient?
2. **Performance baseline** — What are the acceptable performance trade-offs?
   Is O(log n) for append/update acceptable for v0.1?
3. **Compiler support** — How much compiler magic is needed? Can efficient
   structural sharing be achieved with just library code, or does the compiler
   need to know about the data structure internals?
4. **Trait marker vs. type family** — Should `Immutable` be a trait that any
   type can implement, or should persistent collections be a separate type
   family (like `PersistentList[T]`, `PersistentMap[K, V]`)?
5. **Interaction with Sequence** — The Data Model defines `Sequence` as a
   representation (lazy/eager variable-size ordered data). Is a persistent
   list a kind of `Sequence`, or a separate concept?
6. **Graduation path** — If persistent collections are added in v0.2 instead
   of v0.1, does the language need backward-compatible migration?
