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

There is only **Invocation**. The operator determines the **Execution Policy**.

```
fn<.(url)   — Immediate          — выполнить сейчас
fn<~(url)   — Deferred           — отложить (замыкание)
fn<-(url)   — Delegated          — делегировать владельцу (требуется delegate)
fn<=(url)   — Parallel           — независимое исполнение (требуется parallel)
fn<@(url)   — Remote             — удалённое исполнение (требуется remote)
```

Invocation becomes **two-dimensional**:

```
Что вызвать?    →  функция
Как выполнить?  →  оператор политики
```

The function answers only the first question. The operator answers only
the second.

---

### Syntax Pattern

```
target < OPERATOR (args)
```

Where:
- `<` is the invocation marker — visually reads as "send call to target"
- `OPERATOR` selects the execution policy
- `(args)` is the argument container

| Operator | Policy | Meaning | Requires |
|----------|--------|---------|----------|
| `.` | Immediate | Execute now, produce value | Nothing |
| `~` | Deferred | Delay execution, produce closure/callable | Nothing — closure construction |
| `-` | Delegated | Delegate execution to owner | `delegate` context (mailbox, actor) |
| `=` | Parallel | Execute independently | Parallel runtime (thread pool, scheduler) |
| `@` | Remote | Execute on remote node | Remote execution infrastructure |

Full forms:

```orthon
fetch<.(url)      # Immediate — call fetch with url now
fetch<~(url)      # Deferred — produce closure: () => fetch(url)
owner<-fetch(url) # Delegated — execute fetch on owner's context
fetch<=(url)      # Parallel — execute fetch independently
fetch<@(url)      # Remote — execute fetch on remote node
```

---

### What This Replaces

#### `async` / `await` — eliminated

`async` is not a modifier on the function. The function is colourless.
`<~` (Deferred) replaces `async fn` — it produces a `Future<T>` (or
equivalent) at the call site without modifying the function declaration.

`await` is not needed because materialising a deferred result is just
reading the value from the future:

```orthon
let future = fetch<~(url)   # Deferred — Future<String>
let data = future.()         # Materialise — block/await until ready
```

Materialisation is a separate operation on `Future<T>`, not a language
keyword. The execution context (immediate, delegated, parallel) determines
whether materialisation blocks or yields.

#### `spawn` — eliminated

```orthon
# Before:
let t1 = spawn async loadImage("a.jpg")
let t2 = spawn async loadImage("b.jpg")

# After:
let t1 = loadImage<=(image("a.jpg"))
let t2 = loadImage<=(image("b.jpg"))
```

#### `delegate send` — unified

```orthon
# Before:
let counter = delegate(Counter(0))
counter <- increment()

# After:
let counter = Counter()      # ordinary object
# ... create delegate context for counter ...
counter<-increment()         # delegated invocation
```

Or, with method syntax:

```orthon
counter.increment<-(())      # method .increment, policy <-, no args
counter.append<-(item)       # method .append, policy <-, item as arg
```

---

### Colourless Functions

A key consequence: **functions have no colour.** There is no `async fn`
vs `fn`. A function is just a computation. The caller decides whether
it runs immediately, is deferred, delegated, or parallelised.

```orthon
fun fetch(url: Url) -> String
    # Contains I/O, but the function doesn't know or care
    # about the execution policy — that's the caller's choice
    return httpClient.get(url)

# All valid, same function:
let a = fetch<.(url)          # Immediate — blocks
let b = fetch<~(url)          # Deferred — produces Future
let c = owner<-fetch(url)     # Delegated — on owner's context
let d = fetch<=(url)          # Parallel — on thread pool
let e = fetch<@(url)          # Remote — on remote node
```

This eliminates the sync/async bifurcation that fragments language
ecosystems (the "function colouring" problem).

---

### What Remains Separate

**Creating an execution context** is distinct from **invoking with a policy**:

| Operation | Current | Hypothesis |
|-----------|---------|------------|
| Create delegate | `delegate(Counter())` | Still needed — `counter<-(...)` requires a context |
| Create parallel runtime | Implicit | Still needed — `fn<=(x)` requires a scheduler |
| Materialise future | `await future` | `future.()` or `+future` |

The hypothesis separates:
1. **Context construction** — allocating a mailbox, thread pool, remote connection
2. **Invocation policy** — selecting which context to use at the call site

Context construction is an implementation concern; invocation policy is
a language syntax concern.

---

## Implications for Orthon

### Syntax

- `()` ceases to be the call operator. `()` is the argument container.
- Invocation is always `target < OPERATOR (args)`.
- `fn(x)` becomes either `fn<.(x)` (explicit immediate) or remains as
  syntactic sugar for `fn<.(x)`.

### Primitive Set

If accepted, `call` (PRIMITIVE_BLOCKS.md §3.2.3) must be revisited:

- `call` is no longer "invocation of a declared function" — it is
  **Invocation**, parameterised by Execution Policy.
- The primitive may split into:
  - **Invocation** — the operation of calling a target with arguments
  - **Execution Policy** — the how of invocation, selected by operator

