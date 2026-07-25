# Hypothesis: `act` at the Function Level

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It explores whether `act` semantics can be lifted from the type level
> (`class` + `act`, or standalone `act` type) to the function level — creating
> "active functions" with mailbox and delegate semantics without requiring
> a wrapping type.
>
> **Last updated:** 2026-07-26
>
> **⚠️ SUPERSEDED by `DELEGATE.md` (2026-07-26).**
> The `act`-at-function-level hypothesis has been replaced. Delegated execution
> (`delegate`) provides uniform semantics for all callable entities — functions,
> methods, objects, and lambdas — without special-casing `act` at any level.
> See `DELEGATE.md` for the current hypothesis.
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

The `act` concept currently lives at the type level:

1. **Standalone `act` type** ([`ACT_AS_ACTIVE_OBJECT.md`](ACT_AS_ACTIVE_OBJECT.md)):
   `act Counter { ... }` — an active object that replaces both `class` and
   `coroutine`.

2. **`class` + `act` modifier** ([`CLASS_WITH_ACT.md`](CLASS_WITH_ACT.md)):
   `class Counter { act value: Int; act increment() }` — a single reference
   type with optional concurrency isolation.

Both approaches require the programmer to declare a type. But many use cases
need only a **single active callable** — a function that processes messages
through a mailbox, fire-and-forget, without the ceremony of a type declaration.

The question: **can `act` also apply at the function level, giving Orthon a
unified concurrency model that spans functions and types with a single
underlying mechanism (mailbox + delegate)?**

## Hypothesis

`act` is orthogonal to the function/type axis. It can modify:

- A **type** (existing hypotheses): `act` type or `class` + `act` field
- A **function** (this hypothesis): `act func` or `act` lambda

All uses share the same mailbox/delegate runtime model and the `<-` operator.

### Two Forms

#### Form A: `act func` — Named Active Function

```
act func process(item: Data) -> Result
    // body has access to persistent state across calls
    // (since it is an active object with a single entry point)
```

Call site:

```
// fire-and-forget
process <- (item)

// await result
let result = await (process <- (item))
```

The function body is the **sole message handler** for the implicit mailbox.
State declared inside the function body persists across invocations (unlike
a regular function where locals are recreated each call).

#### Form B: `act` Lambda — Anonymous Active Object

```
let worker = act (x: Int) -> Int
    x * 2

let result = await (worker <- (42))
```

An anonymous active object with a single procedure. Equivalent to an `act`
type with one `proc`. The lambda captures its enclosing scope (explicit
capture per function model in [`FUNCTIONS.md`](FUNCTIONS.md)).

### State Persistence

This is the critical design fork:

**Option 1 — Stateless (pure active function):**

```
act func compute(x: Int) -> Int
    x * 2   // no state between calls — fresh locals each message
```

Every message creates fresh locals. The mailbox guarantees FIFO ordering but
no state carries over. This is essentially an async function with
fire-and-forget syntax. Closer to `async func` in other languages, but with
mailbox dispatch instead of coroutine suspension.

**Option 2 — Stateful (single-method active object):**

```
act func counter(by: Int) -> Int
    let total: Mutable(Int) = 0   // persists across calls
    total += by
    total
```

State declared at the top of the function body persists across invocations.
This is equivalent to:

```
act CounterActor:
    private total: Int = 0
    proc increment(by: Int) -> Int:
        total += by
        total
```

but without the type declaration ceremony.

**Recommendation:** Option 2 (stateful) is the more interesting design. Option 1
is already covered by existing async/await models and adds little. The value of
`act func` is precisely that it gives the programmer a single-method active
object with zero boilerplate.

### Zero-Cost Property

If a function is called with `<-` exactly zero times (only `.` direct calls),
the mailbox is never created. The `act func` degrades to a regular `func` at
runtime — same as a `class` with no `act` fields.

### Interaction with `proc` and `func`

| Declaration | Semantics |
|---|---|
| `func f()` | Pure computation, no side effects, no mailbox |
| `proc p()` | May have side effects, synchronous |
| `act func af()` | Has mailbox, callable via `<-`, state persists across calls |
| `act proc ap()` | Has mailbox, callable via `<-`, with side effects |

`act` combines with both `func` and `proc` — the `act` modifier adds mailbox
semantics, while `func`/`proc` controls side-effect purity as usual.

### Method-Level `act` Inside `class`

This hypothesis is independent of whether `act` appears on a class or on a
function. They compose:

```
class Worker:
    act func handle(item: Data) -> Result   // act method with func purity
        ...

    act proc log(msg: String)               // act method with side effects
        ...
```

## Principles

1. **Orthogonality** — `act` applies uniformly to types and functions. The
   same `<-` operator, the same mailbox model, the same FIFO guarantees.

2. **Minimal ceremony** — A single-method active object should not require a
   type declaration. `act func` provides the same capability with lower
   syntactic overhead.

3. **Explicit dispatch** — `<-` vs `.` makes the dispatch mode visible at
   every call site, whether targeting a function or a method.

