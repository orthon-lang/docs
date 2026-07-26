# Hypothesis: Runtime Concurrency Model — Actor-Internals Behind `delegate`

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-26
>
> **Relationship to `DELEGATE.md`:** This document describes the **runtime
> internals** — how the scheduler, mailbox, and memory isolation work under
> the hood. [`DELEGATE.md`](./DELEGATE.md) describes the **language surface** —
> the `delegate` keyword, the `<-` operator, and the "state owner" principle.
> Together they define Orthon's concurrency model: the programmer writes
> `delegate` and ownership transfers, the runtime implements actor semantics.
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

Modern software must exploit multi-core processors to scale, but classical
concurrency models based on threads and mutexes introduce fundamental
scalability and correctness problems:

- **Shared mutable state** — data races when two threads access the same
  memory without synchronisation.
- **Explicit locking** — `mutex`, `lock`, `semaphore` place the burden of
  correctness on the programmer.
- **Composable failures** — deadlocks, livelocks, priority inversion, and
  non-composable lock acquisition orders.
- **Global reasoning** — local logic becomes dependent on global
  synchronisation strategy, breaking modularity.

Orthon's core principle — **no shared mutable state by default** — already
eliminates the root cause of data races within a single execution context.
But without a concurrency model, this principle only applies sequentially.
The challenge: **how does Orthon extend this safety guarantee to parallel
execution without introducing explicit synchronisation primitives?**

## Relationship to DELEGATE.md

[`DELEGATE.md`](./DELEGATE.md) defines the language surface:

| Surface concept | Form |
|---|---|
| Delegation keyword | `delegate(owner)` |
| Message send operator | `owner <- message` |
| State owner principle | Delegate applies to the owner of mutable state, not to arbitrary code |

This document defines what happens when the runtime processes
`owner <- message`:

1. How the runtime represents each delegated context internally.
2. How messages are delivered, queued, and executed.
3. How the scheduler maps delegates to OS threads and cores.
4. How memory isolation is guaranteed without locks in user code.
5. How explicit ownership transfer (`$`/`move`) moves data between delegates.

The hypothesis: **each delegated context is implemented internally as an
actor** — with isolated state, a mailbox, and single-threaded message
processing — but the programmer never writes `actor` or manages mailboxes
directly. The actor model is a runtime implementation strategy, not a
language keyword.

## Principles

1. **No shared mutable state across delegates** — Each delegated context owns
   its state exclusively. No two delegates ever mutate the same memory
   concurrently.

2. **Single-threaded execution per delegate** — At most one message is
   processed at a time for any given delegate. Within a delegate, `balance +=
   amount` is safe without `mutex`, `lock`, or `atomic` because there is no
   concurrent access to its state.

3. **Parallelism from independence** — Two delegates that do not share state
   may execute on different cores simultaneously. The scheduler discovers
   this automatically; the programmer does not annotate parallelism.

4. **Explicit ownership transfer** — Data moving between delegates must use
   explicit ownership transfer (`$` or `move`). No implicit shared access
   across delegate boundaries.

5. **Actor is implementation, not syntax** — The term "actor" describes the
   runtime's internal representation. The programmer works with `delegate`,
   `<-`, and ownership transfer. No `actor` keyword appears in Orthon code.

6. **Concurrency is not a library** — The concurrency model is part of the
   language specification (Core layer), not an add-on library. The `delegate`
   keyword, the `<-` operator, and the ownership transfer rules are defined
   in Core. The scheduler, mailbox, and work-stealing algorithm are runtime
   implementation details.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Dispatch Policy | Determines whether a delegated call is synchronous (immediate) or asynchronous (enqueued). Default: asynchronous for `<-`. |
| Concurrency Policy | Defines the scheduling model — work-stealing, work-sharing, or pinned-to-thread. Default: work-stealing. |
| Allocation Policy | Governs how delegate-internal memory is allocated (per-delegate heap, arena, or shared pool). |
| Lifetime Policy | Defines when a delegate is destroyed — when the owning reference goes out of scope, or when all references are dropped. |
| Ownership Transfer Policy | Governs how data crosses delegate boundaries — by move (`$`/`move`) or by immutable reference (read-only borrow). |

## Model (What)

### Internal Architecture

When the programmer writes:

```
let counter = delegate(Counter(0))
counter <- increment()
counter <- getValue()
```

The runtime internally creates an **actor** with:

- **Isolated heap** — The `Counter` instance lives in a per-delegate allocation
  region. No other delegate can reference it directly.
- **Mailbox** — Incoming messages (`increment`, `getValue`) are enqueued in a
  lock-free queue.
- **Message loop** — A single OS thread (or fiber) dequeues and processes
  messages one at a time.
