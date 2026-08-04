# Concurrency — StdLib Utilities Built on Delegate Model

> **✅ ACCEPTED — [EDR-049](../../how/decision_records/architecture/EDR-049-concurrency.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **Classification:** **StdLib** — channels, select, supervision, timers, and
> structured concurrency patterns built on the delegate model
> ([`CONCURRENCY_MODEL.md`](CONCURRENCY_MODEL.md), EDR-033). No new language
> semantics.
>
> **See also:** [`CONCURRENCY_MODEL.md`](CONCURRENCY_MODEL.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Channel, Delegate,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Orthon's concurrency model (EDR-033) defines the **language-level** model:
delegate-based concurrency with `act` modifier, `delegate` keyword, `<-`
message operator, and ownership transfer rules. This document covers the
**StdLib concurrency utilities** that programmers use day-to-day: typed
channels, `select` expressions, supervision trees, timers, async I/O wrappers,
and structured concurrency patterns (fan-out, fan-in, pipeline).

The core problem: **concurrent programming needs higher-level building blocks**
than raw delegate message passing. A `Channel<T>` with typed `send`/`receive`
is far more ergonomic than hand-managing delegate mailboxes; `select` makes
waiting on multiple channels composable.

## Principles

1. **Built on the delegate model** — Every concurrency utility is implementable
   using the delegate model's primitives (`delegate`, `<-`, `$` ownership
   transfer). No new compiler-level semantics.
2. **StdLib, not language** — Channels and `select` are library types/functions,
   not language constructs. No new keywords.
3. **No shared mutable state** — Utilities preserve the "no shared mutable state"
   invariant of CONCURRENCY_MODEL.
4. **CSP-style, not actor-style** — Channels match Go-style CSP concurrency built
   on the delegate runtime; the `actor` concept is superseded by `delegate`
   (EDR-033).
5. **Supervision is a pattern** — Failure recovery is a library/runtime pattern
   with configurable policy, not a language concern.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Concurrency Policy | Determines the default execution model for concurrent utilities (delegate-based) |
| Channel Policy | Governs buffered/unbuffered channel semantics and backpressure |
| Scheduling Policy | Controls how delegated contexts are scheduled |

## Model (What)

### Typed Channels

`Channel<T>` provides typed message passing between delegates, wrapping
delegate message queues:

```orthon
let ch = Channel<String>(buffer: 10)
delegate producer:
    ch.send("hello")
delegate consumer:
    let msg = ch.receive()
```

### `select` Expression

Wait on multiple channels simultaneously — the first ready channel triggers
its branch. `select` is a StdLib function/macro:

```orthon
select:
    case msg = ch1.receive():
        handle(msg)
    case msg = ch2.receive():
        handle(msg)
    case default(timeout: 1s):
        handle_timeout()
```

### Fan-out / Fan-in

StdLib functions distribute work across worker delegates and collect results,
built on `delegate` + `Channel`.

## Default Strategy

The standard library provides `Channel<T>`, `select`, supervision-tree helpers,
timers, and async-I/O wrappers implemented on the delegate runtime. Buffered and
unbuffered channels are both available; backpressure follows the channel buffer
configuration.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Language-level channels | Adds channel syntax to the compiler — rejected (EDR-049): channels are fully implementable as StdLib types wrapping delegate mailboxes. |
| Language-level `select` | Syntactic sugar over delegate message polling — rejected: a StdLib macro/function provides equivalent ergonomics. |
| Built-in supervision trees | Supervision is a runtime/library pattern, not a language concern — rejected. |
| Actor syntax (`actor` keyword) | Superseded by `delegate` as the language primitive (EDR-033) — rejected. |

## Open Questions

1. Should channels support bounded backpressure primitives beyond buffer size?
2. How do supervision policies compose with the Error Handling model (EDR-020)?

## Decision History

- **EDR-049:** Concurrency accepted as StdLib — concrete async/concurrent
  patterns building on CONCURRENCY_MODEL (EDR-033) without adding new language
  semantics.
- **Classification per D-03:** StdLib. Channels, select, supervision, timers,
  and async I/O are all expressible via delegate primitives and ownership
  transfer.
- **Cross-reference:** CONCURRENCY_MODEL (EDR-033) is the Language-level model;
  this document is its StdLib companion. Do not confuse the two.
