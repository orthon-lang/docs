# EDR-051: Push Streams — StdLib Observable-Style Reactive Streams

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

Orthon's sequence model defines two fundamental production mechanisms:
- **Eager production** — function `return` (single value)
- **Lazy production** — generator `emit` (pull-based sequence, per EDR-021)

The research document at `how/concepts/research/important/PUSH_STREAMS.md` proposes a third fundamental mechanism: **push-based streams** where the producer determines when values are emitted, and the consumer reacts to them. This is the dual of pull-based iteration:

```
Generator (pull):    consumer ---next()---> producer
Stream (push):       producer ---push()---> consumer
```

The question: **should push streams be a built-in language mechanism (alongside functions and generators), or should they be a Standard Library pattern?**

Analysis: Push streams are implementable as StdLib using:
- The delegate model (EDR-033) for independent execution contexts
- Channels (StdLib, per EDR-049) for pushing values to consumers
- Generators (`emit`, EDR-021) for pull-based production
- Callback-based subscription patterns

The conceptual model (`return` → single value, `yield`/`emit` → pull sequence, `push` → push sequence) is elegant, but the push side is fully expressible via composition of existing constructs. No new compiler-level semantics required.

The Decision Pipeline classified PUSH_STREAMS as **StdLib**: Push-based reactive streams are the dual of pull-based sequences. Implementable via composition of existing constructs (delegate, channel, callback).

---

### Decision

Adopt push streams as a **Standard Library abstraction** built on existing constructs:

1. **`Stream<T>` type** — A push-based observable that emits values to subscribed consumers. The producer pushes values; the consumer receives them asynchronously.

    ```orthon
    let stream = Stream<Int>.create()  # Create a stream
    let sub = stream.subscribe(fn (v)  # Subscribe a consumer
        print(v)
    )
    stream.emit(1)
    stream.emit(2)
    stream.complete()
    ```

2. **Construction patterns** — Streams can be created from:
   - Delegate-based producers (delegate pushes values via `send`)
   - Generator conversions (pull → push bridge: consuming an `Iterator[T]` and pushing values to subscribers)
   - Event sources (timers, I/O, GUI events)
   - Manual `emit` + `complete` + `error` control

3. **Combinators** — Stream transformations via combinator chains (map, filter, reduce, merge, combine_latest, throttle, debounce). All StdLib, no language changes.

    ```orthon
    let doubled = stream.map(fn (x) -> x * 2)
    let filtered = stream.filter(fn (x) -> x > 0)
    let merged = stream1.merge(stream2)
    ```

4. **Backpressure** — Configurable backpressure strategy:
   - `Buffer` — buffer values until consumed (bounded or unbounded)
   - `Drop` — drop values when consumer is slow
   - `Block` — block the producer when consumer is slow

5. **Cancellation** — Subscription returns a `Disposable` that can cancel the subscription:

    ```orthon
    let disposable = stream.subscribe(handler)
    disposable.dispose()  # Cancel subscription
    ```

6. **Error handling** — Streams have an error channel parallel to the data channel. Errors terminate the stream:

    ```orthon
    stream.on_error(fn (err) -> log(err))
    ```

7. **Completion** — Streams have a completion signal. Subscribers receive on_data, on_error, and on_complete callbacks.

**Key principle:** The `Stream<T>` type uses the delegate model (EDR-033) internally for concurrent delivery. The producer and consumer may be in different delegates. Push streams are ordinary Orthon code — no special compiler treatment.

---

### Consequences

**Positive:**
- Zero language additions — push streams are fully StdLib, preserving Minimal Core.
- The pull/push duality is conceptually clear: generators for pull, streams for push, both StdLib-composable.
- Delegate-based delivery provides data-race freedom by construction.
- Backpressure is configurable per-application, not baked into the language.
- Combinators match established patterns (RxJS, ReactiveX, Kotlin Flow).
- LLM-friendly: the observable/stream pattern is well-understood.

**Negative:**
- No special syntax for stream subscription (no `for <- stream` syntax) — subscription is via `.subscribe()` or `.listen()`.
- Stream + generator interop requires explicit conversion (pull → push bridge), not automatic.
- No built-in operator fusion — combinator chains may have higher per-event overhead than a built-in stream type.
- Backpressure strategies must be chosen explicitly; no universal strategy exists.

---

### Compliance

1. The `Stream<T>` type must be implementable using only existing language constructs (delegate, channel, generator, callback).
2. No new keywords, syntax, or compiler changes are required.
3. Backpressure strategies must be built on channel capacity and delegate scheduling, not language-level primitives.
4. Combinators must return new `Stream<T>` values without mutating the source stream.
5. Cancellation must be explicit via `Disposable` — no implicit lifecycle management.
6. Error handling must follow Orthon's `Result<T,E>` model for typed error channels.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Built-in `push`/`emit` duality with language-level syntax | Would add new keywords and syntax for push semantics. Streams are fully expressible as StdLib. The language cost (keywords, parser changes, compiler support) outweighs the benefit. |
| ReactiveX/RxJS library only | A specific library would tie Orthon to a particular reactive model. A StdLib `Stream<T>` provides the abstraction; third-party libraries can provide alternative implementations. |
| No push streams in v0.1 | Push streams address a real need (GUI, events, reactive data flow). Deferring would force users to implement ad-hoc callback patterns. StdLib classification means zero language cost. |
| Go-style channels only | Channels provide push semantics but conflate transport with buffering. A `Stream<T>` type provides richer semantics (error channel, completion, backpressure strategies, combinators). |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (Standard Library addition): `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE`.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I need to react to events as they arrive — network data, UI events, sensor readings." Streams are the proven solution. Serves VISION.md's Comfortable by Design pillar — a unified push model eliminates ad-hoc callback patterns. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "Push streams are fully expressible as StdLib on the delegate model." Verification: `Stream<T>` wraps a delegate + channel; combinators are function compositions; subscription is callback registration; backpressure is channel capacity management. All are ordinary Orthon code. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "A stream pushes values to subscribers — like a generator, but the producer decides when values arrive." StdLib classification means streams can evolve independently. No conceptual debt — the reactive stream pattern is production-proven across RxJS, Kotlin Flow, and Java Stream. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | Stream semantics are well-defined: `stream.emit(v)` pushes; `stream.subscribe(fn)` receives; `stream.complete()` signals end; `stream.error(e)` signals error. No self-referential paradoxes. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Streams live in the Standard Library layer, depending on delegates (Core) and channels (StdLib). No layer violations. |

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — StdLib addition, not a language construct. `LLM_GENERABILITY_GATE` — StdLib functions have standard LLM generability properties.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
