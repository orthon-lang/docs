# Actors

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language protect shared mutable state in concurrent code without relying on error-prone manual locking?

Classical approaches to shared-state concurrency each have well-known failure modes:

- **`synchronized` / mutex locks (Java, C++)** — Deadlocks, livelocks, priority inversion, and non-deterministic interleavings. The programmer must manually pair every lock acquisition with a release (or use `synchronized` blocks), and must ensure correct lock ordering across all code paths.

- **Read-write locks (Java `ReadWriteLock`, Rust `RwLock`)** — Better read concurrency, but still vulnerable to writer starvation and the same ordering constraints.

- **Concurrent collections (Java `ConcurrentHashMap`)** — Reduced risk but do not compose. A two-step operation (check-then-act) across collections still requires external synchronisation.

- **Atomic operations (Java `AtomicInteger`, C++ `std::atomic`)** — Fine-grained but do not generalise to multi-field invariants.

The root cause: **shared mutable state with explicit locking** puts the burden of correctness on the programmer for every concurrent access. The compiler cannot verify that locks are used correctly.

**Actors** offer an alternative model: state is isolated within an actor, and concurrent access happens through message passing, not shared memory:

```java
// Java — manual locking
class Counter {
    private int value;
    public synchronized void increment() { value++; }
    public synchronized int get() { return value; }
}

// Swift — actor
actor Counter {
    private var value = 0
    func increment() { value += 1 }
    func get() -> Int { return value }
}
```

The compiler enforces that actor state is only accessed from within the actor's execution context, preventing data races statically.

## Examples

| Language | Actor mechanism | Enforcement | Reentrancy | Key innovation |
|---|---|---|---|---|
| **Swift** | `actor` keyword | Compiler-enforced isolation | Async reentrancy with `await` | Actor-isolation checking at compile time |
| **Erlang/Elixir** | `spawn` + message passing | Runtime isolation (no shared memory) | Always reentrant (process mailbox) | Supervision trees |
| **Pony** | Reference capabilities | Compiler-enforced (deny capabilities) | N/A (no shared state) | Reference capabilities prevent races at type level |
| **Kotlin** | Coroutines + `Mutex` | Runtime (library-level) | Manual | Structured concurrency |
| **Java** | None built-in | Runtime (`synchronized`) | Manual | Oldest, no modern actor support |
| **Akka (JVM)** | Actor library | Runtime (library-level) | Always reentrant | Distributed actors |

### Swift Actors

Swift's `actor` type is a language-level construct with compiler-enforced isolation:

```swift
actor BankAccount {
    private var balance: Double

    init(initialBalance: Double) { self.balance = initialBalance }

    func deposit(amount: Double) { balance += amount }

    func transfer(amount: Double, to other: isolated BankAccount) async {
        // The `isolated` parameter ensures only one actor is accessed
        // at a time — the compiler prevents simultaneous access
        balance -= amount
        await other.deposit(amount: amount)
    }
}

// Usage — actor isolation is enforced by the compiler
let account = BankAccount(initialBalance: 1000)
// account.balance  // compile error: actor-isolated property
await account.deposit(amount: 500)  // must use await
```

Key Swift actor features:
- **Actor isolation** — The compiler verifies that actor state is only accessed through `async` calls on the actor.
- **Non-reentrant by default** — An actor processes one message at a time, preventing data races.
- **`nonisolated`** — Methods that don't access actor state can be marked as `nonisolated`.
- **`MainActor`** — A global actor that ensures code runs on the main thread (replaces `DispatchQueue.main`).
- **`Sendable`** — Types that can be safely passed across actor boundaries are marked `Sendable`; the compiler checks this.

### Erlang Actors (Processes)

Erlang's actor model is the original and most battle-tested:

```erlang
% Erlang — actor (process)
counter() ->
    receive
        {increment, From} ->
            counter_loop(0, From)
    end.

counter_loop(Value, From) ->
    receive
        {get, From} ->
            From ! {value, Value},
            counter_loop(Value, From);
        {increment} ->
            counter_loop(Value + 1, From);
        stop ->
            ok
    end.
```

Each Erlang process has its own heap, so there is **no shared memory at all**. Processes communicate exclusively through messages. This eliminates data races entirely but requires explicit message handling logic.

### Pony: Reference Capabilities

Pony takes a unique approach: the type system enforces isolation through **reference capabilities**:

- `iso` — isolated reference (unique ownership, no aliases)
- `trn` — transition reference (write access, no alias read)
- `ref` — mutable reference (aliases allowed)
- `val` — immutable reference (aliases allowed, cannot mutate)
- `box` — read-only reference (aliases allowed, cannot mutate)
- `tag` — opaque handle (no read/write, can send across actors)

The compiler checks that actor fields are only accessible through capabilities that guarantee isolation. No runtime checking needed.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Concurrency Model Policy | Selects between actors, channels, or shared memory |
| Isolation Policy | Governs whether isolation is compile-time (Swift/Pony) or runtime (Erlang) |
| Reentrancy Policy | Determines whether actors are reentrant during `await` points |
| Failure Policy | Governs actor lifecycle — restart, stop, escalate |

## Implications for Orthon

1. **Actors as the concurrency primitive** — If Orthon follows Swift's model, `actor` could be a built-in type (like `struct` or `class`). This makes concurrent programming safer by default — the compiler enforces isolation.

2. **Compiler-enforced isolation** — A key advantage of the actor model over Java's `synchronized`: the compiler can verify that actor state is not accessed from outside the actor. This eliminates an entire class of concurrency bugs.

3. **Integration with async/await** — Actors naturally pair with async/await (see [`ASYNC_AWAIT.md`](ASYNC_AWAIT.md)): method calls on actors are asynchronous, and `await` suspends until the actor processes the call.

4. **Value types and actors** — If Orthon uses value semantics by default (see [`VALUE_SEMANTICS.md`](VALUE_SEMANTICS.md)), passing values between actors is naturally safe: the value is copied, so there is no shared state. Reference types passed between actors would need `Sendable`-like compiler checking.

5. **Comparison with structured concurrency** — [`CONCURRENCY.md`](CONCURRENCY.md) covers structured concurrency and channels. Actors complement this: structured concurrency manages task lifetimes; actors manage state isolation. They solve different sub-problems.

## Open Questions

1. Should Orthon have a built-in `actor` type, or should actors be a library (like Akka for JVM)?
2. If built-in, should actor isolation be enforced at compile time (Swift model) or runtime (Erlang model)?
3. How does actor isolation interact with Orthon's ownership/memory model (see [`OWNERSHIP.md`](OWNERSHIP.md))?
4. Should Orthon support `MainActor`-like global actors for thread-affine operations (UI, file I/O)?
5. Should actors support supervision (Erlang model) or rely on the enclosing structured concurrency scope for error handling?
6. How does reentrancy work? Swift actors are reentrant during `await` points; Erlang actors are always reentrant. What model fits Orthon?
