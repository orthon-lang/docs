# Push Streams — StdLib Observable-Style Reactive Streams

> **✅ ACCEPTED — [EDR-051](../../how/decision_records/architecture/EDR-051-push-streams.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **Classification:** **StdLib** — push-based reactive streams are the dual of
> pull-based sequences, implementable via composition of existing constructs
> (delegate, channel, callback). No new language semantics.
>
> **See also:** [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`CONCURRENCY.md`](CONCURRENCY.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Stream,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Orthon's sequence model defines two fundamental production mechanisms:
**eager production** (`return`, single value) and **lazy production** (`emit`,
pull-based sequence, per EDR-021). A third mechanism exists in practice:
**push-based streams**, where the producer determines when values are emitted
and the consumer reacts. This is the dual of pull-based iteration:

```
Generator (pull):    consumer ---next()---> producer
Stream (push):       producer ---push()---> consumer
```

The core problem: **event-driven and reactive patterns need a push-based
abstraction** — timers, I/O events, GUI events, and live data feeds all push
values to consumers that react asynchronously.

## Principles

1. **Push is the dual of pull** — `Stream<T>` is the push-based counterpart to
   the pull-based `Iterator<T>`.
2. **StdLib, not language** — Push streams are implementable via existing
   constructs (delegate, channel, callback); no new compiler-level semantics.
3. **Subscription-based** — Consumers subscribe and receive pushed values
   asynchronously.
4. **Lifecycle control** — Streams expose `emit`, `complete`, and `error`
   control from the producer side.
5. **Composable with the sequence model** — Pull-to-push bridging (consuming an
   `Iterator[T]` and pushing to subscribers) is a supported construction.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Stream Policy | Governs push-based stream semantics (subscribe, emit, complete, error) |
| Evaluation Policy | Push values are evaluated eagerly by the producer (unlike lazy pull) |
| Concurrency Policy | Streams build on the delegate model (EDR-033) and channels (EDR-049) for async delivery |

## Model (What)

### `Stream<T>` Type

A push-based observable that emits values to subscribed consumers:

```orthon
let stream = Stream<Int>.create()      # Create a stream
let sub = stream.subscribe(fn (v)      # Subscribe a consumer
    print(v)
)
stream.emit(1)
stream.emit(2)
stream.complete()
```

### Construction Patterns

Streams can be created from:
- Delegate-based producers (delegate pushes values via `send`)
- Generator conversions (pull → push bridge: consuming an `Iterator[T]` and
  pushing values to subscribers)
- Event sources (timers, I/O, GUI events)
- Manual `emit` + `complete` + `error` control

## Default Strategy

The standard library provides a `Stream<T>` type built on the delegate model
(EDR-033) and channels (EDR-049). Producers push values; consumers receive them
asynchronously via subscription callbacks.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Built-in `push`/`emit` duality with language-level syntax | Adds new keywords and parser/compiler support — rejected (EDR-051): streams are fully expressible as StdLib; the language cost outweighs the benefit. |
| ReactiveX/RxJS library only | A specific library would tie Orthon to a particular reactive model — rejected: a StdLib `Stream<T>` provides the abstraction; third-party libraries can offer alternatives. |

## Open Questions

1. Should `Stream<T>` support backpressure signalling from consumer to producer?
2. How do error events interact with the Error Handling model (EDR-020)?

## Decision History

- **EDR-051:** Push Streams accepted as StdLib — observable-style reactive
  streams built on delegate + channel. The conceptual model (`return` → single
  value, `yield`/`emit` → pull sequence, `push` → push sequence) is elegant,
  but the push side is fully expressible via composition.
- **Classification per D-03:** StdLib. No new compiler-level semantics required.
