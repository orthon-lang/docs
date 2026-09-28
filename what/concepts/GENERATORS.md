# Generators — Generator Expressions over `emit`

> **✅ ACCEPTED — [EDR-050](../../how/decision_records/architecture/EDR-050-generators.md), [EDR-091](../../how/decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md), [EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md).**
>
> **Status:** Accepted 2026-07-27. **Amended 2026-09-06** — the
> bidirectional form is withdrawn from the language model (S2): the
> `yield` / `yield from` keywords and the `BidirectionalGenerator[T, U]`
> trait are removed. Generators are **emit-only** — one-way production.
> The withdrawn bidirectional form is preserved as a hypothesis:
> [`COROUTINE_ON_YIELD.md`](../../how/concepts/research/deferrable/COROUTINE_ON_YIELD.md).
>
> **Amended 2026-09-28 ([EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md)).**
> The generator-expression surface form is now `gen(...)`, a **reserved
> comprehension production** (not a stdlib function, not a macro) — pure sugar
> over `emit` / stdlib combinators with no new core primitive. This supersedes
> the 2026-09-06 lock of the bare parenthesised form; the previously rejected
> `gen(...)` call form is now the adopted form. `emit` does not appear inside
> lambdas / closures, closing LAZY_SEQUENCE_GENERATORS Open Question 2
> negatively. Human Sign-off: Reviewed-by: mniedre · Date: 2026-09-28 ·
> Verdict: LOCKED.
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

The generator-expression surface form is `gen(...)` — a **reserved
comprehension production** (per EDR-092). `gen` is grammar, not a stdlib
function and not a macro: the parser recognises `gen(` as the opening of a
comprehension, and its contents are comprehension clause-grammar
(`expr for x in src if cond`), never an argument expression. There are two
canonical shapes — the single form and the nested / flattening form:

```orthon
# Single form
let evens_from_sub = gen(v for v in sub)

# Nested / flattening form (multi-clause)
let flattened = gen(v for s in subs for v in s)

# Basic generator expression
let squares = gen(x * x for x in 1..10)

# With filter
let evens = gen(x for x in 1..100 if x % 2 == 0)

# With transformation
let names = gen(user.name for user in users if user.active)

# With map equivalent
let doubled = gen(x * 2 for x in items)
```

Generator expressions are lazy by default — they produce an `Iterator[T]` without materialising. As pure sugar they desugar to stdlib iterator combinators (`.filter` / `.map` / `.flat_map`) or, equivalently, to an anonymous `emit`-based generator function — introducing no new core primitive:

```orthon
# Desugaring:
let squares = gen(x * x for x in 1..10)
# → let squares = fun () -> Iterator[Int]:
#       for x in 1..10:
#           emit x * x
# → or, equivalently, to combinators:
#   let squares = (1..10).map(|x| x * x)
```

### Generator Delegation (Variant B — Re-emission IS Delegation)

There is **no separate delegation mechanism**. Re-emitting a sub-generator's
values from ordinary iteration *is* delegation. The block form and the
expression form are the **same thing** in two spellings:

```orthon
# Block form: fun ... emit, looping over the delegate
fun combined() -> Iterator[Int]
    for v in fib(10):     # delegate to fib(10)
        emit v
    for v in fib(20):     # then to fib(20)
        emit v

# Expression form: the equivalent gen(...) comprehension for a single delegate
let evens_of_sub = gen(v for v in sub)
```

**Lazy pull-through.** Delegation is fully lazy and streams one value at a
time. One consumer `next()` pulls exactly one delegate `next()`, which yields
exactly one `emit`; execution suspends at each `emit` and resumes on the next
pull. The delegate is never materialised, so delegation works on infinite
sources, and `.take(n)` short-circuits the whole pull chain after `n` values —
no delegate value beyond the `n`-th is ever produced.

**Why there is no `yield from` keyword.** The model is emit-only (EDR-021 /
EDR-091): there is nothing to "forward". Delegation falls out of ordinary
iteration + `emit` (equivalently, a nested `gen(...)` clause), so a dedicated
delegation keyword would add surface with no semantic content.

### Relationship to `emit` (EDR-021)

`emit` (established in EDR-021) is the sole production keyword. Generator
expressions and delegation are sugar and composition over it:

| Construct | Nature |
|---|---|
| `emit value` | One-way production (EDR-021) — the canonical keyword |
| `gen(expr for x in src if cond)` | Sugar: reserved comprehension production (EDR-092), desugars to combinators or an `emit`-based generator function |
| `for v in sub: emit v` | Composition: delegation to a sub-generator (equivalently `gen(v for v in sub)`) |

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

### Rejected alternatives

- **`gen(sub)` — a `gen` wrapping a bare value / iterator** rather than a
  comprehension. **Rejected** (EDR-092): it is a redundant synonym for `.iter()`
  / identity, and it breaks the invariant that `gen(` always introduces a
  comprehension. `gen` must open comprehension clause-grammar
  (`expr for x in src ...`), never wrap a plain value.
- **Bare parenthesised form** `(expr for x in src if cond)` without a `gen`
  marker. **Withdrawn** (EDR-092): no explicit production marker; visually
  indistinguishable at its opening delimiter from a parenthesised expression or
  a tuple.

## Open Questions

1. Should generator expressions support async sources (`async gen(x for x in stream)`)? Depends on the async model (EDR-085).
2. Does `emit` move or borrow the emitted value (ownership)?

## Decision History

- **2026-07-27** — Accepted via EDR-050. Classification: Language. Added bidirectional `yield`, generator expressions, and `yield from` delegation on top of LAZY_SEQUENCE_GENERATORS (EDR-021).
- **2026-09-06** — **Amended (S2):** bidirectional `yield`, the `yield` / `yield from` keywords, and `BidirectionalGenerator[T, U]` are withdrawn. Generators are emit-only; delegation is composition. The bidirectional form is demoted to the [`COROUTINE_ON_YIELD.md`](../../how/concepts/research/deferrable/COROUTINE_ON_YIELD.md) hypothesis ([EDR-091](../../how/decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md)).
- **2026-09-06** — **Generator-expression syntax locked:** the parenthesised form `(expr for x in src if cond)` is the single syntax; a `gen(...)` call form is rejected as a redundant synonym. The named escape hatches remain the full `fun` with `emit` and combinator chains. *(superseded by EDR-092, 2026-09-28)*
- **2026-09-28** — **Adopted `gen(...)` as the single generator-expression surface form** ([EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md)): `gen` is a reserved comprehension production (not a stdlib function, not a macro), pure sugar over `emit` / stdlib combinators with no new core primitive, scope = comprehension only incl. nested `gen(v for s in subs for v in s)`. The bare parenthesised form is withdrawn; `gen(sub)` (wrapping a bare value) is rejected; `emit` inside lambdas is rejected — closing LAZY_SEQUENCE_GENERATORS Open Question 2 negatively. Generators remain emit-only.

---

Governing records: [EDR-050](../../how/decision_records/architecture/EDR-050-generators.md),
[EDR-091](../../how/decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md),
[EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md).

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/LAZY_SEQUENCE_GENERATORS.md` (Open Question 2 closed by EDR-092)
- [ ] `what/SYNTAX.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `../how/IMPLEMENTATION_POLICIES.md`
