# Hypothesis: Observer as a Library Pattern on Top of Delegated Execution

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
>
> This hypothesis proposes that the Observer pattern does not require
> language-level constructs in Orthon. Instead, it emerges naturally as
> a library pattern built on existing primitives — `delegate`, classes,
> and delegation calls (`<-`).
>
> **Related:** `EVENTS.md` (same research tier), `DELEGATE.md` (essential/ —
> the execution model this hypothesis builds on), `PUSH_STREAMS.md` (important/ —
> stream evolution path). `ACTORS.md` is superseded by `DELEGATE.md`.
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax consistent with the
> current `delegate` hypothesis. Final syntax is subject to language-wide
> agreement and will be specified in Phase 5 (Syntax).

---

## Problem

Many systems need to react to state changes in an object without a hard
coupling between the event source and its handlers.

Typical examples:

- UI updates;
- logging;
- caching;
- notifications;
- reactive computations;
- event-driven pipelines.

The classical solution is the **Observer** pattern: an object maintains a
list of subscribers and notifies them when its state changes.

In Orthon, a more fundamental interaction model already exists —
**delegated execution** (`delegate` + `<-`). Delegation wraps a state-owning
entity in a serialised execution context and forwards invocations through it.

This raises the question:

> Does the Observer pattern need dedicated language support, or is it a
> special case of message dispatch to multiple receivers?

---

## Proposed Solution

Do not introduce Observer as a language construct.

Instead, implement it in the standard library on top of existing primitives:

- `class` (state owner)
- `delegate` (execution policy)
- `<-` (delegation call)
- lifetime management (context)

```orthon
class Sensor
    value: int
    listeners: list<delegate(int)>

    proc subscribe(d: delegate(int)):
        listeners.add(d)

    proc set(v):
        value = v
        for l in listeners
            l(value)

let sensor = delegate(Sensor())
sensor <- subscribe(someHandler)
sensor <- set(42)
```

Subscribers receive notifications through ordinary delegation. No special
language features are required.

---

## Why This Is Sufficient

In the Orthon model, every interaction between independent entities already
occurs through delegation of intent to execute an action.

Therefore:

```
Observer

↓

message delegation

↓

Delegated Execution
```

Observer is not a separate execution model — it is a special case of the
existing message-dispatch mechanism.

---

## Standard Library Evolution Path

Higher-level abstractions can be built on this foundation without language
changes:

```
delegate
    │
    ▼
  Signal<T>
    │
    ▼
  Event<T>
    │
    ▼
  Stream<T>
    │
    ▼
  Reactive Pipelines
```

Observer becomes the simplest representation of an event stream. See
[`PUSH_STREAMS.md`](../../important/PUSH_STREAMS.md) for the parallel
hypothesis on push-streams as a built-in abstraction.

---

## Advantages

### Minimal Language Core

No new semantics are added to the language. Everything is built from
existing primitives.

---

### Unified Interaction Model

No separate rules for:

- calls;
- events;
- notifications;
- observers.

All use the same message-dispatch mechanism via `delegate`.

---

### Safety Through Isolation

If subscribers are backed by `delegate`, each subscriber's state changes
occur within its own serialised execution context. This eliminates
classic Observer problems:

- data races;
- concurrent access;
- need for external synchronisation.

See [`DELEGATE.md`](../../essential/DELEGATE.md) for the isolation
guarantees of delegated execution.

---

### Natural Extension to Streams

Observer becomes the first step toward richer abstractions:

- Signal
- Event
- Stream
- Reactive Streams

All without language changes.

---

### Component Independence

The event source knows nothing about subscriber implementation. It merely
delegates a message to each registered handler.

---

## Disadvantages and Trade-offs

### Delivery Cost

Each notification becomes a delegation call. This is more expensive than a
direct synchronous invocation.

---

### Asynchrony by Default

A delegation call does not guarantee immediate execution. This means:

- possible delivery delay;
- potential reordering;
- eventual consistency.

This is acceptable for UI and distributed systems, but unsuitable for
algorithms requiring strict synchrony.

---

### Lifetime Management

The source must correctly remove subscribers. This can be handled through:

- `context`-scoped lifetimes;
- weak references;
- automatic detachment when the delegated entity is destroyed.

---

### Notification Order

With multiple subscribers, the language does not guarantee message-processing
order. If deterministic ordering is required, the library or application
must provide it.

---

## Related Concepts

### Observer

The classical subscriber-list pattern.

---

### Publish–Subscribe

Observer with an intermediate message broker. Easily implemented as a
dedicated delegated entity.

---

### Event Bus

A global mediator between publishers and subscribers. Also naturally
implemented as a delegated entity.

---

### Signal/Slot (Qt Model)

A typed Observer pattern. Maps naturally onto `delegate`.

---

### Reactive Streams

Generalises Observer by adding stream composition, filtering,
transformation, and backpressure.

---

### CSP (Communicating Sequential Processes)

Components communicate through channels. Distinct from Observer in that
data flows through channels rather than messages to an executor.

---

### Delegated Execution

The closest model. Observer becomes a special case of multi-receiver
message dispatch. See [`DELEGATE.md`](../../essential/DELEGATE.md) for
the core hypothesis.

---

## Alternatives

### Observer as a Language Construct

```orthon
observe sensor.changed
```

**Pros:**

- compact syntax;
- less boilerplate.

**Cons:**

- new language construct;
- duplicates the message-dispatch model;
- complicates the specification.

---

### Signal as a Built-in Type

```orthon
signal<int> changed
```

Reduces boilerplate but introduces a new fundamental entity that is, in
essence, a specialised container of delegates.

---

### Framework-Only Implementation

Implement Observer only in the UI framework, for example.

Suitable for graphical applications, but limits reuse in server,
distributed, and embedded systems.

---

## Conclusion

In the Orthon architecture, the **Observer** pattern does not require
special language support. It arises naturally from composition of existing
primitives — `delegate`, `class`, and delegation calls.

This approach preserves the minimality of the language core, provides a
unified interaction model, and enables evolution from simple observers to
richer abstractions — Signal, Stream, reactive pipelines — without
language changes. Observer becomes not a separate language concept but the
first step of Orthon's event architecture.

---

## Decision History

Initial hypothesis — no decisions recorded yet.

---

### Cross-References

- [`DELEGATE.md`](../../essential/DELEGATE.md) — Delegated execution model
  (the primitive this hypothesis builds on)
- [`EVENTS.md`](EVENTS.md) — Events research (same tier, complementary
  analysis)
- [`PUSH_STREAMS.md`](../../important/PUSH_STREAMS.md) — Push-stream model
  (stream evolution path)
- [`ACTORS.md`](ACTORS.md) — Superseded by `DELEGATE.md` (historical context)
