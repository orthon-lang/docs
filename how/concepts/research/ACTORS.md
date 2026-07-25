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
| Concurrency Model Policy | Selects between `act`-based isolation, channels, or shared memory |
| Isolation Policy | Governs whether isolation is compile-time (`act` modifier) or runtime (Erlang) |
| Reentrancy Policy | Determines whether `act` methods are reentrant during `await` points |
| Dispatch Policy | Determines whether `act` method calls use message-passing (`<-`) or direct invocation |
| Failure Policy | Governs actor lifecycle — restart, stop, escalate |

## Hypothesis for Orthon: `act` modifier on `class`, not a separate `actor` type

After research review, Orthon's concurrency model is **not** a separate `actor` type. Instead, it uses an `act` modifier on `class`:

```
class Counter
    private act value = 0

    act increment()
        value += 1

    func describe() -> String
        return "a counter"
```

### Key design decisions

1. **One reference type:** `class` is the only reference type. No separate `actor` keyword. The `act` modifier adds optional compile-time isolation to any `class`.

2. **Opt-in, per-field and per-method:** Only `act`-marked fields are isolated. Only `act`-marked methods can access them. Non-`act` methods are synchronous and pay no isolation overhead.

3. **Zero-cost by construction:** If a `class` has no `act` fields, no mailbox is created. No message loop exists. The class behaves as a plain reference type with no concurrency overhead.

4. **Explicit delegation via `<-` operator:** `act` methods cannot be called with the `.` operator (compile error). They must use the delegation operator `<-`:

    ```
    counter.increment()          // COMPILE ERROR: act method
    counter <- increment()       // OK: message send via mailbox
    ```

    This makes every concurrent dispatch syntactically visible at the call site.

5. **Integration with async/await:** The `<-` operator pairs with `await` for suspension:
    ```
    await (counter <- compute(x))
    ```

6. **Value types pass safely:** `struct` values passed as arguments to `act` methods are copied, eliminating shared-state concerns. `class` references passed to `act` methods require `Sendable`-like compiler checking.

7. **No class inheritance:** Classes derive only from `object`. Behaviour reuse through trait conformance, not inheritance chains. This eliminates the fragile base class problem.

### Comparison with Swift

| Aspect | Swift | Orthon (hypothesis) |
|--------|-------|---------------------|
| Keyword | `actor` | `class` + `act` modifier |
| Default isolation | All methods isolated | Only `act`-marked methods |
| Opt-out | `nonisolated` | Not needed (non-`act` = non-isolated) |
| Call syntax | `await actor.method()` | `target <- method()` |
| Zero-cost | Always allocates mailbox | Only if `act` fields exist |

### Comparison with Erlang

| Aspect | Erlang | Orthon (hypothesis) |
|--------|--------|---------------------|
| Isolation | Process-level (separate heap) | Field-level (`act` fields) |
| State sharing | None (copy everything) | Value types copy; ref types checked |
| Mailbox | Always (per process) | Only if `act` fields exist |
| Supervision | Built-in (supervision trees) | Open question |

### Comparison with Pony

| Aspect | Pony | Orthon (hypothesis) |
|--------|------|---------------------|
| Mechanism | Reference capabilities | `act` modifier |
| Enforced by | Type system | Modifier + compiler checks |
| Flexibility | Six capabilities | One modifier, binary |
| Learning curve | Steep | Shallow |

## Open Questions

1. **Reentrancy:** Are `act` methods reentrant during `await` points inside them? Swift is reentrant; Erlang is always reentrant. What model fits Orthon?

2. **Sendable:** How does the compiler verify that a `class` reference passed to an `act` method does not create shared mutable state outside the actor? Does Orthon need a `Sendable`-like constraint?

3. **`<-` semantics:** Does `target <- method()` block the caller until completion? Return a future/handle? Require `await`? What is the return type of a `<-` expression?

4. **Error handling:** If an `act` method panics/fails, does the enclosing actor die? Restart? Escalate to a supervisor?

5. **`act` on `struct`?** Could `act` theoretically apply to `struct` methods for thread-local isolation without heap allocation?

6. **`MainActor`-like global actors:** Should Orthon support thread-affine execution (UI updates, filesystem I/O) through a special `act` context?

7. **Reentrancy and invariants:** If an `act` method awaits and another `act` method enters, the isolation guarantee still holds (one at a time) but intermediate state is visible. Does this break invariants?

## See also

- [`CLASS_WITH_ACT.md`](CLASS_WITH_ACT.md) — the class + `act` hypothesis document
- [`STRUCT_AS_VALUE_TYPE.md`](STRUCT_AS_VALUE_TYPE.md) — the struct value type hypothesis
- [`ASYNC_AWAIT.md`](ASYNC_AWAIT.md) — async/await integration
- [`CONCURRENCY.md`](CONCURRENCY.md) — structured concurrency and channels
- [`OWNERSHIP.md`](OWNERSHIP.md) — ownership model interaction
