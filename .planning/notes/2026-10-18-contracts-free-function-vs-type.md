---
date: "2026-10-18 17:07"
promoted: false
---

# Contracts: Free Function vs Type

> **Exploratory note.** Captured during the 2026-10-18 review of the CONTRACTS concept (`what/concepts/CONTRACTS.md`, EDR-056). Not a decision record: it explains why contract forms differ between functions and types, and feeds the open question "what kinds of contracts exist".

## Premise

A function describes an **action** with an entry and an exit. A type describes a **set of values** that persist and are observed many times. So a function contract speaks about a *call* (a relation between two moments: entry and exit), and a type contract speaks about *values* (a predicate that must hold wherever the value is observable).

## Comparison

| | Free function | Type (and its methods) |
|---|---|---|
| Subject of the contract | a call | a value or state |
| Form | `requires` / `ensures` | `invariant` |
| What is asserted | relation input → output | property of the state |
| When checked | on entry and exit of the call | at creation and at the boundary of public operations |
| Names in scope | parameters, the result name, snapshot of parameters at entry | `self`, fields, plus the method's parameters |
| Who is responsible | `requires` = the caller, `ensures` = the function | `invariant` = the implementation of the type (constructor and all public methods) |
| `self` | none (free functions have no `self`) | present |

## Orthon-specific refinement

Types are immutable by default (`what/concepts/CONSTRAINED_TYPES.md`, principle 3). For an immutable value an invariant is checked once, at construction. That is exactly a constrained type:

```orthon
type Age = Int requires v >= 0 && v <= 150
```

The "check at every public boundary" rule matters only for mutable types. This gives three kinds of conditions:

1. **Value constraint** — one value, checked at construction (constrained type).
2. **Invariant of mutable state** — type or module, checked at the boundary of public operations.
3. **Call conditions** — `requires` / `ensures` on a function or method.

## Examples by context

```orthon
// Free function: call conditions only
fun withdraw(balance: Int, amount: Int) -> result: Int
    requires amount > 0
    requires amount <= balance
    ensures result == balance - amount

// Mutable type: invariants + method conditions
class Queue<T>
    invariant size >= 0
    invariant size <= capacity

    // Construction (Queue(capacity)) establishes the invariant; requires covers parameters only
    fun call(capacity: Int) -> Queue<T>
        requires capacity > 0

    // Mutating methods are `proc`; `old` is deferred (see CONTRACT_OLD_STATE.md)
    proc enqueue(item: T)
        requires size < capacity
        ensures self.back() == item        // invariants are not restated

// Private helper: the invariant may be temporarily broken, not checked
proc reindex()
    ...
```

Errors:

```orthon
fun f(x: Int) -> Int
    invariant x > 0              // ERROR: an invariant belongs to a type or module
fun g(x: Int)
    ensures self.size > 0        // ERROR: a free function has no self
```

## Open question: re-entrancy

A public method in the middle of an update calls foreign code (a callback) that re-enters another public method of the same object. The invariant is temporarily broken at that point, so "check on entry of every public method" reports a false violation. Either forbid such re-entry or exempt it from the check.

## Related

- Feeds the discussion question "what kinds of contracts exist" (trait vs contract vs `obeys`).
- Decision 3 of the review: `self` exists only for types; free functions have no `self`.
