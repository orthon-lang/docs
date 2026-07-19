# Async/Await (Coroutines)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).

## Issue (Why)

How do you perform blocking I/O without blocking the OS thread?

Asynchronous programming evolved through:
1. **Callbacks** — "callback hell": pyramid of nesting, inversion of control, no error propagation.
2. **Futures/Promises** — composable chains, but still higher-order logic; `then()` chains become "callback spaghetti" in a different syntax.
3. **Coroutines with `async/await`** — the compiler rewrites the function into a state machine; the *syntax is linear*.

The core problem: **expressing suspension without syntactic overhead**. The programmer wants to write straight-line code; the compiler should manage the suspend/resume lifecycle.

## Principles

1. **Syntactic transparency** — `async` code should read like sequential code; `await` is the only visible marker of suspension.
2. **Colour-free by default** — Async functions should be callable from sync contexts (with a blocking wrapper) and vice versa.
3. **No stackful coroutines** — Stackless coroutines (state machine transformation) are the default; stackful is an alternative strategy.
4. **Cooperative scheduling** — No pre-emption; yield points are at `await` boundaries only.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Executor Policy | Determines the default executor (single-threaded, multi-threaded, custom) |
| Colouring Policy | Governs whether async/sync colouring is strict or permissive |
| Yield Policy | Defines what constitutes a yield point (only `await`, or also allocation boundaries) |

## Model (What)

`async` marks a function that may suspend. `await` marks a suspension point. The compiler transforms the function body into a state machine.

```
async func fetchData(url: Url) -> Result<String, HttpError>
    let response = await httpClient.get(url)
    return response.body
```

Key features:
- **Stackless coroutines** — function becomes a state machine; no separate stack per coroutine.
- **Cooperative multitasking** — suspension only at `await` points.
- **Executor-agnostic** — the async runtime is swappable.
- **Structured concurrency** — spawned tasks have a defined lifetime scoped to their parent.

## Default Strategy

Stackless coroutines with a single-threaded executor (event loop). Await is the only yield point. `async fn` signature colouring is enforced (can only call `async` from `async`).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Multi-threaded executor | Tasks distributed across a thread pool (work-stealing). |
| Stackful coroutines | Each coroutine has its own stack; can suspend anywhere (Go-style goroutines). |
| No colouring | Async functions transparently callable from sync context (blocking wrapper). |
| Structured concurrency | Task lifetime is scoped to parent scope; cancellation propagates to children. |

## Open Questions

1. Should `async` colouring be strict or should there be a bridge (`block_on`)?
2. Should cancellation be implicit (scope exit) or explicit (CancellationToken)?
3. How does async interact with ownership and borrowing? (non-Send types across await.)
4. Should the executor be part of the standard library or a third-party choice?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
