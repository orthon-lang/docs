# Execution Context Invocation

> **⚠️ DRAFT — Concept research.**
> This document proposes a unified invocation model: a single **Invocation**
> operation submitted to an **Execution Context** via a two-operator family.
> The operator encodes the ownership relationship (single owner vs.
> distribution); the context constructor determines the execution policy.
>
> **Status:** Exploratory — not accepted.
> **Supersedes:** [`EXECUTION_POLICY_HYPOTHESIS.md`](EXECUTION_POLICY_HYPOTHESIS.md) (earlier hypothesis with per-context operators)
> **Related:** [`DELEGATE.md`](DELEGATE.md) (current execution policy model),
> [`FUNCTIONS.md`](FUNCTIONS.md),
> [`CONCURRENCY_MODEL.md`](CONCURRENCY_MODEL.md) (current concurrency model, superseded if this concept accepted),
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

**Proposal:** All three are the same operation — **Invocation** — submitted to
an **Execution Context** via a two-operator family (`<-` for single owner,
distribution for stateless workers). The operator encodes the ownership
relationship; the context determines the execution policy.

---

## Model

### Core Claim

There is no:
- function call
- delegate send
- parallel spawn
- async / await
- coroutine

There is only **Invocation**. A context constructor creates an execution context.
A two-operator family submits invocations to that context. The operator encodes
the ownership relationship (single owner vs. distribution); the context
determines how they execute.

```orthon
fn(args)                   — Immediate — execute now (no context)

ctx = defer(obj)           — coroutine context
ctx <- method(params)      — submit to coroutine

ctx = delegate(obj)        — mailbox context
ctx <- method(params)      — submit to mailbox

ctx = spawn()              — thread context (no owner)
ctx |> method(params)      — distribute to thread pool

ctx = fork()               — process context (no owner)
ctx |> method(params)      — distribute to process pool
```

Invocation becomes **two-dimensional**:

```
What to invoke?  →  function / method
How to execute?  →  context (created by constructor, not encoded in operator)
```

The function answers only the first question. The context answers only the second.
The operator family does one thing: **submit invocation to context** — the
member operator encodes the ownership relationship (`<-` for single owner,
distribution for stateless workers).

> The distribution operator's concrete glyph is a Phase 5 decision. Examples in
> this document use `|>` provisionally; `<||` is the leading candidate (see OQ5
> Resolution).

---

### Why One Operator, Not Many

> **Superseded (2026-07-31, OQ5 Resolution).** The model moved from a single
> operator to a **two-operator family** grouped by ownership relationship
> (`<-` = single owner; distribution = stateless workers). The rejection of
> per-context-policy operators stands — the two operators are members of one
> Submission family, not one-operator-per-policy. The historical rationale
> below is retained for traceability.

An earlier version of this proposal (see [`EXECUTION_POLICY_HYPOTHESIS.md`](EXECUTION_POLICY_HYPOTHESIS.md))
proposed distinct operators per context type (`<~`, `<-`, `<=`, `<@`, `<#`). This was
rejected in favour of a single `<-`. Rationale:

**Argument for distinct operators:** Each operator visually encodes the execution
semantics at the call site. `ctx <~ fn()` signals "coroutine yield" to the reader;
`ctx <= fn()` signals "parallel dispatch." The reader does not need to know the
type of `ctx` to understand what happens to the caller.

**Argument for a single operator:** The context already knows its policy.
`delegate(Counter)` is a different type from `spawn()`. The operator's only
job is to transfer execution to the context — what the context does with it is the
context's responsibility. This is more orthogonal: the *what* (invocation) is
separated from the *how* (context), and the syntax does not duplicate information
already present in the type system.

**Decision (superseded):** A single operator `<-` was adopted on 2026-07-30. The
OQ5 Resolution (2026-07-31) superseded it with a two-operator family: `<-` (single
owner) and the distribution operator (stateless workers). The context constructor
still determines the policy; the operator now also encodes the ownership
relationship. If the reader needs to understand what `ctx <- fn()` does, they look
at how `ctx` was constructed — typically on the line immediately above.

**Extensibility benefit (preserved):** A new context type (GPU, cluster, quantum)
requires only a new constructor — no new operator. The new context resolves to one
of the two operators by whether it owns state. The syntax surface does not grow.

---

### Why Not `.` (Dot)

The dot operator `.` was considered as a candidate for contextual invocation:

```orthon
ctx.append(item)    # Could mean "submit append to context"
```

This was rejected because `.` already means **immediate method call** in Orthon:

```orthon
list.append(item)       # Immediate. Blocks. Executes now.
ctx.append(item)        # Would this block? Or submit to mailbox?
```

Using the same syntax for two fundamentally different operations (immediate call
vs. deferred submission) violates the Principle of Least Astonishment and breaks
orthogonality. The reader cannot tell from syntax alone whether `x.foo()` blocks
or not — it depends on the type of `x`. This is loss of local reasoning.

`<-` explicitly separates the two intents:

```orthon
list.append(item)       # . → Immediate. Direct call.
ctx <- append(item)     # <- → Submit to context. Not a direct call.
```

---

### Why Not a Symmetric Extraction Operator

A symmetric operator for extracting results from a context was considered:

```orthon
ctx -> lst          # Rejected: -> already used for function return types
lst <- ctx          # Rejected: <- would have two meanings depending on operand types
```

**`->` conflict:** Orthon already uses `->` in function signatures (`fun name(args) -> Type`)
and return expressions (`return value ->`). Overloading it for context extraction
would violate orthogonality.

**Reverse `<-` ambiguity:** `ctx <- expr` means "submit expr to ctx." Reversing to
`var <- ctx` would mean "extract from ctx into var." The same operator would have
opposite dataflow directions depending on operand types — also loss of local reasoning.

**Decision:** Extraction uses named functions, not operators:

```orthon
result = await(ctx)      # Defer — yield scheduler, resume when ready
owner = take(ctx)        # Delegate — extract owner, terminate context
```

`await` and `take` are context-specific named functions (with possible compiler
intrinsic support). Each context type defines its own extraction vocabulary.
For `spawn`/`fork`, extraction uses the generator protocol (`next()`/`stop()`).
`grab`/`gather` are StdLib sugar over the generator.

---

### Key Insight: Context Required

Only **Immediate** execution is available without a context. All other
execution policies require an **explicit execution context**.

The programmer must create the context first:

```orthon
let ctx = defer(obj)       # create a deferred/coroutine context
ctx <- method(params)      # submit invocation to this context
```

This mirrors the natural pattern: you create a tool, then use it.
`defer(...)`, `delegate(...)`, `spawn()`, `fork()` are all
**context constructors** — they create an execution context with the
desired policy.

---

### Syntax Design Principle

> The invocation operator sits **between a context and a call**: `ctx <- fn(args)`.
> It connects the context to the action. This mirrors natural language:
> "context performs action."

| Position | Example | Feels |
|----------|---------|-------|
| Between name and args | `fn<OP(args)` | ❌ Foreign — breaks the natural name-args bond |
| Between context and call | `ctx <- fn(args)` | ✅ Natural — connects context and action |
| No operator needed | `fn(args)` | ✅ Natural — name and args together |

**Immediate** is the base case — no context, no operator. The standard
call syntax `fn(args)` is Immediate by default.

---

### Unified Syntax

| Form | Meaning |
|------|---------|
| `fn(args)` | Immediate — execute now, produce value |
| `ctx <- fn(args)` | Submit to single owner — serialised |
| `ctx |> fn(args)` | Distribute to stateless workers — parallel (glyph provisional, Phase 5) |

**Context constructors:**

| Constructor | Creates | Execution policy | Operator |
|-------------|---------|------------------|----------|
| `defer(obj)` | Coroutine context | Cooperative, suspending | `<-` |
| `delegate(obj)` | Mailbox context | Sequential, ownership-scoped | `<-` |
| `spawn()` | Thread context | Parallel, shared memory | distribution |
| `fork()` | Process context | Parallel, isolated memory | distribution |
| — | Immediate | Synchronous, blocking | — |

---

### What This Replaces

#### `async` / `await` — eliminated

`async` is not a modifier on the function. The function is colourless.
The caller picks a context: `defer(obj)`/`delegate(obj)` for owner state
(`<-`), `spawn()`/`fork()` for stateless work (distribution).

```orthon
# Before:
async fn fetch(url) -> String
    return await httpClient.get(url)
let data = await fetch(url)

# After:
fun fetch(url) -> String           # colourless function
    return httpClient.get(url)

let pool = spawn()                  # create a worker context
pool |> fetch(url)                  # distribute to workers
let data = grab(pool)               # materialise result
```

`await(...)` is a regular function that reads a result from any execution
context. It is not a keyword — it blocks or yields depending on the
context type and the ambient execution environment.

#### `spawn` / `fork` — unified

The old `spawn` keyword splits into two context constructors with different
memory models:

```orthon
# Threading (shared memory):
let pool = spawn()
pool |> loadImage("a.jpg")
pool |> loadImage("b.jpg")
let results = gather(pool)     # StdLib: next() in a loop

# Parallelism (isolated memory):
let cluster = fork()
cluster |> processChunk(data, 0..100)
cluster |> processChunk(data, 100..200)
let first = grab(cluster)       # StdLib: next() + stop()
```

Extraction for `spawn`/`fork` uses the **generator protocol**:
`next() -> Option<T>` (blocking, returns `none` when depleted)
and `stop()` (cancel pending, clean up resources).

#### `delegate send` — unified

```orthon
# Before:
let counter = delegate(Counter(0))
counter <- increment()

# After:
let counter = delegate(Counter())   # create delegate context
counter <- increment()              # submit to context
```

#### `delegate send` — unified

```orthon
# Before:
let counter = delegate(Counter(0))
counter <- increment()

# After:
let counter = delegate(Counter())   # create delegate context
counter <- increment()              # submit to context
let result = take(counter)          # extract owner, terminate context
```

#### Method calls in context

```orthon
list.append(item)              # Immediate — standard method call (no context)

ctx = delegate(lst)
ctx <- list.append(item)       # Submit — method call on delegate context
ctx <- item                    # Standalone message to context
```

#### `using` / context manager — unified