### `delegate` keyword

The current `delegate` keyword serves two roles:
1. **Creating** a delegated execution context: `delegate(Counter())`
2. **Calling** into it: `counter <- msg(args)`

Under the hypothesis, role 2 becomes the `<-` policy operator. Role 1
(creating the context) still needs a construct — possibly a renamed
keyword or a policy constructor.

### Current Documents Affected

| Document | Change |
|----------|--------|
| [`PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) | `call` primitive revisited — becomes Invocation + Execution Policy |
| [`DELEGATE.md`](DELEGATE.md) | Rewrite — `delegate` becomes context constructor, not call mechanism |
| [`CONCURRENCY_MODEL.md`] | Absorbed into policy table |
| [`EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) | Substantial update — define execution policies and their semantics |
| [`SYNTAX.md`](../../what/SYNTAX.md) | Update invocation syntax section |
| [`GLOSSARY.md`](../../what/GLOSSARY.md) | Update `Delegate`, add `Invocation`, `Execution Policy`, remove `Spawn`, `Async` |
| [`CORE_CONCEPTS.md`](../../what/CORE_CONCEPTS.md) | Update CONCURRENCY_MODEL entry |
| [`DESIGN_PRINCIPLES.md`](../../how/DESIGN_PRINCIPLES.md) | Update Uniformity section — all invocations use `<OP` syntax |

---

## Open Questions

### 1. Attribute access vs Immediate

Currently `.` is attribute access (`obj.field`). If `.` is also an
execution policy marker inside `fn<.(x)`, there's no ambiguity — `<.`
is an atomic operator. But what about `obj.method<.(x)` — is that
attribute access + invocation, or a single operation?

**Possible answer:** `obj.method` is attribute access (gets the method
value), then `<.(x)` invokes it. Two separate operations composed
sequentially, consistent with the current model.

### 2. Materialisation syntax

If `await` is eliminated, how does the programmer materialise a future?

Options:
- `future.()` — read with implicit block (consistent with `.` immediate)
- `+future` — unary prefix operator
- Implicit coercion when `Future<T>` is used in a position expecting `T`

### 3. Owner context for `<-`

How is the owner (delegate context) created and associated?

```orthon
let counter = Counter()
let dc = delegate(counter)    # create delegate context
dc<-increment()                # invoke on delegate context

# Or:
let counter = delegate(Counter())
counter<-increment()
```

Is `delegate(...)` still the constructor, or is there a policy
constructor like `counter := Counter() <-`?

### 4. What is `()` without `<OP`?

If `fn(x)` is sugar for `fn<.(x)`, the language may support both.
But if bare `()` is banned outside `<OP`, all existing call syntax
changes. Migration cost vs. clarity gain?

### 5. Composition with ownership

DELEGATE.md establishes that `delegate` applies to **state owners**,
not arbitrary code. The `<-` policy inherits this — it serialises
access to the owner's state. But `<=` (parallel) applies to any
stateless or owned function. How does the type system distinguish?

**Possible answer:** The function's type signature reveals whether it
captures mutable state. `<=` on a state-capturing closure without
delegate context is a compile error.

### 6. What about `await` inside function bodies?

If a function calls `fetch<.(url)` (immediate, blocks) inside its body,
and the function itself is running in a delegated context, does it
block the entire delegate or just yield?

**Possible answer:** The execution policy is ambient — the function
inherits the policy of its caller. `fetch<.(url)` inside a delegated
context yields rather than blocks, because `<.` means "materialise now"
which, in a delegated context, is "yield until result ready."

---

## Evaluation

### Strengths

- **Orthogonal.** Function (what) and policy (how) are independent axes.
  Any policy composes with any function.
- **Minimal.** One concept (Invocation) replaces `call` + `delegate` +
  `spawn` + `async`/`await`. Policies are a table, not new language
  constructs.
- **Explicit.** The execution policy is visible at every call site.
  `fetch<.()` vs `fetch<~()` vs `fetch<=()` — the difference is
  syntactic and obvious.
- **Extensible.** New policies (`<#` for GPU, `<@` for remote) can be
  added without changing the language — they're new operators, not new
  concepts.
- **Colourless functions.** No sync/async bifurcation. Any function
  can be invoked with any policy.

### Open Risks

- **Syntax verbosity.** `fn<.(x)` is more characters than `fn(x)`.
  The visual weight of `<` on every invocation may be fatiguing.
- **Bare `()` ambiguity.** If `fn(x)` is still valid as sugar, when
  do programmers use it vs `fn<.(x)`? Two ways to say the same thing.
- **Materialisation ergonomics.** Without `await`, every future read
  needs an explicit operation. This may be verbose in deeply async
  code unless good sugar exists.
- **Delegated context creation.** The hypothesis is clean about
  *invocation* but doesn't fully resolve how delegated contexts are
  *created* — this is where the complexity migrates.

---

## Decision History

- **2026-07-29:** Hypothesis formulated during design exploration.
  This document created to capture the model, implications, and open
  questions.
- **Status:** Exploratory — not accepted. Requires resolution of open
  questions before EDR.
