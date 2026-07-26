# Delegate as an Execution Policy

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
>
> This hypothesis proposes removing actors from the language surface.
> Instead, Orthon exposes a single execution policy — `delegate`.
> Internally, the runtime may implement it using actors, mailboxes, queues,
> executors, fibers, or any other mechanism. The language specification does
> not mandate a concrete implementation.
>
> **Related:** `ACTORS.md` (superseded — actor removed from language concepts),
> `ACT_AS_ACTIVE_OBJECT.md` (superseded), `ACT_AS_FUNCTION.md` (superseded).
> **Reference:** [../../../notes/actor-implementation-taxonomy.md](../../../notes/actor-implementation-taxonomy.md) —
> taxonomy of 12 actor implementation approaches catalogued during design-space exploration.

---

## Issue (Why)

In Orthon, mutable objects (list, set, map, user-defined types) exist in **direct
mode** by default — their methods are called synchronously and directly. Under
concurrent access from multiple threads or fibers, data races and the need for
manual synchronization (mutexes, locks) arise easily. This adds accidental
complexity, contradicts the "Intent Over Implementation" principle, and
introduces a whole class of concurrency bugs.

Today, most languages address this by introducing concurrency as a separate
language feature:

- `async`
- `actor`
- `coroutine`
- `goroutine`

Each introduces its own object model and execution semantics. As a result,
programmers must decide *how* to represent computation before writing business
logic.

Orthon instead separates two concerns:

- **what** is executed (function, method, object)
- **how** it is executed (execution policy)

We need a way to make any existing (or new) object **concurrently safe with
minimal syntactic cost**, preserving the convenience of method calls while
eliminating accidental direct access from outside.

---

## Hypothesis

**Object delegation** wraps an instance inside a lightweight execution context
managed by the Orthon runtime. The delegate accepts messages and processes them
sequentially, guaranteeing race-free access. The delegation model supports three
operations unified through the keyword `delegate`:

1. **Create a new object directly in delegate mode**

   ```
   let lst = delegate(List())
   let nums = delegate([1, 2, 3])
   ```

   The object is born inside the delegate — direct access never exists.

2. **Transition an existing object from direct to delegate mode**

   ```
   let shared = delegate(move lst)
   ```

   Ownership is moved into the delegate. After this, `lst` becomes invalid,
   and `shared` has type `DelegatedContext<List>`.

3. **Return from delegate mode to direct mode**

   ```
   let direct = release(move shared)
   ```

   The delegate drains remaining messages, then ownership returns to the
   synchronous context. `shared` becomes invalid; `direct` is `List` again.

Internally, the runtime may implement delegation using actors, mailboxes,
queues, executors, fibers, or any other mechanism — the language specification
does not mandate a concrete implementation.

---

## Design Principle

Execution is orthogonal to declaration.

The following declarations remain unchanged:

```
func
proc
class
lambda
bound method
```

Delegation is applied afterwards.

```
callable (state owner)
      │
      ▼
delegate(...)
      │
      ▼
Delegated Execution Context
```

---

## Core Principle: Delegate Applies to State Owners, Not Code

> **`delegate` applies to the owner of mutable state, not to arbitrary code.**

This principle eliminates ambiguity about what can and cannot be delegated:

- `delegate(List())` — **valid.** `List` owns its internal state.
- `delegate(counterClosure)` — **valid.** The closure owns its captured state.
- `delegate(list.insert)` — **invalid.** `insert` is a method — an operation on
  the owner. It does not own state; it provides one of several access paths
  to state owned by `list`.

This principle is deeper than "actor under the hood": it ties the concurrency
model to the ownership model. **State is the unit of serialization**, not
functions or methods. This makes semantics consistent and predictable.

Consider a `Bank` with three methods:

```
class Bank
    balance: int
    proc deposit(amount) { balance += amount }
    proc withdraw(amount) { balance -= amount }
    proc transfer(to, amount) { ... }
```

Only one delegation is valid:

```
let bank = delegate(Bank())    // Valid — Bank owns its state
```

These are **not** valid:

```
let deposit  = delegate(bank.deposit)   // Invalid — method, not state owner
let withdraw = delegate(bank.withdraw)  // Invalid — method, not state owner
```

All three operations share the same mutable state (`balance`). Delegating
them independently would create conflicting execution contexts over the same
state — a correctness violation. Delegating the owner (`Bank`) serializes
all access through a single execution context.

---

## Delegate is an Execution Closure

A closure captures variables.

```
closure =
    code
    + environment
```

A delegate captures execution.

```
delegate =
    callable
    + execution policy
```

The execution policy may include:

- mailbox
- scheduler
- queue
- synchronization
- lifetime
- cancellation
- priorities

The language does not expose these implementation details.

---

## Uniform Model

