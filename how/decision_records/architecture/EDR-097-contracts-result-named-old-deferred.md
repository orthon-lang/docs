# EDR-097: Contracts Amendment — Result Named in the Signature, `old` Deferred

**Status:** Proposed

**Date:** 2026-10-21

**Category:** Architecture

**Scope:** Module

**Human Sign-off:** Reviewed-by: — · Date: — · Verdict: pending

**Amends:** [EDR-056](./EDR-056-contracts.md) — the accepted concept specification [`what/concepts/CONTRACTS.md`](../../../what/concepts/CONTRACTS.md) (Model section: the `result` and `old` bullets, the evaluation section, the desugaring, and the examples).

> **Draft.** Not accepted until the Human Sign-off above is populated by the
> author. Agents and GSD flows must not self-certify that field
> ([`DECISION_PROCESS.md`](../../process/DECISION_PROCESS.md): no EDR without
> human sign-off). Not yet registered in [`INDEX.md`](../INDEX.md); registration
> happens after sign-off.

---

### Context

EDR-056 accepted contracts as a Language feature. The concept specification it
accepted describes two implicit names in postconditions: `result`, "an implicit
variable binding the function's return value", and `old`, "an implicit variable
capturing a parameter's value at function entry". The review of the CONTRACTS
concept (2026-10-18 to 2026-10-20) found problems with both.

**`result` is a name that appears without a declaration.**

- [`NAME_RESOLUTION.md`](../../architecture/NAME_RESOLUTION.md) states that all
  identifiers must be declared and that silent shadowing produces a warning;
  an implicit `result` is an exception to that rule and needs four special
  rules of its own (where it is declared, where it is visible, what its type
  is, and what happens when a parameter or local is called `result`).
- `result` is an ordinary identifier in the specification itself (at least nine
  occurrences in `what/`, for example `let mut result = 0` inside a function
  that could carry a contract), so a reserved or implicit `result` collides with
  common user code.
- Precedents differ: Dafny and Whiley name the return value in the signature,
  JML reserves `\result`, and the C++ proposal P1323 names it in the
  postcondition (checked by search, 2026-10-18).

**`old` matters only for state a callable mutates, and costs more than it
returns in v0.1.**

- Orthon is immutable by default
  ([`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) § Mutation): `fun` never
  mutates `self` and `new` returns a new value, so `old(e)` equals `e` for both.
  Only `proc` (and mutated captures or globals) needs a state before the call.
- The whole-receiver form `old.size` snapshots all of `self`; under
  copy-on-write the first mutation then clones the whole collection.
- `old` is an operator that shifts evaluation time, not a name, so the naming
  fix for `result` does not apply; making it right needs a form, a scope rule,
  a cost bound, and a tie to the effect footprint. The analysis is kept in
  [`CONTRACT_OLD_STATE.md`](../../concepts/research/deferrable/CONTRACT_OLD_STATE.md).

This amendment is also the second substantive change to the CONTRACTS concept
after acceptance (the first is the deferral of `old`, applied inline on
2026-10-19 at the author's explicit instruction). Under the Transparency
architecture metric ([`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)) it
serves as the architectural review for CONTRACTS. Whether the earlier deferral
counts toward the threshold is the author's call.

### Decision

1. **The result is named in the signature.** A function that has a postcondition
   about its result declares the name in the signature, for example
   `-> result: Float`. The name is chosen by the author of the function;
   `result` is a convention, not a reserved word.
2. **Scope.** The name is visible only inside `ensures` clauses. It is not
   visible in `requires`, in `invariant`, or in the body. A name that duplicates
   a parameter name is a duplicate-declaration error.
3. **Unnamed result.** An `ensures` clause that mentions the result of a
   function with an unnamed result is a compile error with a hint to name it in
   the signature. A function with no return value, or whose conditions do not
   mention the result, needs no name.
4. **Successful exits only.** The name denotes the value the function returns on
   a successful exit, as `ensures` applies only to successful exits. For a
   function whose declared result is an error union, the name binds the success
   value. Naming of several results follows the return-type notation decision
   ([`FUNCTION_RETURN_SYNTAX.md`](../../syntax/FUNCTION_RETURN_SYNTAX.md)) and is
   not decided here.
5. **No second way to bind the result.** A lambda binder in `ensures`
   (`ensures fn(r) -> ...`) is not introduced: it would shadow the signature
   name. Predicate-valued clauses (`ensures is_non_negative`) are not introduced
   in v0.1: an `ensures` expression that is a function value is a compile error
   with a hint to call it on the result (`is_non_negative(result)`).
6. **`old` is deferred out of v0.1.** The concept specification no longer defines
   `old`. Postconditions in v0.1 speak about the state after the call and about
   the result. Relations to the state before a mutating (`proc`) call are not
   expressible in v0.1; relations for `fun` and `new` methods are expressible
   because `self` is unchanged. The analysis and the conditions for
   reintroduction are kept in
   [`CONTRACT_OLD_STATE.md`](../../concepts/research/deferrable/CONTRACT_OLD_STATE.md).
7. **Out of scope.** Named contract objects, the conformance keyword, contracts
   through generics and function values, multi-instance contracts, the wording of
   Principle 1, and the future of `invariant` for mutable types are decided (if
   at all) through
   [`TRAIT_CONTRACT_UNIFICATION.md`](../../concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md)
   and its syntax companion, not here.

