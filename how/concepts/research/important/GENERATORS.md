# Generators (Yield)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via ECR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How do you produce sequences of values lazily — computing each element on demand?

Eager collection building (list comprehensions) is simple but wasteful for large or infinite sequences:
- **Memory** — the entire sequence exists in memory at once.
- **Latency** — the caller waits for the entire collection to be built before receiving the first element.
- **Infinite sequences** — impossible to represent eagerly.

Manual iterator classes (`__next__`, `hasNext`) solve the laziness problem but require:
- **Boilerplate** — separate class with state fields, `next()` method, done signal.
- **State management** — the programmer manually maintains position and termination state.

The core problem: **a function should be able to suspend and resume, yielding values one at a time**, without the programmer managing iterator state manually.

## Principles

1. **Laziness by construction** — Generator functions produce values on demand, not eagerly.
2. **Automatic state** — The compiler preserves function state between yields; no manual fields needed.
3. **Composable** — Generators can be combined (map, filter, zip, flat_map) using combinator methods.
4. **Bidirectional** — `yield` should optionally accept values back from the consumer (two-way generators).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Generator Model Policy | Governs stackless (Rust/Python) vs stackful (Lua) generator semantics |
| Iteration Policy | Defines how generators integrate with `for` loops and iterator traits |
| Send/Receive Policy | Controls one-way (yield only) vs two-way (yield + send) generators |

## Model (What)

A generator is a function that uses `yield` to produce values one at a time. The function's state is automatically saved between yields. The caller consumes values lazily.

```
# Generator function with yield
fun fib() -> Iterator<Int>
    a, b = 0, 1
    loop:
        yield a
        a, b = b, a + b

# Usage with for loop
for f in fib():
    print(f)
    if f > 100 then break

# Generator expressions (comprehensions)
numbers = (x * x for x in 1..10 if x % 2 == 0)
# Produces: 4, 16, 36, 64, 100

# Combinator chains on iterators
fib().take(10).filter(|n| n > 5).collect()
# Returns: [8, 13, 21, 34]
```

Key features:
- `yield` keyword produces a value and suspends the function.
- **Automatic state machine** — local variables persist across yields.
- **Iteration integration** — generators are consumed with `for` syntax.
- **Send values back** — optional `yield expr` where `expr` receives a value from the caller.
- **Generator expressions** — inline generator syntax: `(x * x for x in 1..10 if x % 2 == 0)`.
- **Iterator combinators** — `.take(n)`, `.filter(pred)`, `.map(fn)`, `.collect()` — compose lazy transformations without intermediate allocations.
- **Exhaustion** — iterators are single-use (consumed after iteration). The compiler prevents reuse.

## Default Strategy

Stackless generators (state machine). One-way by default (yield only, no receive). Generator expressions desugar into generator functions.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Stackful generators | Separate call stack per generator; can yield from nested calls (Lua coroutines). |
| Async generators | `async yield` — generator that can also await at yield points. |
| Lazy iterator combinators | No `yield` keyword; combinators like `.map()`, `.filter()`, `.flat_map()` on lazy iterables. |
| Eager production | Traditional list/array building — no laziness. |

## Open Questions

1. Should generators implement the same trait as collections for polyglot iteration?
2. How does `yield` interact with destructors and resource cleanup?
3. Should generators support `return value` to signal final value (Python's `StopIteration.value`)?
4. Can generators be composed with async/await? (Async generators.)

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
