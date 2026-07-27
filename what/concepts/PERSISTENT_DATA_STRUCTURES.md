# Persistent Data Structures

## Issue (Why)

Orthon's current collection model has mutable `List[T]` and conditionally immutable `Tuple`. This leaves a gap: what if a programmer wants guaranteed immutability of a collection as a whole — including its contents — without relying on discipline?

Scenarios where persistent (immutable, structural-sharing) collections matter:
1. **Hash keys / set membership** — Mutable collections cannot safely be used as hash map keys.
2. **Concurrent access** — An immutable collection can be shared across threads without locks.
3. **Caching / memoisation** — Functions can safely cache results by input identity.
4. **Undo history / versioning** — Persistent data structures share structure across versions, making snapshots cheap.
5. **API contracts** — An immutable collection type guarantees non-mutation at the type level.

## Principles

1. **Structural sharing** — "Modifying" a persistent collection creates a new collection that shares internal nodes with the original.
2. **Value semantics** — A persistent collection is compared structurally, not by identity.
3. **Thread-safe by construction** — No mutation means no data races.
4. **Usable as hash keys** — Because the value cannot change, it safely participates in hash maps and sets.
5. **Interop with mutable collections** — Conversion functions: `list.to_persistent()`, `persistent_list.to_mutable()`.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Collection Policy | Determines which collection families the StdLib provides |
| Immutability Policy | Governs the semantics of immutable vs mutable collections |

## Model (What)

The standard library provides persistent collection types built on structural sharing:

```orthon
# Marker trait — no methods, purely a guarantee
trait Immutable

# Persistent list — structural sharing on "modification"
type PersistentList[T] is Immutable

# "Modification" returns a new collection, shares structure with old
let new_list = list.append(42)
# list is unchanged
# new_list shares most of its structure with list
```

### Relationship with existing types

| Type | Mutation | Copy semantics | Sharing across versions |
|---|---|---|---|
| `List[T]` | Mutable | Reference (CoW on shared mutation) | No |
| `Tuple` | Conditionally immutable | Value (deep copy on assignment) | No |
| `PersistentList[T]` | Immutable (returns new version) | Value (structural sharing) | Yes |

## Default Strategy

Persistent Data Structures is a **StdLib** concept. The standard library provides persistent (structurally-sharing) collection types. The core defines an `Immutable` marker trait that library types implement. The compiler can use this marker for optimisations (e.g., eliding copies, allowing hash-key usage).

## Alternative Strategies

| Strategy | Description |
|---|---|
| No persistent collections | Rely on `Tuple` for fixed-size immutable data and `List[T]` + CoW for dynamic data. |
| Persistent only | All collections are persistent by default (Clojure model). |
| Freeze operation | `list.freeze()` returns an immutable view. Weaker guarantee (underlying list still mutable). |

## Open Questions

1. Do the listed scenarios (hash keys, concurrent access, caching, snapshots) actually arise in practice for Orthon's target audience?
2. Performance trade-offs — persistent collections have higher constant factors than mutable arrays.
3. Should persistent collections use HAMT or RRB-tree implementations?

## Decision History

- **EDR-069:** Persistent Data Structures accepted as StdLib — structural sharing collections. The `Immutable` marker trait is a type-level guarantee, but the collection implementations are standard library types. No new language semantics.
- **Classification per D-03:** StdLib. Persistent collections are library implementations that use the type system's trait mechanism. The `Immutable` marker trait requires no new compiler semantics beyond the existing trait system (EDR-019).

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