### Functions (stateless — delegation not applicable)

```
proc log(msg)
    print(msg)

// log is stateless — delegate(log) is unnecessary
// Call directly: log("hello")
```

Functions without captured mutable state do not need delegation.
They are naturally safe for concurrent use.

### Closures (state-owning — delegation valid)

```
proc makeCounter()
    count = 0
    return proc()
        count += 1
        return count

let counter = delegate(makeCounter())

counter <- ()   // → 1
counter <- ()   // → 2
```

The closure owns `count`. Delegation serializes access to it.

### Objects (state-owning — delegation valid)

```
let bank = delegate(Bank())

bank <- deposit(100)
bank <- withdraw(30)
```

### Lambdas (state-capturing — delegation valid)

```
let worker =
    delegate(
        (x) => transform(x)
    )

worker <- value
```

Everything follows the same model: **delegate the state owner.**

---

## Mailbox Ownership

Delegation is applied at the **state owner** level.

### Object delegation (valid)

```
let list = delegate(List())
```

All methods share the same execution context.

```
delegate(List)

        mailbox

 append
 erase
 clear
 sort
```

This naturally serializes access to mutable state. Every method call
on the delegated object goes through the same mailbox.

### Method delegation (invalid)

```
let insert = delegate(list.insert)   // Invalid
let erase  = delegate(list.erase)    // Invalid
```

Methods do not own state — they are operations on the owner. Delegating
them independently would create conflicting execution contexts over the
same underlying state, violating the core principle.

---

## Language Semantics

The language specifies only:

- delegated calls preserve ordering
- delegated execution is isolated
- `<-` transfers execution responsibility

The specification intentionally does **not** define:

- actor
- mailbox implementation
- scheduler
- thread
- coroutine
- event loop

Those are runtime implementation details.

---

## Possible Runtime Implementations

A compiler/runtime may implement delegation using:

- Actor model
- Thread pool
- Fiber scheduler
- Event loop
- Lock-free queues
- OS threads

All satisfy the same observable language semantics.

---

## Why This Is Better Than Language Actors

Actors become an optimization rather than a language primitive.

Instead of:

```
act Counter
```

the programmer writes:

```
let counter = delegate(Counter())
```

Instead of:

```
act func process(...)
```

the programmer writes:

```
let process = delegate(process)
```

Business logic never depends on actor semantics.

---

## Benefits

- One concurrency model for every callable entity.
- No special actor syntax.
- No distinction between active and passive objects.
- Runtime remains free to choose the optimal implementation.
- Business logic is independent of the concurrency backend.
- Orthogonal design.
- Smaller language surface.
- Easier to optimize.
- Future execution models can be added without changing the language.

---

## Trade-offs

### Flexibility vs. Model Simplicity

Supporting `delegate(move ...)` and `release(move ...)` transitions enables the
**populate-then-delegate** pattern — construct in direct mode (no mailbox
overhead), populate, then hand off to a concurrent environment, and optionally
return for synchronous post-processing. This is especially valuable for
expensive initialization.

The cost is a more complex mental model: a variable can change modes (via
shadowing with a new `let`), and the programmer must track which mode is
current. The compiler helps by distinguishing types `T` and
`DelegatedContext<T>`, catching errors statically.

### Explicit Ownership via `move`

Transitions always require explicit `move`. This is slightly verbose but fully
consistent with Orthon's ownership model and avoids implicit transfers. Risk of
accidental loss of the delegated object is eliminated.

### Actor Overhead

Method calls through a delegate (even after `<-` is introduced) add message
serialization and dispatch costs. For hot loops, direct mode is preferred, then
delegate the finished result. This is an intentional trade-off: safety and
convenience in exchange for a small performance cost in concurrent access.

### Two New Keywords: `delegate` and `release`

These extend the language vocabulary but are orthogonal and do not overlap with
other concepts. `release` could have been a method on `DelegatedContext`, but a
keyword (or built-in function) emphasizes its fundamental role in the ownership
model.

### Deferred `<-`/`->` Operators

Message-sending operators are not part of this hypothesis. This leaves the
model incomplete for practical use (how do you send commands to a delegate?),
but allows focusing first on ownership and lifecycle without mixing them with
asynchrony and effect systems. This follows the Single Responsibility Principle
at the language design level.

---

## Related Concepts and Alternatives

### Related Orthon Concepts

- **Ownership model and `move`** — the foundation for transferring objects into
  and out of delegates. `delegate(move x)` and `release(move x)` are ownership
  transfers first, delegation operations second.
- **Asynchrony and concurrency** — delegates become a core building block for
  parallel computation after message operators (`<-`/`->`) are introduced.
- **Type safety** — static distinction between `T` and `DelegatedContext<T>`
  prevents an entire class of concurrency bugs at compile time.