Resource management (Python's `with`, C#'s `using`, Java's
`try-with-resources`) is not a separate language feature. It is the
same pattern — **create a context, submit invocations, materialise
results** — with one additional property: the context's destructor
guarantees cleanup of the wrapped resource.

```orthon
# Before (Python-style):
with open("data.txt") as file:
    content = file.read()
    process(content)

# After — explicit context:
let ctx = delegate(open("data.txt"))
ctx <- file.read()
let content = await(ctx)
process(content)
# ctx destroyed at scope exit → file.close() called automatically

# After — using sugar (desugars to the above):
using file = open("data.txt")
    content = file.read()
    process(content)
```

The `using` construct desugars to:
1. Create an execution context wrapping the resource
2. Bind the resource to a name within the block scope
3. Submit invocations via ordinary `call` (immediate inside the context)
4. Destroy the context at scope exit, triggering resource cleanup

`using` introduces **no new semantics** — it is syntactic sugar for
the common pattern of "context wrapping a resource with a
deterministic destructor." The resource's cleanup is the context's
destructor, not a separate `close()` call the programmer must
remember.

**Why `resource(...)` is not a separate constructor.** An earlier
candidate considered adding a dedicated `resource(obj)` context
constructor for resource management. This was rejected: every
execution context (`delegate`, `defer`, `spawn`, `fork`)
already has a lifecycle — a constructor that wraps an object is
naturally a context whose destructor cleans up that object.
`delegate(open("data.txt"))` is already a resource-managing context;
no new constructor type is needed. The `using` sugar simply makes
this pattern more concise.

---

### Colourless Functions

A key consequence: **functions have no colour.** There is no `async fn`
vs `fn`. A function is just a computation. The caller chooses the context:
an operation tied to an owner's state goes to `delegate`/`defer` (`<-`); a
stateless computation goes to `spawn`/`fork` (distribution).

```orthon
fun fetch(url: Url) -> String
    return httpClient.get(url)

# Stateless computation — immediate or distribution:
let a = fetch(url)                          # Immediate — blocks

let pool = spawn()
pool |> fetch(url)                          # Thread
let first = fetch(pool.next())              # Generator: next()
pool.stop()

let cluster = fork()
cluster |> fetch(url)                       # Process
let all = gather(cluster)                   # StdLib: next() loop

# Operations tied to an owner — single-owner contexts:
let counter = delegate(Counter())
counter <- increment()                      # Delegated, serialised
let result = take(counter)
```

A function is not tied to a context kind in its declaration — it is
colourless. The context constructor (and its paired operator) determines
the execution policy. Because `spawn`/`fork` require `Send`/`Move`
captured data, a stateless function like `fetch` is trivially valid in
both distribution contexts.

---

### What Remains Separate

Three distinct operations, two operator glyphs (one family):

| Operation | Mechanism | Notes |
|-----------|-----------|-------|
| Create context | Constructor: `defer(obj)`, `delegate(obj)`, `spawn()`, `fork()` | Allocates resources, establishes policy |
| Submit invocation | Operators: `ctx <- call` (single owner), `ctx |> call` (distribution) | Two-operator family by ownership relationship |
| Materialise result | Context-specific: `await(ctx)`, `take(ctx)`, `next()/stop()` | Per-context vocabulary |

The model separates:
1. **Context construction** — allocating a coroutine, mailbox, thread pool, process pool
2. **Invocation submission** — the operator family transfers execution to the
   context; the member operator encodes the ownership relationship
3. **Result materialisation** — context-specific extraction (`await` for defer,
   `take` for delegate, `next()`/`stop()` generator for spawn/fork)

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

## Context Manager as Syntactic Sugar

The model above eliminates `async`/`await`, `spawn`, and `delegate send`
as separate features. The **context manager** (`with`/`using`) follows
the same trajectory: it is syntactic sugar over the Execution Context +
Scope pattern, not a separate language feature.

### The Pattern

Every resource that needs scoped cleanup follows the same three-step
structure:

```orthon
{
    let ctx = delegate(openResource())   # 1. create context wrapping resource
    ctx <- useResource()                  # 2. submit operations
    let result = await(ctx)               # 3. materialise final result
}                                         # ctx destroyed → resource cleaned up
```

The `using` sugar collapses this to:

```orthon
using resource = openResource()
    result = useResource()
```

### Desugaring Rules

| `using` form | Desugars to |
|---|---|
| `using x = expr` | `let __ctx = delegate(expr); __ctx <- ...; await(__ctx)` |
| `using x = expr` with `finally` | `let __ctx = delegate(expr); try ... finally { release(__ctx) }` |
| `using x = expr1, y = expr2` | Nesting: `using x = expr1 { using y = expr2 { ... } }` |

### Relationship to Ownership

