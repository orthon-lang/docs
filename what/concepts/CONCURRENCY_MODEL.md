# Concurrency Model

> **⚠️ SUPERSEDED — [EDR-085](../how/decision_records/architecture/EDR-085-execution-context-invocation.md) (Execution Context Invocation), 2026-08-06.**
> This concept is superseded by the unified Invocation model. The
> `delegate`/`<-`/`act` model is absorbed into Execution Contexts:
> `delegate(obj)` becomes a context constructor, `<-` becomes the
> single-owner submission operator, `act` is no longer a concurrency
> modifier. Functions are colourless; execution policy is expressed via
> Execution Contexts. Retained for historical reference.

> **✅ ACCEPTED — [EDR-033](../how/decision_records/architecture/EDR-033-concurrency-model.md).**
>
> **Status:** Accepted 2026-07-27 (superseded 2026-08-06 by EDR-085).
>
> **⚠️ IMPLEMENTATION INDEPENDENCE GATE — Critical.** This concept must be
> defined without depending on a specific threading or async runtime model.
> See EDR-033 for gate validation details.
>
> **See also:** [`GLOSSARY.md`](../GLOSSARY.md) § Delegate, Actor, Mailbox,
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Ownership, Mutation,
> [`ERROR_HANDLING.md`](ERROR_HANDLING.md),
> [`TRAITS.md`](TRAITS.md),
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Modern software must exploit multi-core processors to scale, but classical concurrency models based on threads and mutexes introduce fundamental correctness problems:

- **Shared mutable state** — data races when two threads access the same memory without synchronisation.
- **Explicit locking** — `mutex`, `lock`, `semaphore` place the burden of correctness on the programmer.
- **Composable failures** — deadlocks, livelocks, priority inversion, and non-composable lock acquisition orders.
- **Global reasoning** — local logic becomes dependent on global synchronisation strategy, breaking modularity.

Orthon's core principle — **no shared mutable state** — already eliminates the root cause of data races within a single execution context. The concurrency model extends this guarantee to parallel execution: **delegate-based concurrency with message passing and no shared-state threads**.

## Principles

1. **No shared mutable state across delegates** — Each delegated context owns its state exclusively. No two delegates ever mutate the same memory concurrently.
2. **Single-threaded execution per delegate** — At most one message is processed at a time for any given delegate. Within a delegate, mutation is safe without locks.
3. **Parallelism from independence** — Two delegates that do not share state may execute on different cores simultaneously. The scheduler discovers this automatically.
4. **Explicit ownership transfer** — Data moving between delegates must use explicit ownership transfer. No implicit shared access across delegate boundaries.
5. **Actor is implementation, not syntax** — The term "actor" describes the runtime's internal representation. The programmer works with `delegate`, `<-`, and ownership transfer. No `actor` keyword appears in Orthon code.
6. **`act` modifier for concurrent entities** — Types that represent concurrent execution contexts use the `act` modifier on the type declaration.
7. **IMPLEMENTATION_INDEPENDENCE** — The model must be definable without depending on a specific threading, async, or runtime model. Different Implementation Strategies (default, embedded, high-performance) may use different runtime mechanisms as long as the semantics are preserved.

### Relationship to Concurrency (Plan 04-06)

The CONCURRENCY_MODEL (this document) defines the **language-level model** — `delegate`, `act`, `<-`, ownership transfer rules. The CONCURRENCY concept (Plan 04-06, important tier) will define **StdLib concurrency utilities** — channels, timers, async I/O, structured concurrency primitives built on top of this model. This document defines the foundation; Plan 04-06 builds on it.

### Relationship to ERROR_HANDLING

Error propagation across delegate boundaries follows the same `Result<T,E>` model defined in [`ERROR_HANDLING.md`](ERROR_HANDLING.md). A message sent to a delegate returns `Result<T, DelegateError>`. Errors in message processing are propagated to the sender.

### Relationship to TRAITS