### Alternatives (Considered and Rejected)

| Alternative | Rationale for Rejection |
|-------------|------------------------|
| Mode fixed at creation only (no `delegate(move ...)`, no `release`) | Simpler, but loses populate-then-delegate and synchronous-return idioms — too restrictive for real use |
| `List() as delegate` syntax | Reads better per "Data First", but reserves the `as` keyword needed for type casting; less symmetric with transition syntax |
| `:=` shorthand (`let lst := List()`) | Violates Semantic Purity (one symbol → one meaning) and Explicitness (behavior-changing operation is invisible) |
| Full actors (Erlang, Akka) | Heavyweight — require explicit behavior protocol. Our approach: "actor as invisible shell" around a regular object, preserving its interface |
| Automatic delegation (annotations, type inference) | Contradicts the Explicitness principle — delegation must be syntactically visible |

---

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Execution Policy | Core — `delegate` is an execution policy factory |
| Concurrency Policy | Delegated contexts may execute concurrently |
| Lifetime Policy | Delegated context lifetime is explicit |
| Scheduling Policy | Internal to runtime; not exposed to language |

---

## Observation

This represents a qualitative shift in the language architecture.

The original hypothesis was:

> **The language has an actor, and `act` is how you declare one.**

It now becomes significantly more abstract:

> **The language has only delegated execution (`delegate`). Actor is merely one possible implementation of that policy.**

A further refinement ties delegation to state ownership:

> **`delegate` applies to the owner of mutable state, not to arbitrary code. State is the unit of serialization.**

This aligns with Orthon's core philosophy: the language defines a **behavioral contract**, not a concrete implementation mechanism. The ownership model and the concurrency model are not separate concerns — they are two facets of the same design. This abstraction leaves the runtime free to choose the optimal execution strategy without changing the programming model.

---

## Ownership Integration: Move + Shadow + Release

### Problem

The current hypothesis wraps a state owner in a delegated execution context:

```
let list = delegate(List())
list <- append(item)
```

But `list` (the original variable) remains accessible in the caller's scope.
Nothing prevents:

```
list.append(item)        // direct mutation — bypasses mailbox!
```

Isolation is not guaranteed. The delegate does not *own* the state — it merely
*references* it. True concurrency safety requires the delegate to take
exclusive ownership.

### Principle

> **`delegate` takes ownership of the state it wraps. After delegation, the
> original binding is invalidated. All access must go through the delegated
> execution context via `<-`.**

Ownership and execution policy are two sides of the same operation:
`delegate(move x)` transfers both.

### Move Semantics

The caller transfers ownership of the state owner *into* the delegate:

```
let lst = List()
lst.add(1)
lst.add(2)

let lst = delegate(move lst)    // move into delegate, rebind via shadow
// lst.add(3)                   // COMPILE ERROR: lst was moved
```

`move lst`:
1. Transfers ownership of `List` from the outer scope into the delegate's mailbox.
2. Invalidates the original binding — any subsequent use of `lst` referencing
   the moved value is a compile error.

The delegate now holds the *only* path to the state. All mutations are
serialized through its mailbox.

### Shadowing

The same variable name is reused via shadowing (`let lst = ...`), but its
type changes:

| Before move | After move |
|---|---|
| `lst: List` | `lst: DelegatedContext<List>` |
| `lst.add(4)` — direct `.` call | `lst <- add(4)` — delegated `<-` call |

The operator (`<-` vs `.`) is governed by the *type* of `lst` at any given
point. The name stays the same — the programmer thinks about `lst`
throughout. The change in operator makes the dispatch mode syntactically
visible at every call site.

```
List ──delegate(move)──▶ DelegatedContext<List>
  .add()                      <- add()
```

### Delegated Call Syntax

```
lst <- append(4)
lst <- remove(0)
lst <- clear()
```

Semantics:
- `append`, `remove`, `clear` resolve in the **namespace of `List`**,
  not the delegate itself.
- `<-` transfers the Method Descriptor (pure data) to the delegate's mailbox.
- The delegate executes it when its execution context is ready.
- FIFO ordering is preserved within the same caller.

If the delegated context exposes its own methods (e.g., lifecycle), they are
called via `.` — synchronous, outside the mailbox:

```
lst.status()           // direct call — method on DelegatedContext
lst <- append(4)       // delegated call — method on List via mailbox
```

No name conflict: `.` and `<-` route to different namespaces.

### Release: Returning Ownership

Ownership is returned via `release`:

```
let lst = release(move lst)
// lst: List — direct access restored
```

`release(move lst)`:
1. Drains the mailbox — all pending messages are processed.
2. Transfers ownership of `List` back to the calling scope.
3. Destroys the `DelegatedContext` — `lst` is now `List` again.
4. The original `lst` (DelegatedContext) is invalidated.

