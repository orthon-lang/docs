# Concurrency (Actors & Channels)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).

## Issue (Why)

How do you write correct programs that execute on multiple cores?

Concurrency with shared memory and locks produces:
- **Deadlocks** — circular wait between locks.
- **Livelocks** — threads yielding without progress.
- **Priority inversion** — low-priority thread holds a lock needed by high-priority.
- **Non-determinism** — interleaving-dependent behaviour that makes debugging probabilistic.

The root cause: **shared mutable state** accessed without coordination. Locking is a low-level mechanism that addresses symptoms, not the design.

## Principles

1. **Share nothing** — State is owned by a single actor/process; data is transferred, not shared.
2. **Message passing** — Communication happens via messages over channels, not shared memory.
3. **Selective receive** — An actor can wait for specific message patterns.
4. **Supervision** — Failure is isolated to the actor and handled by its supervisor.
5. **No data races by construction** — The type system prevents race conditions statically.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Concurrency Model Policy | Selects between actor model, CSP, or shared-memory threading |
| Isolation Policy | Governs whether state is isolated per-actor or shared with synchronisation |
| Failure Policy | Determines supervisor behaviour — restart, stop, escalate, or log |

## Model (What)

Concurrency follows the **Actor Model** (Erlang-style) and **CSP** (Go-style). Computation is organised into isolated actors that communicate through typed channels.

```
actor Counter:
    var count: Int = 0
    
    handle Increment:
        count += 1
    
    handle GetCount(sender: Chan<Int>):
        sender.send(count)

// Spawn an actor
let counter = spawn(Counter())
counter.send(Increment)
```

Key features:
- **Actors** own state; no external direct access.
- **Channels** for message passing (typed, buffered or unbuffered).
- **Select** — wait on multiple channels.
- **Supervision trees** — actors have supervisors that define failure recovery.
- **Shared-nothing** — no shared memory concurrency; state transfer via messages only.

## Default Strategy

Actor model with typed channels and a supervision tree. Each actor owns its heap; garbage collection is per-actor (generational). No shared-memory primitives in the default strategy.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Channel-only (CSP) | No actors; goroutine-like lightweight threads with channel communication only. |
| Software Transactional Memory | Atomic memory transactions with optimistic locking (Clojure, Haskell). |
| Shared memory with locks | Mutexes, RWLock, atomics for low-level control (opt-in, unsafe). |
| Data parallelism | SIMD and parallel iteration (map/reduce over collections). |

## Open Questions

1. Should actor supervision be part of the language or a library?
2. How to prevent mailbox overflow? (Backpressure protocol.)
3. Should channels be synchronous (rendezvous) or buffered?
4. Interaction with async/await — can an actor be async?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
