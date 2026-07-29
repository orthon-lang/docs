# Execution Policy Hypothesis — Unifying Invocation

> **⚠️ DRAFT — Exploratory hypothesis.**
> This document captures a design hypothesis under active exploration.
> It proposes removing the distinction between `call`, `delegate`, `spawn`,
> and `async`/`await`, replacing them with a single **Invocation** operation
> whose **execution policy** is selected by a syntactic operator at the
> call site.
>
> **Status:** Exploratory — not accepted.
> **Related:** [`DELEGATE.md`](DELEGATE.md) (current execution policy model),
> [`CALL_PRIMITIVE.md`](CALL_PRIMITIVE.md) (if exists), [`FUNCTIONS.md`](FUNCTIONS.md),
> [`CONCURRENCY_MODEL.md`](CONCURRENCY_MODEL.md) (current concurrency model, superseded if hypothesis accepted),
> [`EXECUTION_PROGRAM.md`](EXECUTION_PROGRAM.md)
> **See also:** [`../../what/PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) § 3.2.3 `call`

---

## Problem

Today, most languages introduce concurrency as a separate language feature:

- `async` / `await`
- `actor`
- `coroutine`
- `goroutine`
- `spawn`

Each introduces its own object model and execution semantics. As a result,
the programmer must decide *how* to represent computation before writing
business logic.

Orthon's current design already reduces this to `delegate` + `spawn`, but
still has three separate invocation mechanisms:

| Mechanism | Syntax | Semantics |
|-----------|--------|-----------|
| `call` | `fn(args)` | Immediate execution |
| `delegate` send | `obj <- msg(args)` | Asynchronous message via mailbox |
| `spawn` | `spawn fn(args)` | Parallel execution |

**Hypothesis:** All three are the same operation — **Invocation** — and the
syntactic operator selects the **Execution Policy**.

---

## Hypothesis

### Core Claim

There is no:
- function call
- delegate send
- parallel spawn
- async / await
- coroutine

There is only **Invocation**. A context constructor + operator selects
the **Execution Policy**.

```
fn(args)                  — Immediate         — выполнить сейчас (без контекста)
ctx = defer(obj)                                 корутина
ctx  <~ method(params)    — Deferred            отложить на контексте
await(ctx)                                       материализация

ctx = delegate(obj)                              mailbox
ctx  <- method(params)    — Delegated            делегировать владельцу
await(ctx)

ctx = parallel()                                 пул потоков
ctx  <= method(params)    — Parallel             независимое исполнение
await(ctx)

ctx = remote(url)                                удалённый узел
ctx  <@ method(params)    — Remote               удалённое исполнение
await(ctx)
```

Invocation becomes **two-dimensional**:

```
Что вызвать?    →  метод / функция
Как выполнить?  →  контекст + оператор
```

The function answers only the first question. The operator answers only
the second.

---

### Key Insight: Context Required

Only **Immediate** execution is available without a context. All other
policies require an **explicit execution context**.

The programmer must create the context first:

```orthon
let ctx = defer(obj)       # create a deferred/coroutine context
ctx <~ method(params)      # invoke method on this context
```

This mirrors the natural pattern: you create a tool, then use it.
`defer(...)`, `delegate(...)`, `parallel()`, `remote(url)` are all
**context constructors** — they create an execution context with the
desired policy.

---

### Syntax Design Principle

A critical observation from design exploration:

> The `<` operator feels foreign when placed **between** the function name
> and its argument list: `fn<OP(args)`. It feels natural when placed
> **between a context and a call**: `ctx <- fn(args)`.

| Position | Example | Feels |
|----------|---------|-------|
| Between name and args | `fn<OP(args)` | ❌ Инородно — разрывает естественную связь |
| Between context and call | `ctx <- fn(args)` | ✅ Нативно — соединяет контекст и действие |
| No operator needed | `fn(args)` | ✅ Нативно — имя и args рядом |

**Immediate** is the base case — no context, no operator. The standard
call syntax `fn(args)` is Immediate by default.

**Contextual policies** use a binary operator `<OP` between the execution
context and the call. This mirrors natural language: "context performs action."

---

### Syntax Table

| Form | Policy | Meaning |
|------|--------|---------|
| `fn(args)` | Immediate | Execute now, produce value |
| `ctx <~ fn(args)` | Deferred | Defer execution on coroutine context |
| `ctx <- fn(args)` | Delegated | Delegate to owner's mailbox |
| `ctx <= fn(args)` | Parallel | Execute on thread pool |
| `ctx <@ fn(args)` | Remote | Execute on remote node |
| `ctx <# fn(args)` | GPU | Execute on GPU device |

**Context constructors:**

| Constructor | Creates | Operator |
|-------------|---------|----------|
| `defer(obj)` | Deferred/coroutine context | `<~` |
| `delegate(obj)` | Delegated/mailbox context | `<-` |
| `parallel()` | Parallel thread pool | `<=` |
| `remote(url)` | Remote execution node | `<@` |
| — | Immediate (no context needed) | (none) |

---

### What This Replaces

#### `async` / `await` — eliminated

`async` is not a modifier on the function. The function is colourless.
`defer(obj)` creates a coroutine context; `<~` invokes on it.

```orthon
# Before:
async fn fetch(url) -> String
    return await httpClient.get(url)
let data = await fetch(url)

# After:
fun fetch(url) -> String           # colourless function
    return httpClient.get(url)

let ctx = defer()                   # create deferred context
ctx <~ fetch(url)                   # invoke on context
let data = await(ctx)               # materialise result
```

`await(...)` is a regular function that reads a result from any execution
context. It is not a keyword — it blocks or yields depending on the
context type and the ambient execution environment.

#### `spawn` — eliminated

```orthon
# Before:
let t1 = spawn async loadImage("a.jpg")
let t2 = spawn async loadImage("b.jpg")

# After:
let pool = parallel()
pool <= loadImage("a.jpg")
pool <= loadImage("b.jpg")
let results = await(pool)
```

#### `delegate send` — unified

```orthon
# Before:
let counter = delegate(Counter(0))
counter <- increment()

# After:
let counter = delegate(Counter())   # create delegate context
counter <- increment()               # invoke on context
```

#### Method calls in context

```orthon
list.append(item)              # Immediate — standard method call (no context)

ctx = delegate(lst)
ctx <- list.append(item)       # Delegated — method call on delegate context
ctx <- item                    # standalone message to context
```

---

### Colourless Functions

A key consequence: **functions have no colour.** There is no `async fn`
vs `fn`. A function is just a computation. The caller creates the
appropriate context and invokes with the matching operator.

```orthon
fun fetch(url: Url) -> String
    return httpClient.get(url)

# All valid, same function:
let a = fetch(url)                          # Immediate — blocks

let d = defer()
d <~ fetch(url)                             # Deferred
let b = await(d)

let g = delegate()
g <- fetch(url)                             # Delegated
let c = await(g)

let p = parallel()
p <= fetch(url)                             # Parallel
let d = await(p)

let r = remote("https://worker.node")
r <@ fetch(url)                             # Remote
let e = await(r)
```

---

### What Remains Separate

**Creating an execution context** is distinct from **invoking with a policy**:

| Operation | Construct | Notes |
|-----------|-----------|-------|
| Create deferred | `defer(obj?)` | Creates coroutine context |
| Create delegate | `delegate(obj)` | Allocates mailbox, starts processor |
| Create parallel pool | `parallel()` | Creates thread pool / scheduler |
| Create remote node | `remote(url)` | Establishes remote connection |
| Materialise result | `await(ctx)` | Blocks or yields until result ready |

The hypothesis separates:
1. **Context construction** — allocating a mailbox, thread pool, coroutine
2. **Invocation operator** — `<OP` selects how the call enters the context
3. **Result materialisation** — `await(...)` reads the result from the context

---

### Naming Note: `defer` vs `async`

The constructor is named `defer(...)`, not `async(...)`. Rationale:

- `async` carries heavy baggage from other languages (coloured functions,
  syntax viruses, ecosystem splits).
- `defer` is fresh — it means "defer execution to a context" with no
  preconceptions about how the context works internally.
- The context may use coroutines, fibers, green threads, or OS threads —
  the constructor name does not commit to an implementation model.

---

## Implications for Orthon

### Syntax

- `fn(args)` is Immediate by default — no context, no operator.
- Contextual invocation is always `ctx <OP method(args)`.
- `await(ctx)` is a regular function, not a keyword.
- `obj.method(args)` — remains natural, unchanged.
- `ctx <- obj.method(args)` — contextual method call, natural.

### Primitive Set

If accepted, `call` (PRIMITIVE_BLOCKS.md §3.2.3) must be revisited:

- `call` is no longer "invocation of a declared function" — it is
  **Invocation**, parameterised by Execution Context + Policy.
- The primitive may decompose into:
  - **Invocation** — the operation of calling a target with arguments
  - **Execution Context** — the environment that executes the call
  - **Execution Policy** — the how of invocation, selected by `<OP`

### Context Constructors

Each context constructor is a distinct operation. Some may be language
keywords; others may be StdLib functions. The line is drawn by whether
the constructor requires compiler-level semantics:

| Constructor | Likely classification | Reason |
|-------------|----------------------|--------|
| `defer(obj)` | Language | Coroutine state machine requires compiler support |
| `delegate(obj)` | Language | Isolation guarantees, mailbox semantics |
| `parallel()` | StdLib | Thread pool is an implementation detail |
| `remote(url)` | StdLib | Network protocol is an implementation detail |
| `await(ctx)` | StdLib | Blocking/yielding behaviour depends on context |

### Current Documents Affected

| Document | Change |
|----------|--------|
| [`PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) | `call` primitive revisited — becomes Invocation + Context + Policy |
| [`DELEGATE.md`](DELEGATE.md) | Rewrite — `delegate` becomes context constructor, `<-` is the operator |
| [`CONCURRENCY_MODEL.md`] | Absorbed into policy table |
| [`EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) | Substantial update — define execution contexts, policies, `await()` |
| [`SYNTAX.md`](../../what/SYNTAX.md) | Update invocation syntax section |
| [`GLOSSARY.md`](../../what/GLOSSARY.md) | Update `Delegate`, add `Invocation`, `Execution Context`, `Execution Policy`, remove `Spawn`, `Async` |
| [`CORE_CONCEPTS.md`](../../what/CORE_CONCEPTS.md) | Update CONCURRENCY_MODEL entry |
| [`DESIGN_PRINCIPLES.md`](../../how/DESIGN_PRINCIPLES.md) | Update Uniformity section |

---

## Open Questions

### 1. What does `defer(...)` wrap?

```orthon
let ctx = defer()              # bare context — invoke on nothing
ctx <~ fetch(url)

let ctx = defer(obj)           # context wrapping an object
ctx <~ obj.method(params)
```

Does `defer()` always require an initial value, or can it be empty and
receive invocations dynamically? What does the context *own* — just the
execution state (stack, program counter) or also domain state?

**Possible answer:** `defer()` creates a coroutine context with no state.
`defer(obj)` creates a coroutine context that owns `obj` — all invocations
on that context share `obj`'s state, serialised through the coroutine.
This mirrors `delegate(obj)` but with cooperative scheduling instead of
mailbox-based preemption.

### 2. Return type of contextual invocation

```orthon
ctx <~ fn(args)       # returns Future<T>?
ctx <- fn(args)       # returns Future<T>?
ctx <= fn(args)       # returns Task<T>?
```

Does every contextual invocation return a future/task, or does the
operator determine the return type? If `ctx <- fn(args)` returns
`Future<T>`, then `await(ctx)` must know which invocation to wait for.

**Possible answer:** Each contextual invocation returns a handle
(`Future<T>`), and `await(handle)` materialises that specific result.
`await(ctx)` without a handle waits for all pending invocations on
the context. This distinguishes "wait for this specific call" from
"drain the context."

### 3. Context creation vs context binding

```orthon
let ctx = delegate(Counter())   # creates context + binds object
ctx <- increment()               # invoke on the bound object
```

What about creating a context and binding an object later?

```orthon
let ctx = delegate()             # create empty delegate context
ctx <- Counter()                 # initialise the context with an object
ctx <- increment()               # now invoke on it
```

Is this allowed? Or must the context always be created with its owner?

### 4. `await(...)` semantics

Is `await(ctx)` blocking or non-blocking?

- In an Immediate context: blocks the calling thread.
- Inside a deferred context (`<~`): yields to the coroutine scheduler.
- Inside a delegated context (`<-`): the delegate is single-threaded,
  so `await` would process the next mailbox message.

The behaviour of `await` depends on **where** it is called, not just
**what** context it reads from. This is the ambient policy question.

### 5. Composition with ownership

DELEGATE.md establishes that `delegate` applies to **state owners**,
not arbitrary code. The `<-` policy inherits this — it serialises
access to the owner's state. But `<=` (parallel) applies to any
stateless function. How does the type system distinguish?

**Possible answer:** The function's type signature reveals whether it
captures mutable state. `<=` on a state-capturing closure without
delegate context is a compile error.

### 6. Ambient policy inside contexts

If a deferred function calls `fn(args)` (Immediate) inside its body,
does that block the deferred context or yield?

**Possible answer:** The execution policy is ambient. Inside a
delegated context, `fn(args)` (Immediate) means "execute in this
context synchronously" — it processes the call inline without
blocking the caller's thread. Inside a deferred context, `fn(args)`
means "eagerly evaluate within the current coroutine." The unmarked
`fn(args)` is policy-aware, not context-independent.

---

## Evaluation

### Strengths

- **Orthogonal.** Function (what) and policy (how) are independent axes.
  Any policy composes with any function.
- **Minimal.** One concept (Invocation) + context constructors replace
  `call` + `delegate` + `spawn` + `async`/`await`. Policies are
  operators, not new language concepts.
- **Explicit.** The execution policy is visible at every call site.
  `ctx <- fn()` vs `ctx <= fn()` — the difference is syntactic and
  obvious.
- **Extensible.** New policies (`<#` for GPU, `<@` for remote) can be
  added as new context constructors + operators.
- **Colourless functions.** No sync/async bifurcation. Any function
  can be invoked with any policy.
- **Natural base case.** `fn(args)` is Immediate by default — no
  operator clutter on the most common invocation form. `obj.method(args)`
  remains completely unchanged.
- **Context = tool metaphor.** Create the tool, then use it. Matches
  how programmers think about resources.

### Open Risks

- **Context boilerplate.** Creating a context before every non-immediate
  call adds verbosity. May need sugar for one-shot contexts.
- **`await(...)` as StdLib.** If `await` is not a keyword, can it be
  implemented efficiently? Does the compiler need intrinsic support?
- **Context lifecycle.** When is a context destroyed? `defer()`,
  `delegate()`, `parallel()` create resources — who owns them and
  when are they cleaned up?
- **Policy proliferation.** With extensible policies, could every
  library define its own `ctx <OP fn()` operator, reducing consistency?

---

## Decision History

| Date | Event |
|------|-------|
| 2026-07-29 | Hypothesis formulated: `fn<OP(args)` syntax with `<` between name and args |
| 2026-07-29 | Refined: `<` between name and args feels foreign. Model changed to prefix `~fn(args)` for deferred + binary `ctx <OP fn(args)` for other policies. |
| 2026-07-29 | Refined: `~` prefix removed. All non-immediate policies require an **explicit context** created by a constructor (`defer`, `delegate`, `parallel`, `remote`). Operator `<OP` works on context. `await(ctx)` is a regular function. |

**Status:** Exploratory — not accepted. Requires resolution of open
questions before EDR.
