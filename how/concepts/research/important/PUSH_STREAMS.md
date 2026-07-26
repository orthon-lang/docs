# Hypothesis: Built-in Push-Stream Model as a Primary Language Mechanism in Orthon

## Problem

Most modern languages are designed around a **pull model of computation**.

A function returns a value, a generator produces a sequence of values, an iterator is polled by a consumer:

```text
consumer -> next() -> producer
```

Even async generators remain a pull model — the consumer decides when to get the next element.

However, a large class of problems requires the data source itself to determine when an event occurs:

* sockets;
* HTTP/WebSocket;
* GUI;
* file system events;
* timers;
* sensors;
* message brokers;
* reactive UI.

In every case, additional infrastructure must be built:

* callbacks;
* EventEmitter;
* Observable;
* Channels;
* Streams;
* Signal/Slot;
* AsyncQueue;
* Rx.

Nearly every modern language solves this with libraries rather than language primitives.

Consequences:

* multiple incompatible stream models emerge;
* pull and push are difficult to compose;
* reactivity becomes a "bolt-on" layer rather than part of language semantics.

---

## Proposal

Make **push-streams** (Producer Streams) a built-in language abstraction alongside functions and generators.

The idea:

* generator = pull stream
* stream = push stream

The language supports both models symmetrically.

```
Generator

consumer ----next()----> producer


Stream

producer ----push()----> consumer
```

Instead of an endless set of callback APIs, a single language model applies.

Conceptually:

```orthon
stream numbers {
    emit 1
    emit 2
    emit 3
}
```

or

```orthon
stream socketEvents(socket) {
    while socket.open {
        emit socket.read()
    }
}
```

Subscription:

```orthon
socketEvents(socket)
    .map(parse)
    .filter(valid)
    .listen(print)
```

---

## Why This Belongs in the Language

Today, a language knows only two fundamental computation mechanisms:

* ordinary function call
* generator (yield)

Push-streams are equally fundamental.

Framed as:

```
return  -> single value
yield   -> pull sequence
emit    -> push sequence
```

All three become basic ways to produce results.

---

## Potential Properties

The language can guarantee uniform rules for:

* stream completion;
* errors;
* cancellation;
* backpressure;
* composition;
* asynchrony.

No separate Rx, EventEmitter, or custom framework needed.

---

## Examples

Pull:

```orthon
generator files(path) {
    yield ...
}
```

Used as:

```orthon
for file in files(path)
```

Push:

```orthon
stream fileChanges(path) {
    emit ...
}
```

Used as:

```orthon
fileChanges(path)
    .listen(update)
```

or

```orthon
for event <- fileChanges(path)
```

(if the language provides dedicated subscription syntax).

---

## Advantages

### 1. Unified Event Model

All language events share the same interface.

No distinction between:

* mouse;
* network;
* GUI;
* file;
* websocket.

---

### 2. No Callback Hell

Instead of:

```python
socket.on(...)
```

A standard language construct.

---

### 3. Simple Composition

If a stream is a first-class language type,

operations become natural:

```
map
filter
merge
zip
combine
debounce
buffer
window
```

without third-party libraries.

---

### 4. Unified Async Model

Instead of:

```
async
await
callback
observable
event
channel
```

A single fundamental event-stream abstraction.

---

### 5. Better Fit for Reactivity

Reactive Programming becomes a natural extension of the language rather than a library.

---

## Trade-offs

### 1. Significant Language Complexity

A new fundamental execution model.

This affects:

* type system;
* lifetimes;
* scheduler;
* async;
* exceptions;
* cancellation.

---

### 2. Backpressure

The main challenge of the push model.

If the source is faster than the consumer:

```
producer >>> consumer
```

A strategy must be defined:

* block the producer;
* buffer;
* drop messages;
* cancel the stream.

This must be part of the specification.

---

### 3. Lifetime Management

Must define:

when is a stream destroyed?

who owns it?

who calls dispose?

---

### 4. Cancellation

A uniform mechanism is needed:

```
subscription.cancel()
```

or

```
stream.close()
```

---

### 5. Increased Cognitive Load

The user must understand the distinction between:

```
fun
generator
stream
```

---

### 6. Async Integration

Must decide:

```
Is a stream always async?
```

or

```
Can synchronous push streams exist?
```

This affects the entire language model.

---

## Related Concepts

Similar ideas already exist in various ecosystems:

* Python generators (pull)
* Async generators
* Reactive Streams
* Rx
* Observable
* Kotlin Flow
* Channels
* Go channels
* C# IObservable
* EventEmitter
* Signals/Slots
* Actor model
* CSP

However, nearly everywhere push is implemented as a library or framework, not as part of language syntax.

---

## Alternatives

### 1. Keep Push as a Library Concept

As most languages do today.

Pros:

* simpler language;
* less semantics to specify.

Cons:

* many incompatible implementations.

---

### 2. Use Only async/await

Treat events as ordinary async functions.

Cons: no full stream model emerges.

---

### 3. Use Only Channels

As in Go.

Cons: a channel is a data-transfer primitive, not a declarative stream with compositional operations.

---

### 4. Make Generators Bidirectional

Extend `generator` to support both pull and push.

Pros: fewer new entities.

Cons: two different data-flow control models mix; generator semantics become more complex.

---

## Open Questions

1. Should `stream` be a standalone language entity, or is a generalization of generators sufficient?
2. Is a `stream` inherently async, or are synchronous push-streams permissible?
3. Should the language prescribe a fixed **backpressure** policy, or allow configurable strategies?
4. Should operations like `map`, `filter`, `merge`, `zip`, etc. be built into the language or provided by the standard library?
5. Can `generator` and `stream` be unified under a common "value stream" abstraction differing only in control direction (pull vs push), or would this lead to an overly complex model?

## Preliminary Assessment

The hypothesis appears internally consistent and fits well with Orthon's philosophy of expressing fundamental computation models through explicit constructs (`fun`, `proc`, `new`, generators, etc.). The greatest complexity lies not in the `stream` syntax itself but in formalizing the semantics of lifecycle, cancellation, error handling, and backpressure. If these aspects can be made as predictable as generator semantics, a built-in push model could become a distinguishing feature of the language. Otherwise, there is a risk of importing all the complexity of modern reactive frameworks into the language core.