4. **Zero-cost by construction** — No mailbox unless `<-` is used at least
   once on a given function.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Concurrency Policy | Determines whether a function has a mailbox (act) or runs synchronously |
| Evaluation Policy | `<-` delegates execution; `.` calls directly |
| Lifetime Policy | State in `act func` persists across calls — lifetime tied to mailbox |
| Allocation Policy | Mailbox and persistent state allocated on first `<-` call |

## Comparison with Existing Models

| Model | Unit of concurrency | Fire-and-forget | State | Boilerplate |
|---|---|---|---|---|
| **Swift actor** | Type | No (async/await only) | Fields | Full type declaration |
| **Erlang process** | Function (`spawn`) | Yes (`!`) | Process dictionary | `spawn` + `receive` |
| **Go goroutine** | Function (`go`) | Yes (channel send) | Closure | `go func()` |
| **Rust async fn** | Function (`async fn`) | No (`.await` required) | Self-referential struct | `async fn` |
| **Orthon `act func`** | Function (`act func`) | Yes (`<-`) | Explicit in body | Inline, no type needed |

Key difference from Go: `act func` has a mailbox with FIFO ordering, not an
unbuffered channel. Key difference from Erlang: no explicit `receive` loop —
the function body *is* the handler, called once per message.

## Model (What)

### Basic Form

```
act func handler(msg: Message) -> Response
    // persistent state
    let seen: Mutable(Set[String]) = Set::new()

    seen.insert(msg.id)
    process(msg, seen)
```

Call site:

```
handler <- (Message::new("hello"))        // fire-and-forget
let resp = await (handler <- (Message::new("query")))  // await
```

### Lifecycle

`act func` is created on first use (lazy) or explicitly with `with`:

```
with handler as h:
    h <- (msg1)
    h <- (msg2)
// handler mailbox drained, state released
```

Without `with`, the `act func` lives for the duration of the scope where it
is declared (module-level = program lifetime, block-level = block lifetime).

### Composition with `emit`

An `act func` can `emit` intermediate results:

```
act func stream_processor(source: DataSource) -> Result
    for chunk in source:
        emit transform(chunk)
    finalize()
```

The caller subscribes to the emitted sequence. See
[`EMIT_AS_INTERMEDIATE_RESULT.md`](EMIT_AS_INTERMEDIATE_RESULT.md).

### `act` Lambda as HOF Argument

```
items
    .map(act (x) -> Int { expensive(x) })   // each runs through mailbox
    .reduce(sum)
```

The `act` lambda processes each element through its mailbox, enabling
structured concurrency within pipeline operations. See
[`CODE_BLOCK_AS_HOF.md`](CODE_BLOCK_AS_HOF.md).

## Unified Model

With `act` at both function and type levels, Orthon has a single concurrency
primitive:

| What you write | What it means |
|---|---|
| `act func f(x) -> T` | Active function — single-method active object |
| `act (x) -> T { ... }` | Active lambda — anonymous single-method active object |
| `class C { act field; act method() }` | Active class — multi-method active object |
| `act A { proc p() }` | Standalone active object (from ACT_AS_ACTIVE_OBJECT.md) |

All four use `<-` for delegate dispatch and share the same mailbox runtime.
The difference is only in how many methods the active entity exposes.

## Open Questions

### 1. State Persistence Semantics

If `act func` has persistent state, when is it initialized? On first `<-` call?
On declaration? Options:

- **Lazy:** State initialized on first `<-` call. No allocation if never used.
- **Eager:** State initialized at declaration. Consistent with `let` semantics.
- **With-scoped:** State initialized on `with` entry, destroyed on exit.

### 2. Recursive `act func`

```
act func factorial(n: Int) -> Int
    if n <= 1:
        1
    else:
        n * await (factorial <- (n - 1))
```

Does an `act func` calling itself through `<-` create a deadlock (mailbox waiting
for itself)? Or does the runtime detect self-send and handle it?

### 3. `act func` vs `act` Type: When to Use Which

If `act func counter(by: Int) -> Int` is equivalent to an `act` type with one
`proc`, is `act func` just syntactic sugar? Or does it have different semantics
(lifetime, visibility, capability)?

### 4. Module-Level `act func`

```
// at module level
act func global_handler(msg: Message)
    ...
```

What is its lifetime? Program lifetime? Is there a global mailbox shared across
all call sites?

### 5. `new` and `act func`

Can `new` be `act`? What would `act new()` mean — an active constructor? This
interacts with the `class` + `act` hypothesis in
[`CLASS_WITH_ACT.md`](CLASS_WITH_ACT.md).

## Decision History

None yet — this is an exploratory hypothesis. No decisions have been made.

---

### Affected Documents

- [ ] `ACT_AS_ACTIVE_OBJECT.md` — standalone `act` type hypothesis
- [ ] `CLASS_WITH_ACT.md` — `class` + `act` modifier hypothesis
- [ ] `FUNCTIONS.md` — function model
- [ ] `EMIT_AS_INTERMEDIATE_RESULT.md` — emit/delegate interaction
- [ ] `CODE_BLOCK_AS_HOF.md` — higher-order function integration
- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_STRATEGIES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
