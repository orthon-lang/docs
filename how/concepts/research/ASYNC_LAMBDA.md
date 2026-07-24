# Hypothesis: Async Lambda as First-Class Coroutine

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-24

## Problem

Within the composition model (where `proc`/`fun`/`new` are semantic categories and `async` is an execution modifier), ordinary lambdas cannot contain `await`. When passing a callback to `map`, `forEach`, an event handler, or any higher-order function (HOF), the body cannot suspend. This forces developers to either:

1. Declare a separate named `async` function — polluting the namespace.
2. Use blocking calls inside the lambda — defeating the purpose of async.
3. Chain `Future` combinators manually — harming readability.

Additionally, creating a separate function loses the enclosing scope's closure, requiring explicit parameter passing. Short asynchronous operations — updating UI after a fetch, transforming a collection of URLs into data in parallel — become unnecessarily verbose without inline async lambdas.

## Examples from Other Languages

| Language | Async Lambda Support | Notes |
|----------|---------------------|-------|
| **C#** | `async (x) => { await ... }` | Full support; returns `Task<T>` |
| **JavaScript** | `async (x) => { await ... }` | Full support; returns `Promise<T>` |
| **Python** | No `async lambda` | `async def` only for named functions; rejected PEP |
| **Kotlin** | `suspend` lambdas | Functional equivalent; `async` is not a keyword |
| **Rust** | `async \|x\| { ... }` | Added in Rust 1.85 (2025); returns `impl Future` |
| **Swift** | No inline async closures | Requires explicit `Task { await ... }` wrapper |

**Novelty of the Orthon hypothesis** lies in the systematic combination of:

- **Separation of semantics and execution** — `async` is not a new category but a modifier on `proc`/`fun`/`new`. The compiler uniformly infers the category and applies the same rules (e.g., `exclusive async proc` lambda).
- **Category-preserving** — an `async` lambda is still classified as `proc`, `fun`, or `new` based on its body (mutation, return of new object). This means `map` can accept an `async fun` lambda and return a list of `Future<T>`.
- **Closure support with mutation** — an `async proc` lambda can mutate captured variables while remaining anonymous, integrated into the effect system.

## Proposal

Introduce anonymous coroutine syntax with the `async` keyword before the parameter list:

```orthon
// Basic form
(params) -> async { body with await }

// With explicit typing
async (x: Int, y: Int) -> Future<Int> { await f(x) + await g(y) }
```

Rules:

- The body may contain `await` at any point.
- The return value is implicitly wrapped in `Future` (type is inferred).
- An `async` lambda is a coroutine — its execution may be suspended and resumed.
- It returns `Future<T>` on call (even without `await`), but actual execution begins only on explicit launch (via `await` or `spawn`).
- Category (`proc`/`fun`/`new`) is inferred from the body, exactly as for synchronous lambdas.

### Canonical Forms

```orthon
// async fun lambda — pure computation
let fetchPage = async (url: Url) -> String { await http.get(url) }

// async proc lambda — mutating action
let logAndSave = async proc (data) -> { await log(data); await db.save(data) }

// async new lambda — factory
let createWidget = async new (config) -> Widget { await loadTemplate(config) }

// With exclusivity modifier
let guarded = exclusive async proc (obj) -> { await obj.save() }
```

## Implications for Orthon

### Advantages

- **Uniformity** — anonymous and named coroutines follow the same rules, reducing cognitive load.
- **Functional composition** — `map`, `filter`, `reduce` work with async operations without ceremony:
  ```orthon
  let results = urls.map(async (url) -> await download(url))
  let first = await results.first()
  ```
- **Async closures** — event handlers and callbacks with suspension are natural:
  ```orthon
  button.onClick = async () -> {
      let data = await loadData()
      updateUI(data)
  }
  ```
- **Ad-hoc parallelism** — `spawn` accepts an `async` lambda directly, launching it concurrently without a separate declaration.
- **Less naming** — no need to name disposable async operations; code stays local.
- **Category inference** — the compiler auto-detects whether the lambda is `proc`, `fun`, or `new` and applies appropriate checks (e.g., disallowing `proc` lambda calls from a `fun` context).

### Trade-offs

| Trade-off | Description |
|-----------|-------------|
| **Type inference complexity** | Async lambdas with closures and modifiers (`exclusive`, `transaction`) produce complex types, potentially slowing compilation. |
| **Overhead** | Every `async` lambda creates a coroutine object, even for trivial bodies like `async (x) -> x+1`. The compiler may optimise when no `await` is present. |
| **API bifurcation** | HOFs must be explicitly declared as accepting async lambdas (or be generic). This may lead to duplicated APIs (`map` vs `mapAsync`) without effect polymorphism. |
| **Accidental fire-and-forget** | Calling an `async` lambda without `await` returns a `Future` that may never execute. Shared with all async functions. |
| **Debugging** | Coroutine stacks are more complex, especially with nested lambdas. |

### Combinations

`async` lambdas compose naturally with other modifiers and constructs:

- **Semantic categories** — `async` applies before `proc`/`fun`/`new`: `exclusive async proc (obj) -> { await obj.save() }`
- **Structured concurrency** — `scope { spawn async { ... } }`
- **Timeouts** — `await withTimeout(5s) { async { ... } }`
- **Async iterators** — an `async` lambda may act as a generator if it contains `yield`.
- **Future combinators** — return values are composable with `then`, `map`, etc.

## Alternatives

| Approach | Assessment |
|----------|------------|
| **Named async functions only** | Simplest, but causes namespace pollution and loses locality. |
| **Sync lambdas returning Future** | Cannot use `await` inline; requires manual `then` chaining. |
| **Monadic do-notation** (Haskell) | Expressive but lacks imperative `await` style and closure capture. |
| **Reactive streams** (Rx) | Overkill for single-shot async; designed for event streams. |
| **Effect systems** (Koka, Eff) | More powerful but requires deep language restructuring. |

The hypothesis chooses **explicit, lightweight syntax** that integrates well into an imperative-functional language without rewriting the standard library.

## Open Questions

1. Should HOFs use a separate name (`mapAsync`) or be overloaded via effect polymorphism?
2. Can the compiler reliably optimise `async` lambdas with no `await` to sync equivalents?
3. How does `async` lambda interaction with ownership work — does the lambda capture values by move or by ref?
4. Should `async` lambdas participate in structured concurrency scopes automatically?
5. What is the error-handling semantics — does an unhandled error in an `async` lambda propagate to the caller or to the spawning scope?
