# Contract `old` — State at Entry (Deferred Hypothesis)

> **⚠️ HYPOTHESIS — deferred to v0.2 or later, not an accepted concept.**
> Decision of the author, 2026-10-19: `old` is **not** part of v0.1 contracts.
> It adds complexity now and gives little in return. This document keeps the
> analysis so the work is not lost and can be reopened when a trigger (below)
> fires.
> Home: `how/concepts/research/deferrable/` — deferred to v0.2/v0.3.
>
> **Last updated:** 2026-10-19
>
> **Related:** [`../essential/TRAIT_CONTRACT_UNIFICATION.md`](../essential/TRAIT_CONTRACT_UNIFICATION.md)
> (the contract hypothesis this was split from),
> [`../../../../what/concepts/CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) (EDR-056, defines `old`),
> [`../important/CONTRACTS.md`](../important/CONTRACTS.md) (research draft),
> [`../essential/EFFECT_FOOTPRINT.md`](../essential/EFFECT_FOOTPRINT.md),
> [`FRAME_CONDITIONS.md`](FRAME_CONDITIONS.md),
> [`../../../../what/SEMANTIC_MODEL.md`](../../../../what/SEMANTIC_MODEL.md) § Mutation,
> [`../../../../what/concepts/COPY_ON_WRITE.md`](../../../../what/concepts/COPY_ON_WRITE.md),
> [`../../../DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md)

---

## Decision (deferral)

`old` — the way a postcondition refers to state at function entry — is **deferred**. Contracts in v0.1 are written without it. A postcondition speaks about the state after the call and about the result; it does not compare with the state before.

Reason: a language-level construct that needs rules for what is snapshotted, which state may be named, how cost is bounded, and how it interacts with mutation kinds, copy-on-write, and the effect footprint buys relational statements about **mutating** methods only. That is a small part of the contract use cases (see "What v0.1 contracts can and cannot say").

## Problem

