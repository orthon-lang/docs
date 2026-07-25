# Class with `act` modifier

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It captures the working model under discussion for Orthon's reference type
> and concurrency isolation mechanism. Not yet validated through Concept Design
> Review or accepted via EDR.
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How many type constructors does a language need for reference semantics, behaviour,
and concurrency safety?

Existing languages provide varying answers:

| Language | Reference type | Concurrency type | Behaviour reuse |
|---|---|---|---|
| **Java** | `class` | `synchronized` (modifier) | `interface` + `extends` |
| **Swift** | `class` | `actor` (separate keyword) | `protocol` |
| **Kotlin** | `class` | `Mutex` (library) | `interface` |
| **Rust** | `Box`/`Rc`/`Arc` | `Mutex`/`RwLock` (library) | `trait` |

Each model introduces either:
- A **separate type** for concurrent state (`actor` in Swift), adding cognitive load, or
- A **modifier** on existing primitives (`synchronized` in Java), losing compile-time safety.

The core problem: **can a language provide compile-time concurrent safety without
multiplying type constructors, and without losing explicitness?**

## Hypothesis

Orthon uses a single reference type `class` with an optional `act` modifier for
compile-time concurrent isolation.

### Declaration

```
class Counter
    private act value: Int = 0

    act increment()
        value += 1

    act get_value() -> Int
        return value

    func describe() -> String       // non-act, synchronous
        return "a counter"
```

### Field isolation rules

- `act` field — only accessible from `act` methods or `act new()` constructor.
  Compiler-enforced. Cannot be read/written from non-`act` methods or from outside
  the class.
- Non-`act` field — regular class field, accessible from any method. No isolation
  guarantee.

```
class Example
    act isolated: Int = 0
    plain: Int = 0

    act act_method()
        isolated = 1     // OK
        plain = 1        // OK (non-act field from act method)

    proc regular_method()
        isolated = 1     // COMPILE ERROR: act field from non-act method
        plain = 1        // OK
```

### Method dispatch

```
class Counter
    act increment()
        ...

    func describe() -> String
        return "a counter"

let c = Counter::new()

c.describe()               // OK: synchronous direct call
c.increment()              // COMPILE ERROR: act method, use <-
c <- increment()           // OK: message send via mailbox
```

### `<-` operator semantics

The `<-` operator sends a message to the actor's mailbox and delegates execution
to the actor's message loop:

```
// Fire-and-forget: no return value expected
c <- log_event("started")

// With return value: result is a future/handle
let future = c <- compute(x)

// With await: suspend until result is ready
let result = await (c <- compute(x))
```

**Open:** exact semantics of `<-` return type.

### Zero-cost property

If a `class` has no `act` fields, no mailbox is created. The class is a plain
heap-allocated reference type with direct synchronous calls:

```
class Logger
    name: String

    proc write(msg: String)       // synchronous, no mailbox
        print("[{name}] {msg}")
```

The compiler determines mailbox presence at compile time by scanning for `act` fields.
No runtime check, no dynamic dispatch.

### Traits and `act`

A `class` can implement traits. Trait methods can be `act` if they access isolated
state, or non-`act` if they don't:

```
trait Incrementable
    act increment()

class Counter
    implements Incrementable

    private act value = 0

    act increment()
        value += 1
```

**Open question:** Can a trait require `act` on its methods? What does that mean for
implementors that don't have `act` state?

## Principles

1. **One reference type** — `class` is the only reference type in Orthon.
   No separate `actor` keyword. Concurrency safety is a modifier, not a type.

2. **Explicit isolation** — `act` marks exactly which fields and methods participate
   in the isolation guarantee. No implicit assumptions.

3. **Zero-cost by construction** — no mailbox unless `act` fields exist. No runtime
   overhead for non-concurrent classes.

4. **Visible dispatch** — `<-` operator makes every concurrent message send
   syntactically distinct from direct calls.

5. **No inheritance** — classes derive from `object` only. Behaviour reuse via
   trait conformance. No fragile base class, no virtual dispatch, no deep hierarchies.
   See [`CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`](CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md).

## Comparison with alternatives

| Approach | Languages | Pros | Cons |
|---|---|---|---|
| **Separate `actor` type** | Swift | Clear semantics, compiler safety | Cognitive load, duplication with `class` |
| **`synchronized` modifier** | Java | Simple, familiar | No compile-time safety, runtime-only |
| **Library-based (Mutex)** | Rust, Kotlin | Flexible, zero-cost | No compiler safety, error-prone |
| **`class` + `act`** | Orthon (hypothesis) | Single ref type, compiler safety, zero-cost | Novel, unproven |

## Open Questions

1. **Reentrancy during `await`:** Can a second `act` method execute while the first
   awaits? (Swift: yes; Erlang: yes) What invariants break if it does?

2. **`<-` return type:** Does `c <- method()` return `void`, `Future<T>`, or require
   `await` at the call site?

3. **Sendable constraint:** How does the compiler verify that a `class` reference
   passed to an `act` method is safe? Automatic for `struct`, explicit annotation
   for `class`?

4. **`act` on static methods:** Can a static method be `act`? What would that mean
   (class-level isolation)?

5. **Error propagation:** If an `act` method fails, does the actor die? Can it be
   restarted? Does the error propagate through `<-`?

6. **Object safety for traits:** Can a `dyn Trait` with `act` methods be used for
   dynamic dispatch? How does the vtable handle isolation?

## See also

- [`ACTORS.md`](ACTORS.md) — research on actor model in other languages
- [`STRUCT_AS_VALUE_TYPE.md`](STRUCT_AS_VALUE_TYPE.md) — struct value type hypothesis
- [`CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`](CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md)
  — composition patterns
- [`TRAITS.md`](TRAITS.md) — trait system
- [`ASYNC_AWAIT.md`](ASYNC_AWAIT.md) — async/await integration
- [`CONCURRENCY.md`](CONCURRENCY.md) — structured concurrency
