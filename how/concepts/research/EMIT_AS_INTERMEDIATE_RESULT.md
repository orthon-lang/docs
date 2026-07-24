# Hypothesis: `emit` as Intermediate Computation Result

## Problem

Most languages treat a function as a computation that returns **exactly one** result.

```
input
   │
compute
   │
return value
```

If a function must produce multiple values, languages introduce separate entities:

* generators (`yield`);
* iterators;
* Observable;
* Stream;
* Reactive Flow;
* callbacks.

"Multiple results" are treated as a separate language concept.

This complicates the language.

---

## Hypothesis

In Orthon, a function may return two kinds of results:

* intermediate (`emit`);
* final (`return`).

```
emit
emit
emit
return
```

Computation becomes a sequence of results.

`return` means:

> computation is complete.

`emit` means:

> computation is still ongoing, but a new valid result has arrived.

Thus a stream becomes a natural extension of an ordinary function, not a separate mechanism.

---

## Novelty

The novelty is not in having an analogue of `yield`.

The novelty lies in **changing the semantics of a function**.

A function is viewed as:

```
Input → Sequence<Result> → FinalResult
```

instead of:

```
Input → Result
```

In other words,

`emit` is not a generator.

It is a **partial result of computation**.

---

## Example

Today one writes:

```python
def parse(file):
    for line in file:
        yield parse_line(line)
```

In Orthon this reads differently:

```orthon
parse(file):
    for line in file:
        emit parseLine(line)

    return statistics
```

Here the function:

* publishes results incrementally;
* returns a final summary at the end.

This yields a natural model for long-running computation.

---

## Conceptual Model

A function execution can be viewed as a lifecycle.

```
start

↓

emit

↓

emit

↓

emit

↓

return
```

Each `emit` is a published snapshot of the current computation state.

Not an "iterator".

Not a "generator".

Not a "callback".

An **intermediate function result**.

---

## Advantages

### 1. Simple Model

Only two ways to complete a computation segment:

```
emit
return
```

---

### 2. No Separate Generator Needed

A generator becomes a special case of a function.

No new mental model required.

---

### 3. Good Fit for Pipelines

```
load()

↓

emit rows

↓

filter()

↓

emit filtered

↓

group()

↓

emit groups
```

All functions work the same way.

---

### 4. Suitable for Reactivity

Observable becomes a natural consequence of `emit`.

No separate syntax needed.

---

### 5. Works Well for Large Computations

For example:

```
download()

↓

emit progress

↓

emit chunk

↓

emit chunk

↓

return file
```

---

### 6. Compatible with Memory Safety

No new ownership model appears.

`emit` simply publishes a value temporarily.

---

## Disadvantages

### 1. More Complex Function Model

Today most programmers think:

```
function → value
```

Now:

```
function → values → value
```

This is a new abstraction.

---

### 2. The Question of Orphaned Emits

What if nobody reads an `emit`?

For example:

```
emit 1
emit 2
emit 3
return 5
```

Where did the first three values go?

They must be either:

* ignored;
* collected automatically;
* disallowed.

This must be defined by language semantics.

---

### 3. Completion Strategy Required

Can `emit` appear after `return`?

No.

And after:

```
emit
emit
```

is `return` mandatory?

If not — how does computation end?

---

## Trade-offs

### Instead of a Separate Generator

we get:

```
Function
    +
Intermediate Results
```

This is simpler.

But the function becomes richer in meaning.

---

### Instead of Callbacks

```
callback(value)
```

becomes

```
emit value
```

Much clearer.

---

### Instead of Observable

```
observer.onNext()
```

becomes

```
emit
```

Fewer abstraction layers.

---

### Instead of Stream API

```
stream.map().filter().collect()
```

chains of functions that simply use `emit` can be built.

---

## Compatibility

Very good.

### Pipes

```
read()

→

parse()

→

filter()

→

write()
```

---

### Reactive

```
emit
```

becomes a source of events.

---

### Lazy Evaluation

The next value is computed only when needed.

---

### Async

Here is where it gets interesting.

---

## Is an Analogue of `await` Needed?

I think **not necessarily**.

Because

```
emit
```

is not a counterpart to

```
await
```

They are different axes of the language.

---

`await` relates to:

> execution control.

```
run

↓

wait

↓

continue
```

---

`emit` relates to:

> publication of results.

```
compute

↓

emit

↓

compute

↓

emit
```

These are orthogonal mechanisms.

---

Therefore `emit` has no natural "companion" in the same way `async/await` does.

---

## What Truly Is `emit`'s Companion

In my view, the companion should be a **consumer of intermediate results**, not a waiting mechanism.

For example:

```orthon
parse(file)
    .onEmit(line => ...)
```

or

```orthon
for value in parse(file)
```

or even a dedicated language construct:

```orthon
consume parse(file):
    value =>
        ...
```

The pair looks like this:

```
Producer                Consumer

emit  ------------->    consume
```

not:

```
emit  -------------> await
```

Because `await` answers the question *"when should execution resume?"*, while `consume` answers *"what should be done with each intermediate result?"*.

## A Potentially More General Model

If this idea is taken to its logical conclusion, Orthon can separate three independent concepts:

| Concept                          | Operation | Purpose                                                           |
| -------------------------------- | --------- | ----------------------------------------------------------------- |
| Computation completion           | `return`  | Return the final result and terminate the function                |
| Intermediate result publication  | `emit`    | Publish an intermediate result and continue computation           |
| Execution suspension             | `await`   | Temporarily yield control to the runtime until an external event  |

This separation makes each construct responsible for exactly one semantic task. Unlike Python, where `yield` simultaneously means both value publication and execution suspension, in this model suspension becomes an internal implementation detail of `emit`, not part of its meaning. This, in my view, is the most interesting and potentially novel aspect of the hypothesis for Orthon.