```
DelegatedContext<List> ──release(move)──▶ List
  <- add()                                 .add()
```

The same shadowing pattern applies in reverse: the variable name stays,
the type changes back, the operator reverts to `.`.

### Lifetime Guarantees

| Guarantee | Mechanism | Enforcement |
|---|---|---|
| State inaccessible after `delegate` | `move` — original binding invalidated | Compile-time |
| All mutations serialized | Mailbox — one message at a time | Runtime |
| No surviving reference to inner state | No other path exists after `move` | Compile-time |
| State returned intact after `release` | Ownership transferred out, mailbox drained | Compile-time + Runtime |

### Error Cases

```
let lst = delegate(move lst)    // OK
let d = delegate(move lst)      // COMPILE ERROR: lst was already moved

lst <- append(4)                // OK (DelegatedContext)
lst = release(move lst)         // OK
lst.append(5)                   // OK (List restored)
```

### DelegatedContext<T>

`DelegatedContext<T>` is a transparent wrapper type with static guarantees:

| Property | Rule |
|----------|------|
| Non-copyable | The wrapper cannot be copied — only one reference exists |
| Non-dereferenceable | The wrapped value cannot be accessed directly via `.` or dereference |
| Namespace routing | `.` calls resolve to `DelegatedContext<T>` methods (lifecycle); no calls resolve directly to `T` methods |
| Type distinctness | `DelegatedContext<T>` is a different type than `T` — the compiler enforces the correct dispatch at every call site |

`DelegatedContext<T>` has no public access path to the inner `T` value. The only
way to interact with the wrapped state is through delegated message sending
(via `<-`, designed separately) or by returning ownership via `release(move x)`.

### Interaction with Other Hypotheses

- **`OWNERSHIP.md`** — `move` uses the same ownership transfer semantics.
  `release` is the inverse: a move *out* of the delegate.
- **`DISPOSABLE_PATTERN.md`** — `release` with auto-cleanup parallels
  `using`/`with` scoped resource management.
- **`CLASS_WITH_ACT.md`** — If `class` uses `act` fields, `delegate(move obj)`
  provides runtime isolation for a type that already has compile-time
  isolation markers. The two models compose rather than compete.

---

## Open Questions

- How does the language detect state ownership at compile time? (What is the
  formal rule for "this entity owns mutable state"?)
- Should `delegate` be a built-in or a standard library construct?
- ~~What is the lifetime model for delegated contexts? (Explicit `stop`/`cancel`? Reference counting?)~~
  → **Answered:** Lifetime is managed by `move`/`release` — explicit, deterministic.
- Does `<-` require language-level syntax, or can it be a method call?
- ~~How does delegation interact with Orthon's ownership model? (This is now
  the central question — the two models must be co-designed.)~~
  → **Answered:** `delegate(move x)` transfers ownership; `release(move x)` returns it.

---

## Decision History

| Date | Decision |
|------|----------|
| 2026-07-26 | Hypothesis created. Actor removed from language concepts. `ACTORS.md`, `ACT_AS_ACTIVE_OBJECT.md`, `ACT_AS_FUNCTION.md` superseded by this hypothesis. |
| 2026-07-26 | **Refinement:** `delegate` applies to state owners, not arbitrary code. State is the unit of serialization. Method-level delegation (`delegate(obj.method)`) is invalid — methods are operations on state, not state owners. Concurrency model tied to ownership model. |
| 2026-07-26 | **Refinement:** Ownership integration — `delegate` takes ownership via `move`. Shadowing pattern: same variable name, type changes, operator changes to `<-`. `release` returns ownership. `OWNERSHIP.md` open question resolved. |
| 2026-07-26 | **Refinement:** Hypothesis restructured with three explicit syntax forms (create in delegate mode, transition into delegate, return to direct mode). `DelegatedContext<T>` wrapper type formally defined (non-copyable, non-dereferenceable, type-distinct). Problem statement broadened to include concurrent access safety. |
| 2026-07-26 | **Decision:** Init-time delegation uses `delegate(...)` — rejected `as delegate` to keep `as` available for type casting. |
| 2026-07-26 | **Decision:** Transitions require explicit `delegate(move x)` / `release(move x)` — rejected fixed-mode-only design. |
| 2026-07-26 | **Decision:** Rejected `:=` shorthand for delegation (violates Semantic Purity + Explicitness). |
| 2026-07-26 | **Decision:** Rejected full actors as language feature (delegate-as-shell is lighter). |
| 2026-07-26 | **Decision:** Rejected automatic delegation without `delegate` keyword (violates Explicitness). |
| 2026-07-26 | **Decision:** `<-`/`->` message operators deferred to separate hypothesis. |
