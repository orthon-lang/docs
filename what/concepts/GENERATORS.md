# Generators — Bidirectional Yield and Generator Expressions

> **✅ ACCEPTED — [EDR-050](../how/decision_records/architecture/EDR-050-generators.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`EMIT_AS_INTERMEDIATE_RESULT.md`](EMIT_AS_INTERMEDIATE_RESULT.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Generator, Yield

---

## Issue (Why)

The lazy sequence model (EDR-021) established the `emit` keyword for one-way lazy production — values produced on demand, consumer pulls via `next()`. However, there are patterns where the consumer needs to communicate back to the producer mid-iteration:

- **Interactive coroutines** — The consumer sends configuration or context to the producer between values.
- **Two-way protocols** — A generator acts as a state machine where each emitted value depends on the consumer's previous response.
- **Concise inline sequences** — Writing a full generator function for a simple transformation is verbose.
- **Generator delegation** — Combining multiple generators without manual iteration.

The core problem: **the one-way `emit` model cannot express producer-consumer interaction**, and there is no concise syntax for simple lazy sequences.

## Principles

1. **`yield` extends `emit`** — `yield` is a superset of `emit`. `yield` without a consumer return value is equivalent to `emit`. `yield expr` additionally receives a value from the consumer.

2. **Laziness by construction** — Generator functions produce values on demand, not eagerly.

3. **Automatic state** — The compiler preserves function state between yields; no manual field management.

4. **Composable** — Generators can be combined via `yield from` (delegation) and iterator combinators.

5. **Minimal syntactic addition** — Generator expressions provide concise inline syntax; `yield from` provides composition; both are sugar over existing primitives.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Generator Model Policy | Governs stackless (default) vs. stackful generator semantics |
| Bidirectional Policy | Controls whether generators support consumer-to-producer communication (default: enabled via `yield expr`) |
| Desugaring Policy | Formalises generator expression and `yield from` desugaring |
| Iterator Protocol Policy | Normal generators implement `Iterator[T]`; bidirectional generators implement `BidirectionalGenerator[T, U]` |

## Model (What)

### Bidirectional `yield`

`yield` provides two-way communication: producing a value and optionally receiving a value from the consumer.

```orthon
# One-way yield (equivalent to emit)
fun counter() -> Iterator[Int]
    let i = 0
    while i < 10:
        yield i          # produce value, no consumer interaction
        i = i + 1

# Bidirectional yield (receives value from consumer)
fun interactive() -> Iterator[String]
    let prefix = yield "ready"    # emit "ready", receive value
    yield prefix ++ ": working"   # use received value
    yield prefix ++ ": done"
```

The bidirectional form uses `BidirectionalGenerator[T, U]` trait where `next()` returns `T` and `send(value: U)` sends a value back:

```orthon
let gen = interactive()
let msg = gen@next()        # "ready" (first yield without receiver)
gen@send("user")            # send "user" back to generator
msg = gen@next()            # "user: working"
gen@send("admin")
msg = gen@next()            # "admin: done"
```

### Generator Expressions

Parenthesised inline syntax for simple lazy sequences:

```orthon
# Basic generator expression
let squares = (x * x for x in 1..10)

# With filter
let evens = (x for x in 1..100 if x % 2 == 0)

# With transformation
let names = (user.name for user in users if user.active)

# With map equivalent
let doubled = (x * 2 for x in items)
```

Generator expressions are lazy by default — they produce an `Iterator[T]` without materialising. They desugar to anonymous generator functions:

```orthon
# Desugaring:
let squares = (x * x for x in 1..10)
# → let squares = fun () -> Iterator[Int]:
#       for x in 1..10:
#           yield x * x
```

### `yield from` — Generator Delegation

Delegate production to a sub-generator:

```orthon
fun combined() -> Iterator[Int]
    yield from fib(10)      # delegate to fib(10)
    yield from fib(20)      # then to fib(20)

# Equivalent to:
fun combined() -> Iterator[Int]
    for v in fib(10):
        yield v
    for v in fib(20):
        yield v
```

### Relationship to `emit` (EDR-021)

The `emit` keyword (established in EDR-021) remains the canonical one-way form. The relationship:

| Construct | Direction | Consumer interaction | When to use |
|-----------|-----------|---------------------|-------------|
| `emit value` | One-way | None | Default for lazy sequences |
| `yield value` | One-way | None | Equivalent to `emit` |
| `yield expr` | Two-way | Receives value from consumer | Interactive generators |

## Default Strategy

Stackless generators (state machine). One-way by default. `yield` without expression ≡ `emit`. Bidirectional `yield` tracks a single "consumer sent value" slot in the state machine.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Stackful generators | Separate call stack per generator; can yield from nested calls (Lua coroutines). Higher memory overhead. |
| Bidirectional via channels | Use channels for producer-consumer communication instead of `send()` on the generator. More flexible but more verbose. |
| Eager production | Traditional list/array building — no laziness. Breaks infinite sequences. |

## Open Questions

1. Should `BidirectionalGenerator` be a separate trait or a parameterised `Iterator[T, U]`?
2. How does `yield from` compose with bidirectional generators?
3. Should generator expressions support async (`async (x for x in stream)`)?
4. How does bidirectional yield interact with ownership (sending owned values to the generator)?

## Decision History

- **2026-07-27** — Accepted via EDR-050. Classification: Language. Builds on LAZY_SEQUENCE_GENERATORS (EDR-021). Adds bidirectional yield, generator expressions, and yield-from delegation.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `../how/IMPLEMENTATION_POLICIES.md`
