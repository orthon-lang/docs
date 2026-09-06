# Generators — Generator Expressions over `emit`

> **✅ ACCEPTED — [EDR-050](../how/decision_records/architecture/EDR-050-generators.md).**
>
> **Status:** Accepted 2026-07-27. **Amended 2026-09-06** — the
> bidirectional form is withdrawn from the language model (S2): the
> `yield` / `yield from` keywords and the `BidirectionalGenerator[T, U]`
> trait are removed. Generators are **emit-only** — one-way production.
> The withdrawn bidirectional form is preserved as a hypothesis:
> [`COROUTINE_ON_YIELD.md`](../../how/concepts/research/deferrable/COROUTINE_ON_YIELD.md).
>
> **See also:** [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`EMIT_AS_INTERMEDIATE_RESULT.md`](EMIT_AS_INTERMEDIATE_RESULT.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Generator, Generator Expression

---

## Issue (Why)

The lazy sequence model (EDR-021) established `emit` as the one-way
production keyword — values produced on demand, consumer pulls via
`next()`. Two gaps remained:

- **Concise inline sequences** — Writing a full generator function for a
  simple transformation is verbose.
- **Generator delegation** — Combining multiple generators without manual
  iteration loops.

The core problem: there is no concise inline syntax for simple lazy
sequences. This concept adds **generator expressions** (sugar over
`emit`) and documents delegation as composition. It deliberately does not
add a consumer-to-producer (bidirectional) form — producers do not
consume; see the amendment note above and the
[`COROUTINE_ON_YIELD.md`](../../how/concepts/research/deferrable/COROUTINE_ON_YIELD.md)
hypothesis.

## Principles

1. **`emit` is the sole production keyword** — Generators produce values
   one-way via `emit` (EDR-021). There is no second keyword.

2. **Laziness by construction** — Generator expressions produce values on
   demand, never eagerly. Materialisation is explicit (`.collect()`).

3. **Automatic state** — Generator functions preserve function state
   between emits; no manual field management.

4. **Composable** — Generators combine via iterator combinators and via
   delegation (re-emitting a sub-generator's values).

5. **Minimal syntactic addition** — Generator expressions are the only new
   syntax; delegation is a composition pattern, not a keyword.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Generator Model Policy | Governs stackless (default) vs. stackful generator semantics |
| Desugaring Policy | Formalises generator expression desugaring to `emit`-based generator functions |
| Iterator Protocol Policy | Generators implement `Iterator[T]` — the production side of the protocol pair |

## Model (What)

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
#           emit x * x
```

### Generator Delegation (Composition)

Combine generators by re-emitting a sub-generator's values. Delegation is
a composition pattern — there is no dedicated keyword:

```orthon
fun combined() -> Iterator[Int]
    for v in fib(10):     # delegate to fib(10)
        emit v
    for v in fib(20):     # then to fib(20)
        emit v
```

### Relationship to `emit` (EDR-021)

`emit` (established in EDR-021) is the sole production keyword. Generator
expressions and delegation are sugar and composition over it:

| Construct | Nature |
|---|---|
| `emit value` | One-way production (EDR-021) — the canonical keyword |
| `(expr for x in src if cond)` | Sugar: anonymous `emit`-based generator function |
| `for v in sub: emit v` | Composition: delegation to a sub-generator |

## Default Strategy

Stackless generators (state machine) implementing `Iterator[T]` (EDR-021).
One-way by default — `emit` is the only production keyword. Generator
expressions compile to anonymous generator functions; delegation is
ordinary iteration plus `emit`.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Stackful generators | Separate call stack per generator; can emit from nested calls (Lua coroutines). Higher memory overhead. |
| Dedicated delegation keyword | A `yield from`-style keyword for delegation — rejected: composition (`for v in sub: emit v`) suffices. |
| Eager production | Traditional list/array building — no laziness. Breaks infinite sequences. |

## Open Questions

1. Should generator expressions support async sources (`async (x for x in stream)`)? Depends on the async model (EDR-085).
2. Does `emit` move or borrow the emitted value (ownership)?

## Decision History

- **2026-07-27** — Accepted via EDR-050. Classification: Language. Added bidirectional `yield`, generator expressions, and `yield from` delegation on top of LAZY_SEQUENCE_GENERATORS (EDR-021).
- **2026-09-06** — **Amended (S2):** bidirectional `yield`, the `yield` / `yield from` keywords, and `BidirectionalGenerator[T, U]` are withdrawn. Generators are emit-only; delegation is composition. The bidirectional form is demoted to the [`COROUTINE_ON_YIELD.md`](../../how/concepts/research/deferrable/COROUTINE_ON_YIELD.md) hypothesis ([EDR-091](../../how/decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md)).
- **2026-09-06** — **Generator-expression syntax locked:** the parenthesised form `(expr for x in src if cond)` is the single syntax; a `gen(...)` call form is rejected as a redundant synonym. The named escape hatches remain the full `fun` with `emit` and combinator chains.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `../how/IMPLEMENTATION_POLICIES.md`