[`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) defines `old` as an implicit variable that captures a parameter's value at function entry, "useful in `ensures` for mutable data", and uses `old.size` in its `Queue` example. It leaves open (Open Question 2) how `old` works with mutable reference parameters. Three things are unclear:

1. Where `old` is meaningful at all, given that Orthon is immutable by default.
2. What exactly is snapshotted, and what that costs under copy-on-write.
3. Whether `old` is a name (like `result`) or an operator.

## Findings

### `old` matters only for state the callable mutates

[`SEMANTIC_MODEL.md`](../../../../what/SEMANTIC_MODEL.md) § Mutation fixes three exclusive declaration kinds:

| Kind | Effect on `self` | Needs `old`? |
|---|---|---|
| `fun` | read-only | no: `self` at exit equals `self` at entry |
| `new` | never mutates `self`, returns a new value | no |
| `proc` | mutates `self`, identity preserved | yes |

Captured state (`using`) and global state (`@modifies`) can also be mutated; both are part of the effect footprint in [`EFFECT_FOOTPRINT.md`](../essential/EFFECT_FOOTPRINT.md). The rule that falls out: **`old(e)` is meaningful only for state in the callable's mutate-footprint.** For everything else `old(e)` equals `e`, and a compiler would warn about a redundant `old`.

### Snapshot cost under copy-on-write

A snapshot of the whole receiver (the `old.size` form of EDR-056) keeps a reference to shared data. With copy-on-write ([`COPY_ON_WRITE.md`](../../../../what/concepts/COPY_ON_WRITE.md)) the first mutation inside a `proc` then clones the whole collection, turning an O(1) `append` into O(n) in builds that check contracts. Only the expression named inside `old(...)` should be snapshotted (`old(self.items.len())` copies one number).

### A name from nowhere versus a time-shifting operator

`result` introduced a **name** that appeared without a declaration; naming it in the signature fixed that. `old(expr)` introduces no name: the expression is written explicitly, and the only implicit thing is **when** it is evaluated (at entry). The technique used for `result` therefore does not transfer; the open question is whether to keep an operator form or make the timing explicit through entry bindings.

## Options

| # | Form | Example | Magic | Cost | Prior art |
|---|---|---|---|---|---|
| 1 | Operator `old(expr)` | `ensures self.size == old(self.size) + 1` | none; only the moment | only the expression is snapshotted | common idiom: SPARK `X'Old` and OCL `@pre` (verified by search); Eiffel, JML, Dafny (general knowledge) |
| 2 | Named entry binding in the contract block | `val before = self.size` evaluated at entry | none | same | new form; needs a "evaluated at entry" rule |
| 3 | Primed names | `self.size' == self.size + 1` | the prime means "after" | same | classical notation; possible clash with quote characters |
| 4 | Whole-receiver `old.size` (EDR-056 today) | `ensures size == old.size + 1` | `old` appears from nowhere | snapshot of the whole `self`, copy-on-write clone | EDR-056 |

## Recommended shape if reintroduced

Option 1 with these restrictions:

1. `old(expr)` is allowed only in `ensures`.
2. The argument is mandatory: bare `old` and `old.field` are errors.
3. The argument is pure; only that expression is snapshotted.
4. `old` of state outside the callable's mutate-footprint is a warning (redundant).
5. Snapshotting a whole collection is a warning (copy-on-write cost).
6. Snapshots are evaluated after `requires` succeeds, so the expressions may rely on the precondition.

Option 2 is clearer about timing but adds a second kind of binding to the contract block; it can be added later if one snapshot is needed by several conditions.

## Examples by usage context (illustrative, if `old` returns)

```orthon
// proc: the main case
proc append(item: T)
    ensures self.items.len() == old(self.items.len()) + 1

proc pop() -> item: T
    requires self.len() > 0
    ensures self.len() == old(self.len()) - 1
    ensures item == old(self.last())

// fun and new: old is not needed
new sorted() -> result: List
    ensures result.len() == self.len()           // self is unchanged: plain self

fun len() -> n: Int
    ensures n == old(self.items.len())           // WARNING: fun does not mutate self, old(e) equals e

// Captured and global state: only what is declared mutable
fn(x) -> total += x using total                  // footprint: total is mutated
    ensures total == old(total) + x

// Contract object
contract Stack<T>
    proc push(item: T)
        ensures self.size == old(self.size) + 1
```

Errors and edge cases:

```orthon
requires old(self.size) > 0                      // ERROR: old only in ensures
ensures item == old(item)                        // ERROR: result does not exist at entry
ensures old(old(self.size)) > 0                  // ERROR: nested old
ensures self.size > old(self.next_id())          // ERROR: the expression must be pure
ensures self.items == old(self.items)            // WARNING: collection snapshot, copy-on-write cost
```

## What v0.1 contracts can and cannot say without `old`

| Kind of method | Relation to the state before the call | In v0.1 |
|---|---|---|
| `fun`, `new` | `self` is unchanged, so `self` itself is the state before | expressible: `ensures result.len() == self.len() + 1` on a `new` method |
| `proc` | needs the state before the mutation | **not expressible**: "size grew by exactly one" |

What a `proc` can still say is absolute conditions on the state after the call:

```orthon
proc push(item: T)
    requires self.size < self.capacity
    ensures self.top() == item                   // the pushed item is on top
    ensures self.size > 0
```

The workaround for a relational statement about a mutating operation is a `new` method that returns the changed value (`new pushed(item: T) -> result: Stack`), where `self` and `result` can be compared directly.

## Evidence

- SPARK: postconditions use `X'Old` for values at entry ([user guide](https://docs.adacore.com/spark2014-docs/html/ug/en/source/subprogram_contracts.html)).
- OCL: the `@pre` suffix refers to a value before the call ([lecture notes](https://st.inf.tu-dresden.de/files/teaching/03-swe-ocl.pdf)).
- Not verified here: the exact forms in Eiffel (`old`), JML (`\old`), and Dafny (`old(...)`); to be checked against primary sources before any EDR.

## Conflicts with accepted text (to amend when `old` is applied or formally removed)

These places still describe `old` or use a mutating `fun` with `old.size`; none was changed by this deferral:

- [`what/concepts/CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md): the `old` bullet in the Model section, the desugaring with `old = snapshot(balance)`, the `Queue` example (`fun enqueue(self, item: T)` with `old.size`), and Open Question 2.
- [`../important/CONTRACTS.md`](../important/CONTRACTS.md): the same bullet and `Queue` example.
- The `Queue` example uses `fun` for a mutating method; under SEMANTIC_MODEL a mutating method is `proc`.

An amendment to the accepted CONTRACTS concept (or an EDR note that `old` is deferred) is a separate step requiring the author's decision.

## Reintroduction triggers

Revisit when any of these holds:

1. Mutable (`proc`-heavy) types become a main use case and users need "changed by exactly this much" statements.
2. [`EFFECT_FOOTPRINT.md`](../essential/EFFECT_FOOTPRINT.md) or [`FRAME_CONDITIONS.md`](FRAME_CONDITIONS.md) is accepted, so the set of mutable state is defined by the language.
3. Test synthesis from contracts needs before/after pairs.

## Open Questions

1. Operator `old(expr)` (option 1) or entry bindings (option 2)?
2. Is `old` ever needed outside `proc` once the effect footprint is accepted (captures, globals, `shared` values)?
3. How are snapshots of values with identity (rather than value semantics) taken?
4. Interaction with an `invariant` of a mutable type: are transition conditions (before/after pairs) a separate kind of contract?

## See also

- [`../essential/TRAIT_CONTRACT_UNIFICATION.md`](../essential/TRAIT_CONTRACT_UNIFICATION.md)
- [`../../../../what/concepts/CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md)
- [`../README.md`](../README.md) (research inbox)
