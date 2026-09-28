# Generator Expression Syntax

> **Accepted — EDR-092 (Generator Expression Syntax).** Explicit syntax
> record created so the accepted `gen(...)` surface form has a home in
> `what/syntax/`. The canonical semantic specification lives in
> [`what/concepts/GENERATORS.md`](../concepts/GENERATORS.md); this file
> records the surface form. The Syntax Pipeline reasoning trail is
> [`how/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](../../how/syntax/GENERATOR_EXPRESSION_SYNTAX.md).

## Canonical forms

```orthon
gen(x * x for x in 1..10)              # single form: an Iterator<Int> of squares
gen(x for x in 1..100 if x % 2 == 0)   # filtered form
gen(v for v in sub)                    # single-delegate form (re-emit a sub-generator)
gen(v for s in subs for v in s)        # nested / flattening form (multi-clause)
```

All four are the one construct — a lazy `Iterator<T>` produced without
materialising. The block spelling is the named equivalent:

```orthon
let squares = gen(x * x for x in 1..10)
# equivalently, the named fun ... emit form:
let squares = fun () -> Iterator<Int>:
    for x in 1..10:
        emit x * x
```

## Rules

1. **Single surface form `gen(...)`.** Generator expressions use exactly one
   surface form. `gen` is a **reserved surface grammar production**, not an
   identifier: the parser recognises `gen(` as opening a comprehension, never
   as a call. `gen` cannot be assigned, shadowed, imported, or passed as a
   value (EDR-092 items 1–2).
2. **The parentheses hold clause-grammar, not an argument.** The contents are
   comprehension clauses (`expr for pattern in src if cond`), not an argument
   expression — a `for`-clause is not a value (EDR-092 item 2).
3. **Scope is the comprehension form only**, including nested / multi-clause
   forms such as the flattening `gen(v for s in subs for v in s)`. There is no
   other use of the `gen` production (EDR-092 item 4).
4. **Pure sugar (Desugaring Policy).** `gen(...)` desugars to stdlib iterator
   combinators (`.filter` / `.map` / `.flat_map`) or to an equivalent
   `emit`-based `fun`, introducing **no new core primitive** and no new runtime
   behaviour (EDR-092 item 5).
5. **Generators remain emit-only.** No `yield`, no `yield from` (EDR-021 /
   EDR-091). Delegation is ordinary iteration + `emit`, equivalently a nested
   `gen(...)` clause (EDR-092 item 8).
6. **Rejected forms.** `gen(sub)` wrapping a bare value is rejected — a
   redundant synonym for `.iter()` / identity that breaks the "`gen(` always
   opens a comprehension" invariant; and the marker-less bare parenthesised
   form `(expr for x in src if cond)` is withdrawn. Two further surface
   variants were considered and rejected: a source-first, lambda-style
   ordering `for x in src if cond -> expr` (overloads `->`; its data-flow
   reading is already served by combinator chains; result-first is stronger
   for LLM generability) and an unwrapped infix marker
   `expr gen x in src if cond` (a comprehension is not a binary operator, so
   an infix `gen` needs global precedence and context-dependent `in` / `if`
   scoping and cannot bound multi-clause flattening). See EDR-092
   Alternatives Considered for the full rationale.
7. **`emit` never appears inside lambdas / closures.** `gen(...)` is a
   comprehension production, not a closure that captures an ambient `emit`
   sink — this closes
   [`LAZY_SEQUENCE_GENERATORS.md`](../concepts/LAZY_SEQUENCE_GENERATORS.md)
   Open Question 2 negatively (EDR-092 item 7).

This record implements EDR-092.

## Cross-References

- [EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md) — deciding record.
- [`what/concepts/GENERATORS.md`](../concepts/GENERATORS.md) — full semantic specification.
- [`what/concepts/LAZY_SEQUENCE_GENERATORS.md`](../concepts/LAZY_SEQUENCE_GENERATORS.md) — Open Question 2 (closed negatively by EDR-092).
- [`what/SYNTAX.md`](../SYNTAX.md) — hub.
- [`how/syntax/README.md`](../../how/syntax/README.md) — decision queue (resolved).
- [`how/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](../../how/syntax/GENERATOR_EXPRESSION_SYNTAX.md) — reasoning trail (pipeline Stages 1–6b).
