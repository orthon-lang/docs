# Strengthened Hypothesis: Async as Orthogonal Coroutine Modifier

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-24

## Critical Revision

The original hypothesis contained two errors:

1. **"`async` ⇒ object state is unstable"** — This is incorrect. Asynchrony alone does not create state instability; it only indicates the possibility of suspension. Concurrent access is a separate dimension that must be expressed explicitly.

2. **"`async` cannot be called from a synchronous context"** — This is too restrictive. Returning `Future` without `await` is a useful idiomatic pattern for functional composition.

This document supersedes the earlier analysis and presents a corrected, strengthened hypothesis.

---

## 1. What `async` Actually Means

- **`async` means only one thing:** a method is a **coroutine** — its execution *may be suspended* at `await` points and resumed later. It says nothing about parallelism, multithreading, or concurrent access.
- A coroutine does not create a new task or thread; it runs within the current context (e.g., an event loop) and suspends for I/O or other async operations.
- `async` is an **execution modifier**, not a semantic category. It is therefore orthogonal to `proc`/`fun`/`new` and composes freely: `async proc`, `async fun`, `async new`.

---

## 2. Separating Concurrent Access from Async

The "instability" problem arises only when **operations that mutate state execute concurrently**. This is a property of **concurrent execution** (e.g., explicit `spawn` of multiple tasks), not of `async`.

Introduce an explicit modifier **`exclusive`** (or `locked`):

> A method marked `exclusive` requires **exclusive access** to the object for its duration; any other calls (including synchronous reads) must wait or block until the operation completes.

```orthon
exclusive async proc save()     # async save with exclusive object access
exclusive proc update()         # synchronous update with exclusive access
```

If a method is not marked `exclusive`, it is considered **non-blocking** with respect to state — the object may be used concurrently by other methods (responsibility for correctness falls on the developer, or the state is immutable, or the method is read-only).

This cleanly separates:

- **`async`** — suspendibility (coroutine)
- **`exclusive`** — access serialisation (locking)

---

## 3. Returning `Future` Without `await` — Flexibility

The requirement that `async` methods **must** be called with `await` or only from async contexts is eliminated. Instead:

- An `async` method always returns `Future<T>`.
- Calling an `async` method **without `await`** merely creates a `Future` and continues immediately — the coroutine is not yet started (or runs in the background, depending on runtime). This enables constructing transformation chains without immediate suspension.

```orthon
fun downloadAll(urls):
    futures = urls.map(async fun download(url))   # returns list of Future
    return futures                                 # pass along or combine
```

Here `download` is async, but we collect `Future` objects for later processing instead of awaiting each. This is natural in a functional style.

Rule: **`await` is required only when the current code needs the result** (i.e., the `Future` is unwrapped to a concrete value). Without `await`, `Future` is a first-class object.

This gives maximum flexibility and eliminates artificial context colouring — a synchronous function can call an async function and return a `Future` (known as "colourless async programming").

---

## 4. Launching Concurrent Tasks — Explicit `spawn`

A coroutine (`async` function) does not by itself create a new task — it runs in the current context when explicitly launched (via `await`) or scheduled. For concurrent execution of multiple coroutines **simultaneously**, introduce an explicit operator:

- **`spawn`** — creates a new task (a coroutine running in parallel with the current one) and returns `Task<T>` (which is also a `Future`).

```orthon
async proc main():
    t1 = spawn async loadImage("a.jpg")
    t2 = spawn async loadImage("b.jpg")
    img1 = await t1
    img2 = await t2
    # parallel loading
```

`spawn` explicitly indicates the creation of parallel work. Without `spawn`, calling an `async` function (even with `await`) executes sequentially in the same context.

This matches the Kotlin model (coroutines + `launch`/`async`) and separates *suspendibility* from *parallelism*.

---

## 5. Compositional Model: Modifiers as Dimensions

The base system with `proc`, `fun`, `new` is the **semantic dimension** (what the method does: mutate, compute, create). `async` is the **execution dimension** (coroutine). `exclusive` is the **access dimension** (serialisation). Additional dimensions can be added:

- `transaction` — method executes in a transaction (atomic, rollback on error).
- `reactive` — method returns an event stream (reactive).
- `cached` — result is cached.
- `distributed` — method executes remotely.

All modifiers are **independent** and can combine freely with `proc`/`fun`/`new`. The compiler treats them as a set of aspects; semantics are determined by the combination.

```orthon
exclusive async proc save()           # async save with locking
transaction async fun transfer()      # async computation in a transaction
cached fun compute()                  # synchronous cached computation
distributed new create()              # remote object creation
```

This makes the language conceptually uniform and extensible — new modifiers can be added without altering base semantics.

---

## 6. Additional Required Constructs

### Async Lambdas
Anonymous coroutines compose with higher-order functions:

```orthon
numbers.map(async fun(x) -> await fetch(x))
```

See [ASYNC_LAMBDA.md](ASYNC_LAMBDA.md) for detailed treatment.

### Async Iterators (`async for`)

```orthon
async for item in asyncStream:
    process(item)
```

### Async Generators
`yield` inside a coroutine produces an async stream.

### Task Cancellation
Explicit `cancel()` on `Task` with `is_cancelled` check.

### Timeouts
Syntax for bounded wait:

```orthon
await socket.read(timeout: 5s)
```

### Structured Concurrency — `scope`
A block where all spawned tasks are automatically awaited or cancelled on exit:

```orthon
scope {
    t1 = spawn async loadA()
    t2 = spawn async loadB()
}
# t1 and t2 guaranteed completed (or cancelled on error)
```

`scope` may be implemented as a context manager (like `asyncio.TaskGroup`), but syntactically it is a built-in block.

---

## 7. Summary

| Concept | Original Hypothesis | Strengthened Hypothesis |
|---------|-------------------|------------------------|
| `async` meaning | State instability | Coroutine (suspension only) |
| Async + sync boundary | Strict colouring | Colourless — `Future` as first-class value |
| Concurrent access | Implicit / conflated | Explicit `exclusive` modifier |
| Parallel execution | Implicit | Explicit `spawn` |
| Modifier system | Flat (`async proc`) | Orthogonal dimensions (`async`, `exclusive`, `transaction`, etc.) |
| Structured concurrency | Not addressed | `scope` block |
| Cancellation / timeouts | Not addressed | First-class constructs |

### Advantages

1. **Clear separation of concerns** — `async` = suspendibility, `exclusive` = access serialisation, `proc`/`fun`/`new` = semantics.
2. **Flexibility** — returning `Future` without `await` simplifies functional composition.
3. **Explicit parallelism** — `spawn` for creating tasks, no implicit assumptions.
4. **Composability** — modifiers are independent; the language is easily extensible with new aspects.
5. **Structure** — `scope` and timeouts make concurrent code safe and manageable.
6. **Uniformity** — all constructs (lambdas, iterators, generators) have async analogues, preserving language style.

This model corrects the errors in the original hypothesis and provides a robust foundation for async programming in a language with explicit semantic categories.

## Cross-References

- [ASYNC_AWAIT.md](ASYNC_AWAIT.md) — Coroutine execution model (stackless, cooperative)
- [ASYNC_LAMBDA.md](ASYNC_LAMBDA.md) — Anonymous async functions
- [CONCURRENCY.md](CONCURRENCY.md) — Concurrency model overview
- [EXECUTION_PROGRAM.md](EXECUTION_PROGRAM.md) — Execution program model