```orthon
fun sqrt(x: Float) -> result: Float
    requires x >= 0.0
    ensures result >= 0.0
    ensures abs(result * result - x) <= 0.000001 * x

fun withdraw(balance: Int, amount: Int) -> new_balance: Int
    requires amount > 0
    requires amount <= balance
    ensures new_balance == balance - amount

fun half(x: Int) -> Int
    ensures result == x / 2          // ERROR: the result is not named; write `-> result: Int`

fun f(result: Int) -> result: Int    // ERROR: duplicate declaration of `result`

fun g(x: Int) -> r: Int
    requires r > 0                   // ERROR: `r` is not visible in `requires`
    ensures is_positive              // ERROR: function value in ensures; write is_positive(r)
```

### Consequences

- **Positive:**
  - No name appears without a declaration; the four special rules for an
    implicit name disappear and ordinary name resolution applies.
  - A clash with a user identifier `result` becomes a visible, ordinary error.
  - The name sits in the signature next to its type, where a reader or an LLM
    looks first.
  - The v0.1 contract model is smaller: no time-shifting operator, no snapshot
    cost model, no tie to the effect footprint.
  - Prepares named contracts: the signature of a contract can name everything
    its clauses mention.
- **Negative:**
  - A function with postconditions on its result carries a name that only
    contracts use; it is noise where the name is not needed.
  - No relation between the state before and after a mutating `proc` call can be
    stated in v0.1 (for example, "the size grew by one"); the workaround is a
    `new` method that returns the changed value.
  - Authors and LLMs familiar with a reserved `result` (Eiffel, JML, OCL) must
    write the name themselves; the convention `result` keeps the habit.
  - Examples and the registry text must change (see Compliance).

### Compliance

1. [`what/concepts/CONTRACTS.md`](../../../what/concepts/CONTRACTS.md): the
   `result` bullet, the examples, the evaluation section and the desugaring use
   the named result; `old` is described as deferred with a link to
   `CONTRACT_OLD_STATE.md`; Decision History records this EDR and replaces the
   wording "no EDR status change" of the 2026-10-19 entry.
2. [`what/CORE_CONCEPTS.md`](../../../what/CORE_CONCEPTS.md): the CONTRACTS entry
   no longer says "`result` and `old` implicit variables"; it records the named
   result and the deferral of `old`, and carries the Open Questions and Resolved
   Questions fields per the Open Questions Tracking format.
3. [`what/GLOSSARY.md`](../../../what/GLOSSARY.md): the Contract entry example
   uses the named result.
4. [`INDEX.md`](../INDEX.md) registers EDR-097 with "Amends EDR-056" after
   sign-off; [`DECISION_LOG.md`](../../gates/DECISION_LOG.md) receives an entry
   with the reasoning of the affected gates.
5. Check: no accepted document describes an implicit `result` or an available
   `old` (a search of `what/` for "implicit variable" and for `old(` finds only
   the deferral pointers).

Inline (Tier 4) clarifications made together with this amendment, not decided
by it: contracts apply to any function or method and `self` exists only in
methods; contracts of trait methods are inherited by implementations (weaken
`requires`, strengthen `ensures`); violation reports name the failed predicate;
named pure predicates are reused across clauses; higher-order contracts and the
external imposition of rules are recorded as deferred.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep the implicit `result` as accepted in EDR-056 | A name without a declaration; collides with the common identifier `result`; needs four special name-resolution rules |
| Sigil-prefixed implicit name (`\result` as in JML) | Avoids the clash but the name is still undeclared; adds a symbol, against Named Before Symbolic |
| Name per clause through a lambda binder (`ensures fn(r) -> ...`) | Repeats the binder in every clause; shadows the signature name when both exist; a second way to introduce the same name |
| Predicate-valued clauses (`ensures is_non_negative`) | Implicit application by type; covers only conditions on the result alone; asymmetric with `requires`. Deferred, not rejected; can return as sugar |
| Keep `old` as the whole-receiver `old.size` (EDR-056 today) | A name from nowhere; copy-on-write clone cost for the first mutation; matters only for `proc` |
| `old(expr)` operator or named entry bindings in v0.1 | **Deferred, not rejected:** the forms and restrictions are recorded in `CONTRACT_OLD_STATE.md` for reintroduction |

### Gate Validation

Prose and a short table, per the Post-Acceptance Gaps (Type B) rule that only
the affected gates are re-run
([`CONCEPT_PIPELINE.md`](../../CONCEPT_PIPELINE.md) § 10). The Decision Pipeline
is not re-run for an accepted concept. The set below is proposed; the author
confirms it at sign-off. Verdicts are **Pending**: no gate has been run for this
draft.

| Gate | Method | Verdict | What the run has to show |
|------|--------|---------|--------------------------|
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pending | Terms defined (named result, scope); counterexamples: parameter named `result`, `ensures` without a named result, error-union result, function value in `ensures`, use in `requires` or in the body |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pending | The named result needs no special name-resolution rule; removing `old` removes a construct and its cost model |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pending | An LLM places the result name in the signature and uses it in `ensures` at least as reliably as with an implicit `result`; the extra token in the signature is acceptable |

Gates not proposed: `USER_VALUE_GATE` (no new capability is introduced; one is
withdrawn and deferred), `ARCHITECTURAL_INTEGRITY_GATE` and
`IMPLEMENTATION_INDEPENDENCE_GATE` (the change stays inside the contract
signature and uses the existing name-resolution rules), and
`LONG_TERM_MAINTAINABILITY_GATE` (covered by the traceable deferral record).
Confirm or change this selection at sign-off.

---

> **Human Sign-off:** `Reviewed-by: — · Date: — · Verdict: pending`
> Required before this EDR may be accepted and registered. Agents and GSD flows
> must not self-certify this field.