- **State** — `count` is mutated by the message loop only. No locks needed.

```
Delegate "counter"
┌──────────────────────────────┐
│  Mailbox                     │
│  ┌─────────┐ ┌─────────┐    │
│  │increment│ │getValue │... │
│  └─────────┘ └─────────┘    │
│         │        │          │
│         ▼        ▼          │
│  Message Loop (single thread)│
│         │                   │
│         ▼                   │
│  State: count = 0           │
└──────────────────────────────┘
```

### Memory Isolation Guarantee

Within a single delegate, the programmer writes normal imperative code:

```
proc increment(self)
    self.count += 1     // safe — no concurrent access
    self.log.append("+1")
```

The runtime guarantees that `self.count += 1` and `self.log.append("+1")`
execute atomically with respect to other messages on the same delegate.
No `mutex`, no `lock`, no `atomic` — the isolation is structural.

### Parallelism from Independence

Given two independent delegates:

```
let accountA = delegate(BankAccount(100))
let accountB = delegate(BankAccount(200))

accountA <- deposit(50)
accountB <- withdraw(30)
```

The runtime can execute both messages in parallel because:

- `accountA` and `accountB` own disjoint state.
- No message in `accountA`'s mailbox references `accountB`'s state.
- The scheduler detects independence and distributes across cores.

```
Core 1          Core 2
│               │
accountA ←      accountB ←
deposit(50)     withdraw(30)
│               │
▼               ▼
```

This is automatic. The programmer does not specify parallelism — it emerges
from the absence of conflict.

### Cross-Delegate Communication via Ownership Transfer

When data must move between delegates, explicit ownership transfer is used:

```
let receiver = delegate(Collector())
let data = $largeBuffer    // ownership extracted from local scope
receiver <- process($data) // ownership transferred into the message
```

The ownership transfer (`$`) ensures:

1. The sending delegate loses access to `data`.
2. The receiving delegate gains exclusive ownership.
3. No shared mutable state exists at any point.
4. The transfer is syntactically visible — no accidental cloning.

For immutable data, the runtime can optimise by passing a read-only reference
instead of copying. The ownership system guarantees the data is not mutated
during the borrow.

### CPU-Bound Workloads

For compute-intensive tasks, the programmer creates many independent
delegates, each owning a partition of the input:

```
let tiles = [
    delegate(ImageProcessor(tileData[0])),
    delegate(ImageProcessor(tileData[1])),
    delegate(ImageProcessor(tileData[2])),
    delegate(ImageProcessor(tileData[3])),
]

for tile in tiles
    tile <- process()
```

The work-stealing scheduler distributes these delegates across all available
cores. If the machine has 4 cores, all 4 tiles may run simultaneously.
If the machine has 2 cores, the scheduler time-slices accordingly — the
programmer's code does not change.

### IO-Bound Workloads

Since `delegate` is already asynchronous (the `<-` operator enqueues a
message and returns immediately), IO-bound operations integrate naturally:

```
let fileActor = delegate(FileHandler("log.txt"))
let socketActor = delegate(SocketHandler(":8080"))

fileActor <- write("request received")
socketActor <- accept()     // non-blocking at the delegate level
```

The IO-bound delegate blocks internally (waiting for disk or network), but
other delegates continue executing on the same core. The scheduler uses this
to maximise utilisation.

## Default Strategy

The default runtime implementation uses:

- **Work-stealing scheduler** — Each OS thread maintains a local queue of
  delegates. When a thread's queue is empty, it steals from another thread's
  queue. This adapts automatically to load imbalance.
