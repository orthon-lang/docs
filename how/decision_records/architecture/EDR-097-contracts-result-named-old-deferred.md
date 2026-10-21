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
confirms it at sign-off. The verdicts come from the run of 2026-10-21 made by
the drafting agent (analysis only, no LLM sampling); the author confirms or
overrides them at sign-off.

| Gate | Method | Verdict | Summary |
|------|--------|---------|---------|
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | **Flag** | The core rules hold; four resolvable gaps (R1–R4): "successful exit" is undefined for error results, a body name equal to the result name, result names across a trait and its implementation, no general same-scope duplicate rule |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | **Pass** | No keyword added; an implicit name and a construct (`old`) are removed; one optional name slot in the return position (notation provisional, R5) |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | **Flag** | Structurally clean and diagnosable; the share of correct generations is not measured, and a habitual implicit `result` or `old` from other languages is an anticipated systematic error (R6, R7) |

Gates not proposed: `USER_VALUE_GATE` (no new capability is introduced; one is
withdrawn and deferred), `ARCHITECTURAL_INTEGRITY_GATE` and
`IMPLEMENTATION_INDEPENDENCE_GATE` (the change stays inside the contract
signature and uses the existing name-resolution rules), and
`LONG_TERM_MAINTAINABILITY_GATE` (covered by the traceable deferral record).
Confirm or change this selection at sign-off.

#### Run of 2026-10-21

**1. `LOGICAL_CONSISTENCY_GATE` — Socratic Method — Flag**

*Terms.* Named result: the identifier introduced after `->` in a signature.
Scope: `ensures` clauses only. Successful exit: **not defined** by the concept
(`CONTRACTS.md` says "every successful return" without defining it).
`old`: deferred, no longer a term of v0.1.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| All terms precisely defined | Flag | "Successful exit" is ambiguous for an error-union result: is returning an error value a successful return? Decision 4 settles what the name binds, but the term itself needs a definition (R1) |
| No self-referential paradoxes | Pass | The rule does not invalidate itself; the result name is declared by the same mechanism that declares parameters |
| Behaviour deterministic across contexts | Flag | With `-> result: Int` and `let mut result = 0` in the body, the same identifier means the returned value in `ensures` and an unrelated local in the body. The scope rule makes this deterministic, but it can astonish a reader (R2) |
| Composition with existing concepts defined | Flag | Trait methods: names across a trait and its implementation are unspecified (R3). `NAME_RESOLUTION.md` states a shadowing rule but no same-scope duplicate rule, so Decision 2 rests only on the precedent of duplicate type parameters in `GENERICS.md` (R4). Several results are deferred to the return-notation decision |

*Counterexamples tried.* A parameter named `result` (error by Decision 2,
subject to R4); `ensures` on an unnamed result (error with a hint, Decision 3);
an error-union result (binds the success value); a function value in `ensures`
(error with a hint, Decision 5); use in `requires` or in the body (name not in
scope); `old` written in a postcondition (undefined name in v0.1, diagnostic
is R6). *Devil's advocate:* the signature now carries a name that exists only
for contracts, so two functions with the same behaviour differ in signature
text. That is a real cost, recorded in Consequences (Negative), not a
contradiction.

**2. `CONCEPTUAL_SIMPLICITY_GATE` — Scientific Method — Pass**

*Hypothesis.* Naming the result in the signature is the simplest way to give
postconditions access to the returned value, and removing `old` is the simplest
v0.1 model. *Known.* Parameters are declared in the signature and referenced by
name; name resolution exists. *Unknown.* The final notation; how often
contracts need the result. *Experiment.* The canonical examples (`sqrt`,
`withdraw`, `clamp`) rewritten with named results: each signature gains one
name, and the four special rules for an implicit name and the cost model of
`old` disappear. The experiment is shorter than the proposal. *Simpler
alternatives checked.* An implicit `result` is shorter to write but needs a
reserved or implicit name and four rules; predicate-valued clauses are shorter
in one case and cover only conditions on the result alone.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Expressible through composition | Pass | No existing construct lets `ensures` refer to the returned value |
| Single responsibility | Pass | One responsibility (access to the returned value); the second (state at entry) is withdrawn |
| No keyword or syntax bloat | Pass | No keyword added; an implicit name and a construct removed; one optional name slot in the return position |
| Minimal learning surface | Pass | Learnable in isolation: "name the result like a parameter" |