Delegates support trait dispatch: a delegate can implement traits, and trait methods can be invoked on delegate references via the `<-` operator. This enables polymorphic message passing.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Dispatch Policy | Determines whether a delegated call is synchronous (immediate) or asynchronous (enqueued). Default: asynchronous for `<-` |
| Concurrency Policy | Defines the scheduling model — work-stealing, work-sharing, or pinned-to-thread |
| Allocation Policy | Governs how delegate-internal memory is allocated (per-delegate heap, arena, or shared pool) |
| Lifetime Policy | Defines when a delegate is destroyed — when the owning reference goes out of scope, or when all references are dropped |
| Ownership Transfer Policy | Governs how data crosses delegate boundaries — by move (`$`/`move`) or by immutable reference (read-only borrow) |
| Isolation Policy | Ensures no shared mutable state across delegate boundaries |

## Model (What)

### Delegate Declaration

```orthon
act Counter    # `act` modifier declares a concurrent type
    value: Int
    
proc increment(self)
    self.value += 1

fun get_value(self) -> Int
    return self.value
```

The `act` modifier on the type declaration signals that instances of this type are concurrent — their state is isolated and accessed via message passing.

### Delegate Creation and Message Passing

```orthon
# Create a delegate
let counter = act Counter(0)

# Send a message (asynchronous by default)
counter <- increment()
counter <- increment()

# Send a message and wait for result
let result = counter <- get_value()    # result = 2
```

The `<-` operator enqueues a message on the delegate's mailbox. For void-returning messages (`increment`), the call is fire-and-forget. For value-returning messages (`get_value`), the call returns a future that resolves when the message is processed.

### Memory Isolation

Within a single delegate, the programmer writes normal imperative code without locks:

```orthon
proc increment(self)
    self.value += 1     # safe — no concurrent access
    self.log.append("+1")
```

The runtime guarantees that `self.value += 1` executes atomically with respect to other messages on the same delegate. No `mutex`, no `lock`, no `atomic` — the isolation is structural.

### Cross-Delegate Communication

```orthon
let receiver = act Collector()
let data = $large_buffer    # ownership extracted from local scope
receiver <- process($data)   # ownership transferred into the message
```

The ownership transfer (`$`) ensures the sending delegate loses access to `data` and the receiving delegate gains exclusive ownership.

### Automatic Parallelism

```orthon
let account_a = act BankAccount(100)
let account_b = act BankAccount(200)

account_a <- deposit(50)
account_b <- withdraw(30)
```

The runtime can execute both messages in parallel because `account_a` and `account_b` own disjoint state. This is automatic — the programmer does not annotate parallelism.

## Default Strategy

Default scheduling uses a work-stealing thread pool with one OS thread per available core. Each delegate has a lock-free mailbox (single-producer, single-consumer queue). Messages are processed in FIFO order per delegate. Per-delegate allocation uses a local heap for cache locality.

## Alternative Strategies

| Strategy | Trade-offs |
|---|---|
| **Pinned-to-thread scheduling** | Better cache locality; worse load balancing. Suitable for embedded targets with fixed core assignment. |
| **Stackful coroutines (fibers)** | More delegates than threads possible (N:M scheduling). More complex runtime. |
| **Single-threaded** | All delegates on one thread. Deterministic execution, no parallelism. Suitable for embedded targets. |
| **Shared-memory with locks** | Standard thread/mutex model. Rejected for Orthon — violates "no shared mutable state" principle. |

## Open Questions

1. Should delegated calls support timeouts and cancellation?
2. How should backpressure be handled when a delegate's mailbox grows unbounded?
3. Should delegates support supervision trees (Erlang/OTP-style) for fault tolerance?

## Decision History

- **2026-07-27:** Accepted via EDR-033. Delegate-based concurrency model with `act` modifier, `<-` message passing, and explicit ownership transfer. No shared-state threads. Cross-ref with CONCURRENCY (Plan 04-06) noted — that plan will define StdLib concurrency utilities on top of this model. Cross-ref with ERROR_HANDLING (EDR-020) and TRAITS (EDR-019) established.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/ERROR_HANDLING.md` — cross-reference
- [x] `what/concepts/TRAITS.md` — cross-reference
- [ ] `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