- **Lock-free mailbox** — Each delegate's message queue is a lock-free
  multi-producer, single-consumer (MPSC) channel. Multiple senders may
  enqueue concurrently; the single consumer (the delegate's message loop)
  dequeues without contention.
- **Per-delegate heap** — Each delegate allocates from its own memory region,
  avoiding global allocation contention. Large objects may use a shared pool
  with ownership tracking.
- **Single-threaded message processing** — Each delegate processes one message
  at a time. No preemption within a message handler. This eliminates the need
  for user-space synchronisation.

## Alternative Strategies

| Strategy | Description | Trade-offs |
|---|---|---|
| **Work-sharing scheduler** | Incoming delegates are distributed to threads immediately; each thread has its own queue but no stealing. | Lower overhead for predictable workloads; worse load balancing for bursty or heterogeneous loads. |
| **Pinned delegates** | Delegates are pinned to specific OS threads or cores (no migration). | Predictable cache locality; poor utilisation if pinned delegate is idle. Useful for real-time or latency-sensitive contexts. |
| **Shared heap with GC** | All delegates allocate from a shared heap with a garbage collector. | Simpler memory model; GC pauses affect all delegates. Suitable for embedded strategy where throughput is secondary to simplicity. |
| **Fiber-based execution** | Each delegate runs on a user-space fiber; multiple fibers are multiplexed onto OS threads. | Lower context-switch overhead than OS threads; more complex scheduler. May be optimal for IO-heavy workloads. |
| **Single-threaded (no parallelism)** | All delegates execute on a single OS thread. | Maximum simplicity for embedded or sequential targets; no parallelism. |

## Alternative Concurrency Models

| Model | Languages | How it differs from Orthon's model |
|---|---|---|
| **CSP (Communicating Sequential Processes)** | Go (goroutines + channels) | Concurrency is built around channels, not state-owning entities. Go's goroutines are not tied to ownership — they communicate by sending values over channels. Orthon's model centres on the *state owner* as the unit of serialisation. |
| **Shared Memory + Threads** | C++, Java, Rust | Programmers manage threads and synchronisation explicitly. Maximum control, but correctness depends on discipline. Orthon rejects this for default-safety. |
| **Fork/Join** | Java `ForkJoinPool`, C++ TBB | Designed for divide-and-conquer parallelism. Can be implemented as a library pattern on top of Orthon's delegate model — fork creates child delegates, join awaits completion. |
| **Dataflow** | TensorFlow, Dask | Computations trigger when all inputs are ready. Can be implemented as a high-level framework on top of `delegate`, not a replacement for the execution model. |
| **Structured Concurrency** | Kotlin coroutines, Java Loom | Manages lifetime and cancellation of concurrent tasks through scoped hierarchies. Complementary — can be layered via a `context` mechanism without modifying the base execution model. |

## Open Questions

1. **Work-stealing vs work-sharing** — Should the default strategy use
   work-stealing (better load balancing) or work-sharing (lower overhead)?
   The answer may depend on the implementation strategy (Default vs Embedded
   vs High-Performance).

2. **Mailbox capacity and backpressure** — Should mailboxes have bounded
   capacity? What happens when the sender outpaces the consumer — block the
   sender, drop messages, or apply backpressure to the scheduler?

3. **Cancellation** — How does the programmer cancel an in-flight message on
   a delegate? Is cancellation explicit (`delegate.cancel(msg)`) or
   implicit (via scope exit)?

4. **Priority scheduling** — Should messages support priorities (urgent
   messages processed before routine ones)? How does priority interact with
   fairness?

5. **Delegate lifecycle** — Is a delegate destroyed when the last reference
   goes out of scope (like RAII), or does it persist until explicitly shut
   down? What happens to unprocessed messages in the mailbox?

6. **Blocking within a message handler** — If a delegate's message handler
   blocks (e.g., waiting for IO), should the runtime multiplex another
   delegate onto the same thread (fiber-style), or does the thread block?

7. **Error propagation across delegates** — If a delegate's message handler
   panics, what happens to the delegate's state? To other delegates that
   depend on it?

8. **Serializability vs parallelism** — How does the runtime determine that
   two delegates are independent? Does it use static analysis (ownership
   types guarantee separation) or runtime checks?

## References

- [`DELEGATE.md`](./DELEGATE.md) — language surface counterpart: the `delegate`
  keyword, `<-` operator, and "state owner" principle
- [`OWNERSHIP.md`](./OWNERSHIP.md) — ownership model: each value has exactly
  one owner, ownership can be moved, borrowing creates references
- [`OWNERSHIP_TRANSFER_OPERATOR.md`](./OWNERSHIP_TRANSFER_OPERATOR.md) —
  explicit syntax for ownership transfer (`$`/`move`), the primary mechanism
  for cross-delegate data movement
- [`EXECUTION_MODEL.md`](../../../../what/EXECUTION_MODEL.md) — broader execution
  semantics, execution targets, and the semantics-vs-implementation boundary
- [`../../../../notes/actor-implementation-taxonomy.md`](../../../../notes/actor-implementation-taxonomy.md) —
  taxonomy of 12 actor implementation approaches catalogued during design-space
  exploration
- [`../../../../how/strategies/DEFAULT_STRATEGY.md`](../../../../how/strategies/DEFAULT_STRATEGY.md) —
  default implementation strategy profile
- [`../../../../how/strategies/EMBEDDED_STRATEGY.md`](../../../../how/strategies/EMBEDDED_STRATEGY.md) —
  strategy for resource-constrained targets with single-threaded execution
- [`../../../../how/strategies/HIGH_PERFORMANCE_STRATEGY.md`](../../../../how/strategies/HIGH_PERFORMANCE_STRATEGY.md) —
  strategy for throughput-optimised targets with full work-stealing
