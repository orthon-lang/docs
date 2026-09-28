# EDR-092: Generator Expression Syntax — gen(...) as a Reserved Comprehension Production

**Status:** Accepted

**Date:** 2026-09-28

**Category:** Architecture

**Scope:** Subsystem

**Human Sign-off:** Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED

**Partially supersedes:** [EDR-050](./EDR-050-generators.md) (the parenthesised generator-expression syntax lock) and [EDR-091](./EDR-091-withdraw-bidirectional-yield.md) (decision item 3 — the prior rejection of the `gen(...)` call form).

---

### Context

EDR-050 (2026-07-27) introduced generator expressions and locked their surface
form to a bare parenthesised comprehension — `(expr for x in src if cond)` — and
EDR-091 (2026-09-06, item 3) reaffirmed that single form while explicitly
rejecting a `gen(...)` call form as "a redundant synonym". A subsequent
concept-level review of GENERATORS found that the bare parenthesised form carries
no explicit production marker: a reader (human or LLM) must recognise a generator
expression purely from the interior `for` clause, and the form is visually
indistinguishable at its opening delimiter from an ordinary parenthesised
expression or a tuple. The review concluded that an explicit, reserved leading
marker improves both learnability and LLM generability without adding any new
semantics, since the construct remains pure sugar over already-accepted
constructs (EDR-021 `emit`, EDR-032 iterator combinators).

The forces at play: the emit-only production model (EDR-021 / EDR-091) is fixed
and must not be reopened; the minimal-core principle forbids introducing a new
core primitive for what is sugar; and the review required that the new marker be
grammar the parser recognises directly, not a library identifier or a macro that
would blur the language/library boundary. This decision records how the surface
form is fixed — the emit-only semantics beneath it are unchanged.

### Decision

1. **Single surface form `gen(...)`.** Generator expressions use the single
   surface form `gen(...)`. `gen` is a **reserved surface grammar production** —
   not an identifier available to user code. The bare parenthesised form is
   withdrawn.

2. **`gen` is NOT a stdlib function.** The content between the parentheses is
   comprehension clause-grammar (the `expr for x in src if cond` clause form),
   not a value or argument expression. `gen(...)` cannot be assigned to a
   variable named `gen`, passed as a higher-order value, or shadowed; the parser
   recognises `gen(` as the opening of a comprehension, never as a call.

3. **`gen` is NOT a macro.** AST macros (EDR-029) receive a typed AST and expand
   within the existing grammar; `gen` instead introduces new surface grammar that
   the parser recognises directly. It is therefore a language production, not a
   macro expansion.

4. **Scope is the comprehension form only.** `gen(...)` covers the comprehension
   form exclusively, including nested / multi-clause forms such as the flattening
   comprehension `gen(v for s in subs for v in s)`. There is no other use of the
   `gen` production.

5. **Pure sugar (Desugaring Policy) — no new core primitive.** `gen(...)`
   desugars to stdlib iterator combinators (`.filter` / `.map` / `.flat_map`) or
   to an equivalent `emit`-based `fun`. It introduces no new core primitive and
   no new runtime behaviour.

   ```orthon
   # Surface form
   let squares = gen(x * x for x in 1..10)

   # Desugars to an emit-based generator function ...
   let squares = fun () -> Iterator<Int>:
       for x in 1..10:
           emit x * x

   # ... or, equivalently, to stdlib combinators
   let squares = (1..10).map(|x| x * x)
   ```

6. **Block / emit form unchanged.** The block form remains the named
   `fun ... emit`. `gen(...)` is the expression spelling of the same emit-only
   production; the two are interchangeable.

7. **`emit` does not appear inside lambdas / closures.** This **closes
   [`what/concepts/LAZY_SEQUENCE_GENERATORS.md`](../../../what/concepts/LAZY_SEQUENCE_GENERATORS.md)
   Open Question 2** ("Should generators support `emit` from within nested
   closures?") **negatively.** `gen(...)` is a comprehension production, not a
   closure that captures an ambient `emit` sink; there is no construct through
   which `emit` could escape into a lambda body.

8. **Generators remain emit-only.** One-way production per EDR-021 / EDR-091 — no
   `yield`, no `yield from`. Delegation is ordinary iteration plus `emit`
   (equivalently, a nested `gen(...)` clause).

### Consequences

- **Positive:**
  - Every generator expression now opens with an explicit, reserved marker
    (`gen(`), so the construct is unambiguous to readers and highly generable by
    LLMs — the opening token names the intent.
  - No new core primitive and no new semantics: the change is confined to the
    surface grammar; the emit-only model (EDR-021 / EDR-091) is untouched.
  - The language/library boundary stays crisp: `gen` is grammar, not a stdlib
    identifier and not a macro, so it cannot be shadowed, imported, or
    redefined.
  - Open Question 2 of LAZY_SEQUENCE_GENERATORS is resolved, removing a lingering
    ambiguity about `emit` inside closures.
- **Negative:**
  - `gen` becomes a reserved word, unavailable as a user identifier.
  - Documents and examples that used the bare parenthesised form must be updated
    to the `gen(...)` spelling (anti-memory of the old form is retained in
    Decision History).

### Compliance

1. `what/concepts/GENERATORS.md` presents `gen(...)` as the single
   generator-expression surface form (bare parenthesised form removed), with the
   Variant-B delegation model and the `gen(sub)` rejection.
2. `what/concepts/LAZY_SEQUENCE_GENERATORS.md` Open Question 2 is annotated
   resolved negatively by this EDR (anti-memory preserved).
3. `what/CORE_CONCEPTS.md` GENERATORS summary shows `gen(x * x for x in 1..10)`
   and references this EDR.
4. No bare parenthesised generator expression and no `gen(sub)` form appears as a
   language construct in accepted (`what/`) documents.

### Alternatives Considered

> Populated from the concept-review discussion (2026-09-28).

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| `gen(sub)` — a `gen` wrapping a bare value / iterator rather than a comprehension | Redundant synonym for `.iter()` / identity, and it breaks the invariant that `gen(` always introduces a comprehension. `gen` must open clause-grammar, never wrap a value. |
| The bare parenthesised form `(expr for x in src if cond)` without a `gen` marker | No explicit production marker: indistinguishable at its opening delimiter from a parenthesised expression or tuple, and weakly generable. This is the portion of EDR-050 / EDR-091 (item 3) being reversed. |
| `gen` as a stdlib function | Its parentheses would then contain an argument expression, not comprehension clause-grammar; a `for`-clause is not a value, so `gen` cannot be an ordinary callable. Would also make `gen` shadowable. |
| `gen` as an AST macro (EDR-029) | Macros expand within the existing grammar over a typed AST; the comprehension clause form is new surface grammar the parser must recognise directly, which is a language production, not a macro. |

### Gate Validation

Resolved through the concept-review discussion (2026-09-28) and human-signed-off
as LOCKED. The `gen(...)` production is pure sugar over already-accepted
constructs (EDR-021 `emit`, EDR-032 combinators), so it adds no new semantics to
validate; the review confirmed the production against the emit-only model
(EDR-021 / EDR-091) and against the minimal-core and Desugaring principles — no
new core primitive is introduced, and the language/library boundary is preserved
(grammar, not a stdlib function or macro). Per the EDR-091 precedent, a full
seven-gate table is not reproduced for a locked sugar decision over settled
constructs.
