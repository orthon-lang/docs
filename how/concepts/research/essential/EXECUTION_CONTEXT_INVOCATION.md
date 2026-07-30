# Execution Context Invocation

> **⚠️ DRAFT — Concept research.**
> This document proposes a unified invocation model: a single **Invocation**
> operation submitted to an **Execution Context** via one operator `<-`.
> The context constructor determines the execution policy, not the operator.
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
an **Execution Context** via a single operator `<-`. The context, not the operator,
determines the execution policy.

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
The single operator `<-` submits invocations to that context. The context
determines how they execute.

```orthon
fn(args)                   — Immediate — execute now (no context)

ctx = defer(obj)           — coroutine context
ctx <- method(params)      — submit to coroutine

ctx = delegate(obj)        — mailbox context
ctx <- method(params)      — submit to mailbox

ctx = parallel()           — thread pool context
ctx <- method(params)      — submit to pool

ctx = remote(url)          — remote context
ctx <- method(params)      — submit to remote node
```

Invocation becomes **two-dimensional**:

```
What to invoke?  →  function / method
How to execute?  →  context (created by constructor, not encoded in operator)
```

The function answers only the first question. The context answers only the second.
The operator `<-` does one thing: **submit invocation to context**.

---

### Why One Operator, Not Many

An earlier version of this proposal (see [`EXECUTION_POLICY_HYPOTHESIS.md`](EXECUTION_POLICY_HYPOTHESIS.md))
proposed distinct operators per context type (`<~`, `<-`, `<=`, `<@`, `<#`). This was
rejected in favour of a single `<-`. Rationale:

**Argument for distinct operators:** Each operator visually encodes the execution
semantics at the call site. `ctx <~ fn()` signals "coroutine yield" to the reader;
`ctx <= fn()` signals "parallel dispatch." The reader does not need to know the
type of `ctx` to understand what happens to the caller.

**Argument for a single operator:** The context already knows its policy.
`delegate(Counter)` is a different type from `parallel()`. The operator's only
job is to transfer execution to the context — what the context does with it is the
context's responsibility. This is more orthogonal: the *what* (invocation) is
separated from the *how* (context), and the syntax does not duplicate information
already present in the type system.

**Decision:** Single operator `<-`. The context constructor determines the policy.
This is consistent with Orthon's orthogonality principle: one concept, one
mechanism. If the reader needs to understand what `ctx <- fn()` does, they look at
how `ctx` was constructed — typically on the line immediately above.

**Extensibility benefit:** A new context type (GPU, cluster, quantum) requires only
a new constructor — no new operator. The syntax surface does not grow.

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
result = await(ctx)      # Materialise result from any context
owner = return(ctx)      # Extract owner, terminate delegate
```

`await` and `return` are regular functions (with possible compiler intrinsic support).
Their semantics are polymorphic on context type — no new operators needed.

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
`defer(...)`, `delegate(...)`, `parallel()`, `remote(url)` are all
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
| `ctx <- fn(args)` | Submit to context — context determines how |

**Context constructors:**

| Constructor | Creates | Execution policy |
|-------------|---------|------------------|
| `defer(obj)` | Coroutine context | Cooperative, suspending |
| `delegate(obj)` | Mailbox context | Sequential, non-blocking |
| `parallel()` | Thread pool context | Parallel, non-blocking |
| `remote(url)` | Remote execution context | Network, non-blocking |
| — | Immediate (no context needed) | Synchronous, blocking |

---

### What This Replaces

#### `async` / `await` — eliminated

`async` is not a modifier on the function. The function is colourless.
`defer(obj)` creates a coroutine context; `<-` submits invocations to it.

```orthon
# Before:
async fn fetch(url) -> String
    return await httpClient.get(url)
let data = await fetch(url)

# After:
fun fetch(url) -> String           # colourless function
    return httpClient.get(url)

let ctx = defer()                   # create deferred context
ctx <- fetch(url)                   # submit to context
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
pool <- loadImage("a.jpg")
pool <- loadImage("b.jpg")
let results = await(pool)
```

#### `delegate send` — unified

```orthon
# Before:
let counter = delegate(Counter(0))
counter <- increment()

# After:
let counter = delegate(Counter())   # create delegate context
counter <- increment()              # submit to context
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
execution context (`delegate`, `defer`, `parallel`, `remote`)
already has a lifecycle — a constructor that wraps an object is
naturally a context whose destructor cleans up that object.
`delegate(open("data.txt"))` is already a resource-managing context;
no new constructor type is needed. The `using` sugar simply makes
this pattern more concise.

---

### Colourless Functions

A key consequence: **functions have no colour.** There is no `async fn`
vs `fn`. A function is just a computation. The caller creates the
appropriate context and submits invocations with `<-`.

```orthon
fun fetch(url: Url) -> String
    return httpClient.get(url)

# All valid, same function:
let a = fetch(url)                          # Immediate — blocks

let d = defer()
d <- fetch(url)                             # Deferred
let b = await(d)

let g = delegate()
g <- fetch(url)                             # Delegated
let c = await(g)

let p = parallel()
p <- fetch(url)                             # Parallel
let d = await(p)

let r = remote("https://worker.node")
r <- fetch(url)                             # Remote
let e = await(r)
```

---

### What Remains Separate

Three distinct operations, two syntactic mechanisms:

| Operation | Mechanism | Notes |
|-----------|-----------|-------|
| Create context | Constructor: `defer(obj)`, `delegate(obj)`, `parallel()`, `remote(url)` | Allocates resources, establishes policy |
| Submit invocation | Operator: `ctx <- call` | Single operator for all contexts |
| Materialise result | Function: `await(ctx)`, `return(ctx)` | Polymorphic on context type |

The model separates:
1. **Context construction** — allocating a mailbox, thread pool, coroutine
2. **Invocation submission** — `<-` transfers execution to the context
3. **Result materialisation** — `await(...)` / `return(...)` reads the result

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
- Contextual invocation is always `ctx <- call`.
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
| `parallel()` | StdLib | Thread pool is an implementation detail |
| `remote(url)` | StdLib | Network protocol is an implementation detail |
| `using` | Language (sugar) | Syntactic sugar over `delegate(obj)` + scope + destructor |
| `await(ctx)` | StdLib | Blocking/yielding behaviour depends on context |

### Current Documents Affected

| Document | Change |
|----------|--------|
| [`PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) | `call` primitive revisited — becomes Invocation + Context |
| [`DELEGATE.md`](DELEGATE.md) | Rewrite — `delegate` becomes context constructor, `<-` is the submit operator |
| [`CONCURRENCY_MODEL.md`] | Absorbed into context model |
| [`EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) | Substantial update — define execution contexts, `await()`, `return()`, resource lifecycle via context destructor |
| [`SYNTAX.md`](../../what/SYNTAX.md) | Update invocation syntax section — single `<-`; add `using` sugar |
| [`GLOSSARY.md`](../../what/GLOSSARY.md) | Update `Delegate`, add `Invocation`, `Execution Context`, `using`, remove `Spawn`, `Async` |
| [`CORE_CONCEPTS.md`](../../what/CORE_CONCEPTS.md) | Update CONCURRENCY_MODEL entry |
| [`DESIGN_PRINCIPLES.md`](../../how/DESIGN_PRINCIPLES.md) | Update Uniformity section |
| [`SCOPED_RESOURCE_LIFECYCLE.md`](../essential/SCOPED_RESOURCE_LIFECYCLE.md) | Superseded — absorbed into context model. `using` is syntactic sugar over context + scope, not a separate mechanism. |
| [`DECLARATIVE_CONSTRUCTS.md`](../../important/DECLARATIVE_CONSTRUCTS.md) | Update § Resource Management — replace standalone `using` analysis with desugaring to Execution Context pattern. |

---

## Open Questions

### 1. What does `defer(...)` wrap?

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
- Inside a deferred context: yields to the coroutine scheduler.
- Inside a delegated context: the delegate is single-threaded,
  so `await` would process the next mailbox message.

The behaviour of `await` depends on **where** it is called, not just
**what** context it reads from. This is the ambient policy question.

### 5. Composition with ownership

`delegate` applies to **state owners**, not arbitrary code. The `<-`
operator serialises access to the owner's state. But `parallel()`
applies to any stateless function. How does the type system distinguish?

**Possible answer:** The function's type signature reveals whether it
captures mutable state. `ctx <- fn()` on a parallel context with a
state-capturing closure is a compile error.

### 6. Ambient policy inside contexts

If a function submitted to a deferred context calls `fn(args)` (Immediate)
inside its body, does that block the deferred context or yield?

**Possible answer:** The execution policy is ambient. Inside a
delegated context, `fn(args)` (Immediate) means "execute in this
context synchronously" — it processes the call inline without
blocking the caller's thread. Inside a deferred context, `fn(args)`
means "eagerly evaluate within the current coroutine." The unmarked
`fn(args)` is policy-aware, not context-independent.

### 7. One operator: loss of local reasoning?

With a single `<-`, the reader must know the type of `ctx` to understand
whether `ctx <- fn()` blocks, yields, or dispatches. Is this acceptable?

**Mitigation:** Contexts are typically created on the line immediately
above their first use. Naming conventions (`pool`, `worker`, `counter`)
communicate intent. The type system guarantees correctness. This is a
conscious trade-off: less syntax for slightly less local explicitness.

### 8. Context destructor contract

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

**Possible answer:** Option A. Automatic cleanup is consistent with
the Principle of Least Astonishment: a file handle created inside a
context should not leak when the context is destroyed. `using` remains
pure sugar, and the context destructor contract is uniform across all
context types.

---

## Evaluation

### Strengths

- **Orthogonal.** Function (what) and context (how) are independent axes.
  Any context composes with any function.
- **Minimal.** One concept (Invocation) + context constructors replace
  `call` + `delegate` + `spawn` + `async`/`await`.
- **Single operator.** `<-` means exactly one thing: submit to context.
  No operator-per-policy proliferation.
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

- **Loss of local reasoning.** `ctx <- fn()` does not visually signal
  whether the caller blocks, yields, or continues. The reader must
  know the context type. Mitigated by typical proximity of context
  construction to first use.
- **Context boilerplate.** Creating a context before every non-immediate
  call adds verbosity. May need sugar for one-shot contexts.
- **`await(...)` as StdLib.** If `await` is not a keyword, can it be
  implemented efficiently? Does the compiler need intrinsic support?
- **Context lifecycle.** When is a context destroyed? `defer()`,
  `delegate()`, `parallel()` create resources — who owns them and
  when are they cleaned up?

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

**Status:** Exploratory — not accepted. Requires resolution of open
questions before EDR.
