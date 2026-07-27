# Async/Await — Coroutine Execution with Orthogonal Modifier

> **✅ ACCEPTED — [EDR-047](../how/decision_records/architecture/EDR-047-async-await.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`CONCURRENCY_MODEL.md`](CONCURRENCY_MODEL.md),
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Evaluation,
> [`GLOSSARY.md`](../GLOSSARY.md) § Async, Await, Future,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How do you perform blocking I/O without blocking the OS thread?

Asynchronous programming requires expressing suspension without syntactic overhead. The programmer wants to write straight-line code; the compiler should manage the suspend/resume lifecycle. Existing approaches evolved through:

1. **Callbacks** — "callback hell": pyramid of nesting, inversion of control, no error propagation.
2. **Futures/Promises** — composable chains, but still higher-order logic; `then()` chains require typing transformations.
3. **Coroutines with `async`/`await`** — the compiler rewrites the function into a state machine; the syntax is linear. This is the proven solution adopted by Rust, JavaScript, Python, C#, and Kotlin.

## Principles

1. **`async` is an execution modifier, not a semantic category** — `async` modifies `proc`/`fun`/`new` to indicate suspension capability. It does not create a new declaration kind.

2. **Syntactic transparency** — `async` code should read like sequential code; `await` is the only visible marker of suspension.

3. **Colourless by default** — `async` functions return `Future<T>`. `Future` is a first-class value; `await` is required only when the caller needs the result value. This eliminates the function colouring problem.

4. **No stackful coroutines** — Stackless coroutines (state machine transformation) are the default. The compiler transforms async functions into state machines without separate stacks.

5. **Cooperative scheduling** — No pre-emption; yield points are at `await` boundaries only.

6. **Structured concurrency** — Spawned tasks have a defined lifetime scoped to their parent via `scope` blocks.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Executor Policy | Determines the default executor (single-threaded event loop, multi-threaded work-stealing, embedded) |
| Colouring Policy | Governs whether async/sync colouring is strict or permissive (default: colourless — Future as first-class value) |
| Yield Policy | Defines what constitutes a yield point (only `await`, or also allocation boundaries) |
| Scheduling Policy | Determines how async tasks are scheduled (cooperative, pre-emptive, work-stealing) |

## Model (What)

### `async` as Modifier

`async` marks a function that may suspend. It is a modifier on any of the three declaration kinds:

```orthon
async proc save(self)          # async procedure (may mutate self)
async fun fetch(url) -> String # async function (returns Future[String])
async new create(name) -> T    # async constructor
```

The modifier is orthogonal to the declaration kind. An `async proc` may suspend during mutation; an `async fun` may suspend during computation.

### `await` as Suspension Point

`await` marks a suspension point. The compiler transforms the function body into a state machine. Execution suspends at `await` and resumes when the awaited future completes.

```orthon
async fun fetchData(url: Url) -> String
    let response = await httpClient.get(url)
    return response.body
```

### Colourless Async — Future as First-Class Value

An `async` function returns `Future<T>`. Calling an `async` function without `await` creates a `Future` and continues immediately:

```orthon
# Await required only when result is needed
let future = fetchData(url)     # Future[String], no suspension
let result = await future       # suspension, unwrap to String

# Functional composition with futures
let futures = urls.map(fetchData)  # List of Future[String]
let results = futures.map(await)   # List of String (waits for each)
```

Rule: **`await` is required only when the current code needs the result.**

### `spawn` for Parallel Execution

`spawn` creates a new concurrent task running in parallel with the current one:

```orthon
async fun loadAll() -> Pair[Image, Image]
    let t1 = spawn async loadImage("a.jpg")
    let t2 = spawn async loadImage("b.jpg")
    let img1 = await t1
    let img2 = await t2
    return (img1, img2)
```

Without `spawn`, calling an `async` function executes sequentially in the current context. `spawn` makes parallelism syntactically visible.

### Structured Concurrency — `scope`

A block where all spawned tasks are automatically awaited or cancelled on exit:

```orthon
scope {
    let t1 = spawn async loadA()
    let t2 = spawn async loadB()
}
# t1 and t2 guaranteed completed (or cancelled on error)
```

### Async Lambdas

Anonymous coroutines compose with higher-order functions:

```orthon
numbers.map(async fun(x) -> await fetch(x))
```

### Task Cancellation and Timeouts

Explicit `cancel()` on `Task` with `is_cancelled` check. Timeout syntax for bounded wait:

```orthon
await socket.read(timeout: 5s)
```

### `exclusive` Modifier for Access Serialisation

Separates concurrency (`async`) from access control (`exclusive`):

```orthon
exclusive async proc save(self)    # async save with exclusive object access
exclusive proc update(self)        # synchronous update with exclusive access
```

## Default Strategy

Stackless coroutines with a single-threaded executor (event loop). `await` is the only yield point. State machine compilation — no per-coroutine heap allocation. `Future` is a compiler-managed type with single-subscriber semantics.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Multi-threaded executor | Tasks distributed across a thread pool (work-stealing). Appropriate for CPU-bound async work. |
| Stackful coroutines | Each coroutine has its own stack; can suspend from nested calls (Go-style). Higher memory overhead. |
| Embedded/no-colouring | Blocking IO with dedicated threads instead of async. Simplifies runtime for embedded targets. |

## Open Questions

1. How does `async` interact with ownership — what about non-Send types across `await` points?
2. Should `scope` support custom cancellation policies (timeout, errors propagate vs. isolate)?
3. How do async generators compose with the `emit` model (EDR-021)?
4. Should `Future` support multi-subscriber semantics (broadcast) or remain single-subscriber?

## Decision History

- **2026-07-27** — Accepted via EDR-047 (combined ASYNC_AWAIT + ASYNC_AS_EXPLICIT_MODIFIER). Classification: Language. Async as orthogonal modifier on `proc`/`fun`/`new`. Colourless model with Future as first-class value. `spawn` for explicit parallelism. `scope` for structured concurrency.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `../how/DESIGN_PRINCIPLES.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/EXECUTION_MODEL.md`
