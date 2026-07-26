# Software Transactional Memory

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language coordinate concurrent access to shared mutable state without the fragility of locks?

Three broad concurrency models exist for shared state:

1. **Locks and mutexes (Java, C++, C#)** — explicit `synchronized`, `Lock`, `Mutex`. Programmer acquires and releases locks to ensure mutual exclusion. Simple for single-resource access, but fragile: deadlocks (circular wait), livelocks, priority inversion, and composability failure (locking on resource A then B requires knowing all other callers' locking order). Locks do not compose — combining two lock-based operations is not itself safe.

2. **Transactional memory** — inspired by database transactions, applied to in-memory state. A transaction reads and writes shared memory optimistically. If two transactions conflict (write-write or read-write), the runtime rolls one back and retries. The programmer declares *what* should be atomic; the runtime manages *how*. Transactions compose — combining two transactional operations produces a larger transaction.

3. **Actors and channels** — share-nothing message passing (Erlang, Go). State is isolated per actor; communication is via messages. No shared memory, thus no locks. But some algorithms are naturally shared-memory (caches, shared counters, complex state graphs) and become awkward when forced into message-passing.

The core problem: **locks are the lowest-level and most error-prone model**, yet they remain the default in most mainstream languages. STM offers composable atomicity without the programmer manually managing lock order, but at a runtime cost (transaction bookkeeping, rollback, retry). The question for Orthon is whether STM deserves a place alongside (or instead of) actors as a first-class concurrency model.

## Principles

1. **Automatic conflict detection** — The runtime detects conflicting concurrent accesses and retries failed transactions automatically. The programmer never manages locks.

2. **Composability** — Two transactional operations compose into a larger atomic transaction without additional coordination. `do_atomic { a.transfer(b, 100); b.log_transfer(100) }` is atomic even though two separate refs are involved.

3. **Opt-in, not default** — STM is available as an explicit concurrency primitive, not the default execution model. Actors/channels are the default; STM is for cases where shared mutable state is the natural solution.

4. **No lock hierarchy** — The programmer does not specify lock order. The runtime resolves conflicts through optimistic retry, eliminating deadlock as a class of error.

5. **Explicit transaction boundaries** — Transactional blocks are syntactically visible. There is no implicit transactional behaviour — the programmer marks where atomicity is needed.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Concurrency Model Policy | Determines whether STM is available alongside actors (actor-default, STM-opt-in) or excluded entirely |
| STM Conflict Resolution Policy | Governs conflict detection strategy (optimistic vs pessimistic, read-write vs write-write) |
| STM Retry Policy | Controls retry limits, backoff strategy, and side-effect handling within transactions |
| STM Primitive Policy | Defines which transactional primitives exist (Ref/coordinated, Atom/uncoordinated, Agent/async) |

## Model (What)

STM provides **transactional references** — memory cells whose reads and writes become visible atomically when the enclosing transaction commits.

### Primitives

Inspired by Clojure's model, three STM primitives exist, each with a distinct coordination scope:

**Ref** — A coordinated transactional reference. Reads and writes participate in the current transaction. Multiple Refs can be updated atomically within a single transaction. Transactions are optimistic: the runtime detects conflicts (two transactions writing to the same Ref) and retries the loser automatically.

```orthon
# Ref — coordinated, synchronous
counter = Ref(0)

transaction:
    old = counter.read()
    counter.write(old + 1)
# On commit, both counter and other Refs in this transaction become visible atomically
```

**Atom** — An uncoordinated atomic reference. Changes are applied atomically but independently. An Atom is not part of a transaction — each swap is its own atomic operation. Use when you need atomicity for a single value without coordinating with other Refs.

```orthon
# Atom — uncoordinated, synchronous
counter = Atom(0)
counter.swap(fn (old) -> old + 1)  # atomic compare-and-swap
```

**Agent** — An uncoordinated asynchronous reference. Changes are dispatched and applied on a separate thread. The caller does not wait for the result. Use for background updates where consistency with other state is not required.

```orthon
# Agent — uncoordinated, asynchronous
logger = Agent([])
logger.send(fn (log) -> log + [entry])  # dispatched, returns immediately
```

### Transaction Semantics

- **Optimistic concurrency**: Transactions read Refs optimistically and validate on commit. If a Ref was modified by another transaction since the read, the transaction is retried.
- **Automatic retry**: The runtime retries failed transactions. Side effects within a transaction must be idempotent or handled via `ensure` (run-on-commit) callbacks.
- **No nested transactions**: A transaction is flat — starting a transaction within a transaction is a no-op (the inner block joins the outer transaction).
- **No I/O in transactions**: Side effects (file I/O, network calls) are prohibited within transactions because they cannot be rolled back. `ensure` callbacks can schedule side effects on commit.

## Default Strategy

STM is available as an opt-in concurrency model. The default strategy uses actors and channels for inter-actor communication. STM via Refs is available for coordinating shared state within an actor or across a small group of cooperating actors. Atoms and Agents are available as lighter-weight primitives.

```
actor CounterGroup:
    mut refs: Map<String, Ref<Int>> = {}

    fun increment(name: String)
        transaction:
            current = refs[name].read()
            refs[name].write(current + 1)

    fun get(name: String) -> Int
        refs[name].read()   # implicit transaction for single read
```

## Alternative Strategies

| Strategy | Description |
|---|---|
| Full STM default | All mutable state is transactional by default. Simpler mental model but higher runtime overhead. |
| No STM (actors only) | No transactional memory. All state is actor-isolated. Simplifies the runtime but forces awkward patterns for shared-state algorithms. |
| Hardware TM | Uses CPU transactional memory extensions (Intel TSX, IBM z) for hardware-accelerated transactions. Not portable. |
| Persistent data structures + CAS | Immutable data with atomic reference swaps (Clojure's `alter` on Refs, `swap!` on Atoms). Avoids rollback but requires structural sharing. |

## Open Questions

1. Should STM be part of the language (built-in syntax) or a library (functions on Refs/Atoms/Agents)?
2. How does STM interact with the actor model — can actors use Refs internally?
3. Should transactions be explicit blocks (`transaction:`) or implicit (function-level annotation)?
4. What happens when a transaction retries — how are side-effectful operations (logging, metrics) handled?
5. Is `ensure` (run-on-commit callback) sufficient for scheduling side effects, or does the language need a stronger mechanism?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

**See also:**
- [`CONCURRENCY.md`](./CONCURRENCY.md) — actor model and channel-based concurrency
- [`MUTABILITY.md`](./MUTABILITY.md) — immutability-by-default and explicit mutation
- [`COMPILE_TIME_CONCURRENCY_SAFETY.md`](./COMPILE_TIME_CONCURRENCY_SAFETY.md) — static concurrency correctness