`using` is consistent with Orthon's ownership model
([`SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md) § Ownership):

- The resource (file, socket, handle) is a **resource-typed value** —
  it has exclusive responsibility and cannot be silently duplicated.
- `using` transfers the resource into the context's ownership at
  construction. The context becomes the owner and is responsible for
  destruction.
- The bound name inside the `using` block is a **borrow** of the
  resource held by the context — the programmer can invoke operations
  on it, but the context retains ownership.
- At scope exit, the context's destructor runs, which closes/releases
  the resource. This is deterministic and guaranteed, regardless of
  how the block is exited (normal return, error, early break).

### Semantic Dimensions Involved

| Dimension | Role in `using` |
|-----------|-----------------|
| **Lifetime** | Primary — scope-bound destruction is the core guarantee |
| **Ownership** | Transfer of resource ownership to context; borrow inside block |
| **Evaluation** | Operations inside the block execute in the context's policy (immediate by default) |
| **Mutation** | `proc` operations on the resource are serialised through the context |

### Comparison with Alternatives

| Mechanism | How it works | Orthon equivalent |
|-----------|-------------|-------------------|
| Python `with` | Context manager protocol (`__enter__`/`__exit__`) | `using` desugars to context constructor + scope |
| Go `defer` | Statement-level, not scope-level | `using` is scope-level, deterministic |
| Rust RAII | Ownership-based, destructor on drop | Same principle: context destructor = resource cleanup |
| Java `try-with-resources` | Closeable interface, try-block syntax | `using` block with desugaring to context |

### Why Not RAII-only?

Orthon's ownership model already supports RAII: a resource-typed value
with a destructor is cleaned up when it goes out of scope. Why add
`using` at all?

Because **RAII only works when the resource has a single owner that
stays in scope**. Two patterns break pure RAII:

1. **Shared ownership** — a resource held by a delegate that outlives
   the creating scope: `let shared = delegate(resource)` — the context
   owns the resource, not the creating scope.
2. **Conditional cleanup** — a resource may or may not outlive its
   creating scope depending on a runtime path (lend to a parallel
   worker vs. close immediately).

`using` provides an explicit, scope-anchored guarantee that is
independent of ownership topology. It is the **declarative form** of
RAII: "this resource is scoped to this block."

### Relationship to `SCOPED_RESOURCE_LIFECYCLE.md`

The earlier research in
[`SCOPED_RESOURCE_LIFECYCLE.md`](../essential/SCOPED_RESOURCE_LIFECYCLE.md)
posed three open questions. The Execution Context model answers them:

| Question | Answer |
|----------|--------|
| RAII or explicit scope blocks? | Both. RAII is the semantic foundation (ownership + destructor). `using` is the explicit scope block — syntactic sugar over the context pattern. They compose: use RAII for single-owner resources, `using` for context-wrapped resources. |
| Resources that outlive their scope? | Move the context: `let shared = delegate(file); return shared` — the context (and its resource) outlives the creating scope. The context is the owner; its lifecycle is decoupled from the creating scope. |
| Built-in or library? | The context constructors (`delegate`, `defer`) are language-level because they require compiler support. `using` is a syntax-level desugaring — it could be language or library sugar (Phase 5 decision). The key semantic commitment (context destructor → resource cleanup) is language-level because it ties into the Lifetime dimension's scope-based destruction guarantee. |

---

## Implications for Orthon

### Syntax

- `fn(args)` is Immediate by default — no context, no operator.
- Contextual invocation uses the operator family: `ctx <- call` (single
  owner) or `ctx |> call` (distribution, glyph provisional).
- `await(ctx)` is a regular function, not a keyword.
- `obj.method(args)` — remains natural, unchanged.
- `ctx <- obj.method(args)` — contextual method call, natural.
- `using x = expr` — desugars to context construction + scope-bound destructor.

### Primitive Set

If accepted, `call` (PRIMITIVE_BLOCKS.md §3.2.3) must be revisited:

- `call` is no longer "invocation of a declared function" — it is
  **Invocation**, parameterised by Execution Context.
- The primitive decomposes into:
  - **Invocation** — the operation of calling a target with arguments
  - **Execution Context** — the environment that executes the call
  - **Submit operator** — `<-` transfers invocation to context

### Context Constructors

Each context constructor is a distinct operation. Some may be language
keywords; others may be StdLib functions. The line is drawn by whether
the constructor requires compiler-level semantics:

| Constructor | Likely classification | Reason |
|-------------|----------------------|--------|
| `defer(obj)` | Language | Coroutine state machine requires compiler support |
| `delegate(obj)` | Language | Isolation guarantees, mailbox semantics |
| `spawn()` | Language | Thread semantics, cancellation require runtime support |
| `fork()` | Language | Process isolation requires OS-level support |
| `using` | Language (sugar) | Syntactic sugar over `delegate(obj)` + scope + destructor |
| `await(ctx)` | StdLib | Yield/resume behaviour depends on context |
| `next()`/`stop()` | Language | Generator protocol on spawn/fork contexts |

### Current Documents Affected

| Document | Change |
|----------|--------|
| [`PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) | `call` primitive revisited — becomes Invocation + Context; add generator protocol |
| [`DELEGATE.md`](DELEGATE.md) | Rewrite — `delegate` becomes context constructor, `<-` is the submit operator, `take()` extraction |
| [`CONCURRENCY_MODEL.md`](../important/CONCURRENCY_MODEL.md) | Absorbed into context model — replaced by spawn/fork contexts |
| [`EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) | Substantial update — define execution contexts, `await()`, `take()`, generator `next()`/`stop()`, resource lifecycle via context destructor |
| [`SYNTAX.md`](../../what/SYNTAX.md) | Update invocation syntax section — two-operator family (`<-` single owner, distribution glyph TBD); add `using` sugar; generator syntax (deferred to Phase 5) |
| [`GLOSSARY.md`](../../what/GLOSSARY.md) | Update `Delegate`, `Spawn`; add `Invocation`, `Execution Context`, `using`, `take`, generator protocol; remove `Async`, `parallel` |
| [`CORE_CONCEPTS.md`](../../what/CORE_CONCEPTS.md) | Update CONCURRENCY_MODEL entry |
| [`DESIGN_PRINCIPLES.md`](../../how/DESIGN_PRINCIPLES.md) | Update Uniformity section |
| [`SCOPED_RESOURCE_LIFECYCLE.md`](../essential/SCOPED_RESOURCE_LIFECYCLE.md) | Superseded — absorbed into context model. `using` is syntactic sugar over context + scope, not a separate mechanism. |
| [`DECLARATIVE_CONSTRUCTS.md`](../important/DECLARATIVE_CONSTRUCTS.md) | Update § Resource Management — replace standalone `using` analysis with desugaring to Execution Context pattern. |

---

## Open Questions

### 1. What does `defer(...)` wrap? — **RESOLVED (see OQ5)**

> Bare `defer()` does not exist: like `delegate`, `defer` always wraps an
> object. The speculative body below is retained for traceability.

```orthon
let ctx = defer()              # bare context — invoke on nothing
ctx <- fetch(url)

let ctx = defer(obj)           # context wrapping an object
ctx <- obj.method(params)
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
ctx <- fn(args)       # returns Future<T>? or void?
```

Does every contextual invocation return a handle, or is the return
discarded and `await(ctx)` drains all pending results?

**Possible answer:** Each contextual invocation returns a handle
(`Future<T>`), and `await(handle)` materialises that specific result.
`await(ctx)` without a handle waits for all pending invocations on
the context. This distinguishes "wait for this specific call" from
"drain the context."

### 3. Context creation vs context binding — **RESOLVED (see OQ5)**

> A context is always created with its owner; late binding
> (`let ctx = delegate()` then `ctx <- Counter()`) does not exist.
> The speculative body below is retained for traceability.

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

### 4. `await(...)` semantics — **RESOLVED**

Each context type uses its own extraction vocabulary, eliminating the
ambient policy problem.

### OQ4 Resolution — Context-Specific Extraction Vocabulary

The following matrix was adopted on 2026-07-30 through the Concept
Design Review (Convergence Check) process.

| # | Lens | Constructor | Extraction | Underlying model |
|---|------|-------------|------------|------------------|
| 1 | **Direct** | — | `fn(args)` | Immediate call |
| 2 | **Async** | `defer(obj)` | `await(ctx)` | Coroutine (cooperative, suspending) |
| 3 | **Delegation** | `delegate(obj)` | `take(ctx)` | Actor (ownership transfer, mailbox) |
| 4 | **Threading** | `spawn()` | `next()` / `stop()` | Thread (shared memory, parallel) |
| 5 | **Parallelism** | `fork()` | `next()` / `stop()` | Process (isolated memory, parallel) |

**Key decisions:**

1. **`defer`** — constructor stays. `async` rejected (baggage, coloured functions).
2. **`await`** — only for `defer`. Each context uses its own extraction word.
3. **`yield`** — removed from concept. Runtime yields on `await` automatically.
4. **`take`** — replaces `return` for delegate. Owner extraction + context termination.
5. **`spawn` vs `fork`** — separate contexts. `spawn` = threads (shared memory),
   `fork` = processes (isolated memory). Not merged into `delegate`.
6. **Generator protocol** — primary extraction for `spawn`/`fork`:
   `next() -> Option[T]` (blocks until next result, `none` when depleted),
   `stop()` (cancel pending, clean up resources).
7. **`grab`/`gather`** — StdLib sugar over the generator, not language
   primitives. `grab` = `next()` + `stop()`, `gather` = `next()` loop.
8. **`peek`** — deferred to v0.2 (non-blocking check without extraction).
9. **Generator syntax** (e.g., `for-in` loop, `while(pool)`, expression form
   `f(x) for x in pool`) — deferred to Phase 5 (Syntax Design).

### 5. Composition with ownership — **RESOLVED**

### OQ5 Resolution — Two-Operator Submission Family

Adopted on 2026-07-31 through the Concept Design Review (Convergence
Check) process. The single-operator model (`<-` for all contexts) is
superseded by a **two-operator family** that partitions contexts by
their **ownership relationship**. The operator encodes the ownership
relationship; the context constructor still determines the execution
policy.

| Operator | Relationship | Contexts | Type-system check |
|----------|--------------|----------|-------------------|
| `<-` | Submit to the context's **single owned state** — serialised, in order | `delegate(obj)`, `defer(obj)` | Compatible with the owner's type; may capture/mutate owner state |
| distribution | **Distribute** to stateless workers — independent, parallel | `spawn()`, `fork()` | Captured data must satisfy `Send` (`spawn`) or `Move` (`fork`) |

**Key decisions:**

1. **Two operators, not one.** `<-` is reserved for contexts that own a
   single state object. A second operator (distribution) is used for
   contexts that dispatch to stateless workers. The operator set is
   **closed (2)**; the constructor set is **open** — a new context type
   resolves to one of the two by whether it owns state.
2. **`<-` = single owner.** `delegate(obj)` and `defer(obj)` both wrap
   and own exactly one object; invocations are serialised against it.
   Both **always require an object** — bare contexts (`delegate()`,
   `defer()`) do not exist. This consequently resolves OQ1 and OQ3.
3. **Distribution = stateless workers.** `spawn()` (threads, shared
   memory) and `fork()` (processes, isolated memory) dispatch
   invocations to independent workers. Functions submitted to these
   contexts must not depend on a single owner's mutable state.
4. **`Send` / `Move` marker traits (Variant B).** The type system checks
   captured data at compile time, Rust-style auto-traits: `spawn`
   requires all captured values be `Send` (safe to transfer across
   threads); `fork` requires `Move` = `Send` + serialisable (transferable
   across process boundaries). Stateless functions are trivially `Send`
   and `Move`. Errors surface at compile time, not runtime.
5. **Binary axis.** Submission is one of exactly two relationships:
   serialised-to-owner or distributed-to-workers. Candidate third
   contexts (lock/shared-state, channel, reduce/aggregate, actor pool,
   reactive push) each resolve to one of the two — respectively as `<-`,
   as a data structure, as extraction, as composition, or as a
   materialisation vocabulary — so no third operator is required.
6. **`<=` rejected as the distribution glyph.** `<=` is already Orthon's
   less-than-or-equal comparison (e.g. `requires v >= 0 && v <= 150`),
   violating `SYNTAX.md`'s "one symbol → one meaning". The family-
   coherent candidate `<||` (shared `<` prefix = "submit into context";
   suffix encodes the relationship) is the leading alternative; `|>` is
   the maximally-distinct alternative. The concrete glyph is deferred
   to Phase 5 (Syntax Design); examples in this document use `|>`
   provisionally.

### 6. Ambient policy inside contexts — **RESOLVED**

If a function submitted to a deferred context calls `fn(args)` (Immediate)
inside its body, does that block the deferred context or yield?

The execution policy is **not** ambient. The unmarked call `fn(args)`
is context-independent with fixed semantics.

### OQ6 Resolution — Fixed Inline Call Semantics

Adopted on 2026-07-31 through the Concept Design Review (Convergence
Check) process.

**Answer to the question:** `fn(args)` inside a deferred context
neither blocks the context nor yields — it executes synchronously
inline, in the current execution environment. It suspends only if the
callee itself hits an explicit extraction point (`await`, `take`,
`next`).

**Key decisions:**

1. **Fixed call semantics.** `fn(args)` is context-independent: always
   a synchronous, inline call. A context affects only the
   *capabilities* available inside it (which extraction operations,
   which state is reachable), never the semantics of the call itself.
   This preserves local reasoning and orthogonality.
2. **No self-delegation.** There is no `this <- fn(...)` / `yield
   from` mechanism. Submitting an invocation to one's own context is
   not a language feature.
3. **Long computations go to another context.** The uniform idiom for
   work that may run long inside a context is cross-context submission
   to a stateless-worker context: `pool |> fn(...)` (distribution
   operator, OQ5), materialised via the generator protocol
   (`next()`/`stop()`, or StdLib `grab`/`gather`, OQ4). No self-
   submission, no special casing, and no automatic yield points.
4. **Cooperative model, accepted.** Inside a `defer` coroutine, a long
   inline call with no extraction points occupies the coroutine's
   thread until it returns. This is the accepted cooperative trade-off;
   the remedy is cross-context submission, not preemption.
5. **Actor serialisation is local.** Inside a `delegate` actor, an
   inline call runs in the actor, serialised via its inbox. A long
   inline call causes head-of-line blocking *within the actor* only —
   it does not starve other actors.
6. **Actor threading is a strategy concern.** Whether a `delegate`
   actor runs on its own thread or shares a pool is a
   `DEFAULT_STRATEGY` decision, not a language-semantic guarantee.

### 7. Local reasoning — **RESOLVED**

### OQ7 Resolution — Ownership-Level Signal Restored

Adopted 2026-07-31 through the Concept Design Review (Convergence
Check) process.

**Question:** Must the reader know the context type to understand what
`ctx <- fn()` does to the caller (block, yield, or continue)?

**Answer:** The two-operator family (OQ5 Resolution) restores the
ownership-level signal at the call site. `<-` visibly means "message to
a single owner" (serialised); the distribution operator visibly means
"dispatch to stateless workers" (parallel). The reader no longer needs
the context type to know *which relationship* is in play.

The residual ambiguity — the exact policy *within* each operator (yield
vs. block vs. dispatch) — is accepted as a conscious trade-off,
mitigated by:

1. **Proximity.** Contexts are typically created on the line immediately
   above their first use (`let ctx = delegate(obj); ctx <- ...`).
2. **Naming.** Conventional context names (`pool`, `worker`, `counter`,
   `mailbox`) communicate intent.
3. **Type-system guarantee.** The operator-context pairing is checked at
   compile time; the reader can never observe the wrong policy.
4. **Ownership precedes policy.** The ownership relationship is the
   load-bearing distinction for correctness; the exact scheduling policy
   is an implementation detail the programmer rarely needs to pin down
   at the call site.

**Conclusion:** Acceptable. The local-reasoning loss that motivated the
question is largely removed by the two operators; the remainder is
mitigated and is a deliberate simplicity trade-off (less syntax, slightly
less local policy detail).

### 8. Context destructor contract — **RESOLVED (see OQ8 Resolution)**

> The destructor contract is resolved in two parts: the wrapped object
> drops automatically as a consequence of ownership + lifetime (not a
> new rule), and pending `spawn`/`fork` work requires explicit
> resolution before destruction. The speculative body below is retained
> for traceability.

Does every context automatically clean up its wrapped resource when
destroyed, or is cleanup the programmer's responsibility?

```orthon
let ctx = delegate(open("data.txt"))
ctx <- read_all()
# ctx goes out of scope — is file.close() called automatically?
```

**Option A (default):** Contexts clean up their wrapped resources by
default. Consistent with the Lifetime dimension's scope-based
destruction (Semantic Invariant 3) and Ownership's single-owner model.
The context owns the resource; when the context is destroyed, it calls
the resource's destructor. `using` is pure sugar over `{ ctx =
delegate(obj); ... }`.

**Option B (explicit):** Contexts do NOT automatically clean up
wrapped resources. The programmer must call `release(ctx)` or
`close(resource)` explicitly before the context is destroyed. `using`
adds semantics beyond sugar — it guarantees cleanup that bare context
usage does not.

**Possible answer (superseded by OQ8 Resolution):** Option A. Automatic
cleanup is consistent with the Principle of Least Astonishment: a file
handle created inside a context should not leak when the context is
destroyed. `using` remains pure sugar, and the context destructor
contract is uniform across all context types.

### OQ8 Resolution — Context Destructor Contract

Adopted on 2026-07-31 through the Concept Design Review (Convergence
Check) process.

**Question:** Does destroying a context automatically clean up its
wrapped resource, or is cleanup the programmer's responsibility?

**Answer (two parts):**

1. **Wrapped object — automatic, as a consequence of ownership, not a
   new rule.** `delegate(obj)`/`defer(obj)` move the object into the
   context; the context is its single owner (OQ5 Resolution). Destruction
   at the context's final scope exit therefore drops the wrapped object
   exactly as any owned value is dropped (Semantic Invariant 1 + 3,
   [`SEMANTIC_MODEL.md`](../../../../what/SEMANTIC_MODEL.md) § Lifetime —
   Deterministic destruction). `take(ctx)` moves the object back out —
   destruction happens once, at the new owner (Ownership ↔ Lifetime
   pairing). `using` remains pure sugar. There is no "explicit release"
   alternative for the wrapped object: making the context not drop its
   owned value would violate the ownership model.

2. **Pending work on `spawn()`/`fork()` — no safe automatic behaviour
   exists; explicit resolution is required.** Silent cancellation loses
   work, silent joining blocks, silent detaching leaks — none is a safe
   default. The developer must resolve pending work explicitly before
   the context is destroyed (materialise via `next()`/`gather()`, or
   cancel via `stop()`, per OQ4 Resolution). Destroying a `spawn`/`fork`
   context with unresolved work is a program error — never silent loss,
   blocking, or leak. The manner of worker shutdown (graceful drain,
   cooperative cancel, immediate kill) is a `DEFAULT_STRATEGY` decision
   (Phase 7), not a Core Language semantic. Enforcement (runtime check
   vs. static liveness) follows the Ownership precedent: the semantic
   contract is Core; the enforcement mechanism is Implementation
   Strategy.

3. **`delegate`/`defer` pending messages need no separate guard.** The
   mailbox/coroutine state is owned machinery; destruction mid-life is
   ordinary drop of the owned object, and unprocessed invocations die
   with it — consistent with the ownership model, not a silent-loss
   case.

**Scope of the guard:** the explicit-resolution requirement applies to
`spawn()`/`fork()` only — contexts that dispatch to stateless workers
whose results are materialised via the generator protocol.

**Conscious consequence:** fire-and-forget (submit work, discard the
context, let the worker run to completion) is not expressible without a
`detach()` mechanism. This is tracked as a separate research hypothesis
(see [`../important/DETACHED_EXECUTION.md`](../important/DETACHED_EXECUTION.md)),
not resolved here.

**Consequence for OQ2:** the return-type question (handle vs. void)
remains open; this resolution assumes each submission yields a
resolvable unit of work that can be materialised or cancelled before
destruction.

---

## Evaluation

### Strengths

- **Orthogonal.** Function (what) and context (how) are independent axes.
  Any context composes with any function.
- **Minimal.** One concept (Invocation) + context constructors replace
  `call` + `delegate send` + `spawn`/`fork` + `async`/`await`.
- **Two-operator family, not per-policy proliferation.** `<-` (single owner)
  and the distribution operator (stateless workers) are two members of one
  Submission family. The operator set is closed (2); the constructor set is
  open.
- **Extensible.** New context types (GPU, cluster) require only a new
  constructor — no new syntax.
- **Colourless functions.** No sync/async bifurcation. Any function
  can be invoked with any context.
- **Natural base case.** `fn(args)` is Immediate by default — no
  operator clutter on the most common invocation form.
- **Context = tool metaphor.** Create the tool, then use it. Matches
  how programmers think about resources.
- **Clear boundary.** `.` for immediate calls, `<-` for contextual
  submission — two intents, two operators, no ambiguity.

### Open Risks

- **Loss of local reasoning (partially mitigated).** The two operators now
  distinguish "message to a single owner" from "distribution to workers" at
  the call site. Within each operator the reader still must know the context
  type for the exact policy (yield vs. block vs. dispatch). Mitigated by
  typical proximity of context construction to first use.
- **Context boilerplate.** Creating a context before every non-immediate
  call adds verbosity. May need sugar for one-shot contexts.
- **`await(...)` as StdLib.** If `await` is not a keyword, can it be
  implemented efficiently? Does the compiler need intrinsic support?
- **Context lifecycle.** When is a context destroyed? `defer()`,
  `delegate()`, `spawn()`, `fork()` create resources — who owns them
  and when are they cleaned up?

---

## Decision History

| Date | Event |
|------|-------|
| 2026-07-29 | Hypothesis formulated: `fn<OP(args)` syntax with `<` between name and args |
| 2026-07-29 | Refined: `<` between name and args feels foreign. Model changed to prefix `~fn(args)` for deferred + binary `ctx <OP fn(args)` for other policies. |
| 2026-07-29 | Refined: `~` prefix removed. All non-immediate policies require an **explicit context** created by a constructor (`defer`, `delegate`, `parallel`, `remote`). Operator `<OP` works on context. `await(ctx)` is a regular function. |
| 2026-07-30 | Refined: **Single operator `<-`** replaces per-context operators (`<~`, `<-`, `<=`, `<@`, `<#>`). Context constructor determines policy, not the operator. Rationale: orthogonality (one concept → one mechanism), extensibility (new context types need no new syntax). |
| 2026-07-30 | Rejected: `.` for contextual invocation — conflicts with immediate method call syntax. |
| 2026-07-30 | Rejected: `->` for extraction — conflicts with function return type syntax. |
| 2026-07-30 | Rejected: reverse `<-` for extraction — creates operator ambiguity (direction depends on operand types). |
| 2026-07-30 | Confirmed: `await(ctx)` and `return(ctx)` as named functions for result extraction. No new operators. |
| 2026-07-30 | Renamed: `EXECUTION_POLICY_HYPOTHESIS.md` → `EXECUTION_CONTEXT_INVOCATION.md`. Old file deprecated. |
| 2026-07-30 | Integrated Context Manager (`using`) as syntactic sugar over Execution Context + Scope. `using` desugars to `delegate(obj)` + block + scope-bound destructor. No new semantics. `SCOPED_RESOURCE_LIFECYCLE.md` superseded. |
| 2026-07-30 | **OQ4 resolved.** Context-specific extraction vocabulary adopted. `defer`/`await`, `delegate`/`take`, `spawn`/`fork` with generator protocol (`next()`/`stop()`). `yield` removed — runtime yields on `await` automatically. `grab`/`gather` as StdLib sugar. Generator syntax deferred to Phase 5. `parallel()` and `remote()` replaced by `spawn()` and `fork()`. |
| 2026-07-31 | **OQ6 resolved.** Fixed inline call semantics adopted. `fn(args)` is context-independent — always synchronous inline; never blocks or yields by itself (the callee may suspend at an explicit extraction point). No self-delegation (`this <- fn(...)`); long computations are submitted to a stateless-worker context via the distribution operator (`pool |> fn(...)`) and materialised through the generator protocol (OQ4/OQ5). Actor threading deferred to `DEFAULT_STRATEGY`. |
| 2026-07-31 | **OQ5 resolved.** Single operator superseded by a **two-operator family by ownership axis**: `<-` = single owner (`delegate`, `defer`; both always wrap an object), distribution = stateless workers (`spawn`, `fork`). `Send`/`Move` marker traits check captured data at compile time. `<=` rejected as distribution glyph (comparison conflict); `<||`/`|>` candidates, glyph deferred to Phase 5. |
| 2026-07-31 | **OQ7 resolved.** Local-reasoning concern addressed by the two-operator family: the ownership relationship (`<-` vs. distribution) is visible at the call site; residual per-policy ambiguity (yield vs. block vs. dispatch) accepted as a conscious trade-off with proximity, naming, and type-system mitigation. |
| 2026-07-31 | **OQ8 resolved.** Context destructor contract: the wrapped object drops automatically as a consequence of ownership + lifetime (not a new rule); `take(ctx)` = move-out; `using` remains pure sugar. For `spawn`/`fork`, no safe automatic behaviour for pending work exists — explicit resolution (materialise via `next()`/`gather()`, or cancel via `stop()`) is required before destruction; destroying a context with unresolved work is a program error. Worker-shutdown manner and enforcement are Implementation Strategy concerns (Phase 7). Fire-and-forget is not expressible without `detach()` — deferred to a separate research hypothesis (`../important/DETACHED_EXECUTION.md`). |

**Status:** Exploratory — not accepted. Requires resolution of open
questions before EDR.