The notation `-> name: Type` is provisional (R5).

**3. `LLM_GENERABILITY_GATE` — Empirical Analysis — Flag**

*Structural scan.*

| Sub-construct | Result |
|---------------|--------|
| Named result slot `-> name: Type` | Clean: the colon after the identifier separates it from `-> Type`; relies on the colon of annotations, whose form is still open (`TYPE_ANNOTATION_SYNTAX.md`) |
| Scope rule (`ensures` only) | Clean; violations are compile errors with a hint |
| Error-union result binds the success value | One edge: a model may expect the whole error union |
| `old` deferred | Risk: models trained on Eiffel, JML, Dafny or OCL may write `old(...)`; a dedicated diagnostic is needed (R6) |
| Several results | Not generable until the notation is decided |

*Schema round-trip (grammar fragment).*

```text
FunctionDecl   := 'fun' Name Params '->' ResultSlot ContractClause* Body
ResultSlot     := [ Name ':' ] Type
ContractClause := 'requires' Expr | 'ensures' Expr      ; Name is bound in the Expr of 'ensures' only
```

The syntax round-trips fully. The scope rule and the duplicate rule must be
added to the name-resolution schema (R4).

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Schema-serializable | Pass | The grammar fragment above plus one scope rule |
| Predictable generation | Flag | **Not measured** (no sampling was run). One anticipated systematic error: a habitual implicit `result` or `old` from other languages (R7) |
| No hallucination surface | Pass | Each form has one meaning; the colon separates the named slot from the plain type |
| Strategy-aware default | Pass | Contract semantics and naming do not depend on the Implementation Strategy; enforcement stays a Contract Enforcement Policy matter |
| Self-correctable | Flag | Diagnostics are specified for an unnamed result, a scope violation, a duplicate, and a function value in `ensures`; none is specified yet for `old` (R6) |

**Overall.** No Fail: two gates Flag, one Pass. A Flag without a Fail may
proceed; the root causes are recorded and are to be resolved before the final
gate record (`EMPIRICAL_ANALYSIS_METHOD.md` § 4, Gate Flow).

**Resolutions proposed** (the author decides at sign-off; none is applied to the
Decision above):

| # | Resolution | Closes |
|---|------------|--------|
| R1 | Define "successful exit" as a normal return with a non-error value; for an error-union result, the success case; termination through a system error is not an exit | Logical Consistency: terms |
| R2 | A body declaration with the same name as the named result is a warning (the author may choose error or allow) | Logical Consistency: determinism |
| R3 | Result names need not match between a trait method and its implementation; inherited clauses use the name of the declaring signature | Logical Consistency: composition |
| R4 | State the general same-scope duplicate-declaration rule in `NAME_RESOLUTION.md`, citing the `GENERICS.md` precedent, and add the scope rule of the named result there | Logical Consistency: composition; schema |
| R5 | Mark the notation `-> name: Type` provisional, following `TYPE_ANNOTATION_SYNTAX.md` and `FUNCTION_RETURN_SYNTAX.md`; the notation for several results stays open | Simplicity note; LLM structural scan |
| R6 | Specify a dedicated diagnostic for `old` ("deferred out of v0.1"), with the two workarounds: an absolute condition, or a `new` method returning the changed value | LLM: self-correctable |
| R7 | Before the final gate record, run a measurement: a fixed set of generation tasks over the named-result form, success threshold 90 percent (the gate's own threshold), recording the failures that come from a habitual implicit `result` or `old` | LLM: predictable generation |

---

> **Human Sign-off:** `Reviewed-by: — · Date: — · Verdict: pending`
> Required before this EDR may be accepted and registered. Agents and GSD flows
> must not self-certify this field.
