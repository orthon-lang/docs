# EDR-093: Generator Expression — Single-Clause Only

**Status:** Accepted

**Date:** 2026-09-28

**Category:** Architecture

**Scope:** Subsystem

**Human Sign-off:** Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED

**Amends:** [EDR-092](./EDR-092-generator-expression-syntax.md) (decision item 4 — the nested / multi-clause scope).

---

### Context

EDR-092 (2026-09-28) adopted `gen(...)` as a reserved comprehension production
and its decision item 4 scoped `gen` to the comprehension form "including
nested / multi-clause forms" such as the flattening comprehension
`gen(v for s in subs for v in s)`. A re-review narrows that scope. A
multi-clause comprehension bundles two orthogonal operations — iteration and
flattening — into a single surface form, whereas a single-clause comprehension
plus named iterator combinators keeps each operation explicit and
independently composable. Coupling iteration with flattening inside one `gen`
also enlarges the grammar and the parse-ambiguity surface for no expressive
gain the combinator family does not already provide.

Only the clause count is narrowed. The emit-only production model
(EDR-021 / [EDR-091](./EDR-091-withdraw-bidirectional-yield.md)) and the
reserved `gen(` surface marker
([EDR-092](./EDR-092-generator-expression-syntax.md)) are unchanged — `gen(`
still opens exactly one comprehension, and the construct remains pure sugar
with no new core primitive.

### Decision

1. **Single-clause only.** A `gen(...)` generator expression has exactly one
   `for` clause plus zero or more `if` filters —
   `gen(expr for x in src if cond ...)`.
2. **Multi-clause withdrawn.** Multiple `for` clauses in one `gen`
   (nested / flatten / cartesian) are WITHDRAWN from v0.1.
3. **Generator source stays allowed.** A `gen` whose SOURCE is itself a `gen`
   remains allowed: `gen(x for x in gen(...))` — single-clause with a
   generator source.
4. **Flatten / cartesian route to iterator combinators**
   ([EDR-022](./EDR-022-iterator-protocol.md)). This EDR introduces the
   `.flatten()` StdLib combinator — a one-level flatten,
   `Iterator<Iterator<T>> -> Iterator<T>` — and routes pure flatten to it.
   Map-then-flatten (dependent inner iterator / cartesian product) routes to
   `.flat_map(...)`, which is unchanged and NOT renamed — `flat_map` keeps its
   name. No combinator is renamed.
5. **Still pure sugar.** `gen(...)` remains pure sugar over `emit` and iterator
   combinators; the emit-only model (EDR-021 / EDR-091) and the reserved `gen(`
   marker (EDR-092) are unchanged. No new core primitive is introduced.
6. **Future extension (v1.x): multi-clause.** Widening single-clause to
   multi-clause in v1.x is purely additive and breaks no v0.1 code.

```orthon
# Allowed: single `for` clause plus zero or more `if` filters
let evens = gen(x for x in 1..100 if x % 2 == 0)

# Allowed: single-clause with a generator source
let relayed = gen(x for x in gen(y for y in 1..10 if y > 3))

# Flatten / cartesian are NOT written as multi-clause gen in v0.1:
# pure flatten via .flatten(); cartesian / dependent inner via .flat_map(...)
let flat = subs.flatten()
let pairs = xs.flat_map(|x| ys.map(|y| (x, y)))
```

### Consequences

- **Positive:**
  - One `gen` = one iteration plus optional filters — orthogonality is
    restored; iteration and flattening are no longer bundled in one form.
  - Flatten and cartesian are expressed explicitly through named combinators
    (`.flatten()` / `.flat_map(...)`), so the operation is visible at the call
    site.
  - Smaller, unambiguous grammar; the multi-clause parse-ambiguity surface is
    removed.
  - Higher LLM generability — one clause per `gen` is a simpler, less
    error-prone shape to generate.
  - Forward-compatible: the v1.x widening to multi-clause is purely additive
    and breaks no v0.1 code.
- **Negative:**
  - Nested flattening loses its comprehension spelling in v0.1 and must be
    written with `.flatten()` / `.flat_map(...)` or a nested-source `gen`.

### Compliance

1. [`what/concepts/GENERATORS.md`](../../../what/concepts/GENERATORS.md)
   presents generator expressions as single-clause (one `for` + optional
   `if`s), keeps the nested-source `gen(x for x in gen(...))` form, routes pure
   flatten to `.flatten()` and dependent / cartesian to `.flat_map(...)`, and
   carries an EDR-093 amendment banner plus a Decision-History bullet.
2. [`what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](../../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md)
   narrows the canonical forms and Rules to single-clause with a
   "multi-clause deferred to v1.x" note.
3. [`how/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](../../syntax/GENERATOR_EXPRESSION_SYNTAX.md)
   carries an appended single-clause amendment note in its reasoning trail.
4. [`how/architecture/PARSER.md`](../../architecture/PARSER.md) tightens the
   `GeneratorExpr` grammar to one `ForClause` plus zero-or-more `IfClause`.
5. [`what/concepts/ITERATOR_PROTOCOL.md`](../../../what/concepts/ITERATOR_PROTOCOL.md)
   documents the `.flatten()` combinator introduced by this EDR.
6. [`INDEX.md`](../INDEX.md) registers EDR-093 in both tables.
7. [`EDR-092`](./EDR-092-generator-expression-syntax.md) carries an
   "Amended by EDR-093" back-pointer banner.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep the multi-clause comprehension in v0.1 (EDR-092 item 4) | **Deferred to v1.x, NOT rejected:** it couples iteration with flattening, the iterator combinators (`.flatten()` / `.flat_map()`, [EDR-022](./EDR-022-iterator-protocol.md)) already express it, and widening single-clause to multi-clause later is purely additive. Recorded as Future extension (v1.x). |
| Add / rename a comprehension keyword to spell flatten | Unnecessary surface: the iterator-combinator family ([EDR-022](./EDR-022-iterator-protocol.md)) already covers flatten and cartesian. `flat_map` is not renamed, and `.flatten()` is added as an ordinary StdLib combinator rather than as new comprehension grammar. |

### Gate Validation

Prose only, per the EDR-091 / EDR-092 precedent — a locked amendment over
settled sugar does not reproduce the full seven-gate table. Single-clause is a
strict narrowing of an already gate-passed sugar: `gen(` still opens exactly
one comprehension, so the EDR-092 Coupling & Overload "one symbol → one
meaning" result is unaffected; the construct remains pure sugar over `emit` and
iterator combinators (no new core primitive); and the emit-only model
(EDR-021 / EDR-091) is untouched. The newly introduced `.flatten()` is an
ordinary StdLib combinator on `Iterator<T>` (EDR-022), not new language
surface, so it adds no core semantics to validate.
