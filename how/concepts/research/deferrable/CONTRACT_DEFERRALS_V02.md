# Contract Unification — Items Deferred to v0.2

> **⚠️ HYPOTHESIS REGISTER — deferred to v0.2 or later, not an accepted concept.**
> Register of the items that [EDR-098](../../../decision_records/architecture/EDR-098-trait-contract-unification.md)
> (Proposed) and the hypothesis [`TRAIT_CONTRACT_UNIFICATION.md`](../essential/TRAIT_CONTRACT_UNIFICATION.md)
> defer out of v0.1. The analysis stays where it was written; this file keeps one
> list with the reason and the trigger for reopening each item, so nothing is lost.
> Home: `how/concepts/research/deferrable/` — deferred to v0.2/v0.3.
>
> **Last updated:** 2026-10-22
>
> **Related:** [`CONTRACT_OLD_STATE.md`](CONTRACT_OLD_STATE.md) (the deferral of `old`, kept separately),
> [`../../../syntax/CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md),
> [`../../../syntax/FUNCTION_RETURNING_FUNCTION.md`](../../../syntax/FUNCTION_RETURNING_FUNCTION.md)

---

## Problem

The unification of traits and function contracts is decided for v0.1 only where
it can be defined without function-type notation, lambdas and generics over
function values. The rest is real design work that must not be forgotten and must
not be decided ad hoc inside the v0.1 text.

## Deferred items

| ID | Item | Why it waits | Decision | Analysis lives in |
|----|------|--------------|----------|-------------------|
| D-1 | Function contracts through generics and function values: a function contract as a generic bound (`<Sqrt as F>`), `dyn C` on function values, collections of contract-bearing functions, and generic functions used as values | The notation for function types is undecided (`->` is overloaded); the relation between the unique type of a function and the structural function type (coercion or erasure) is unexamined | Review decision 15; Open Questions 1, 7, 8 | Hypothesis, "Deferred to v0.2" examples and Open Questions |
| D-2 | `obeys` on a lambda, and whether a lambda may be accepted into `dyn C` without declaring it; mutating callables (captured state) that need a non-`fun` callable member | Lambdas are deferred; the kind of a callable member derives from captures, which is the effect-footprint model (hypothesis) | Review decisions 15, 19; Open Question 12 | Hypothesis, Open Question 12; `EFFECT_FOOTPRINT.md` |
| D-3 | Multi-instance contracts (provisional name *law*): properties over several values or calls, for example transitivity of `Eq` and the round trip of `Serialize` | Needs a name (the candidates collide or are heavy), a checking story (property-based generation) and tool support | Review decision 17; Open Questions 3, 4, 14 | Hypothesis, the deferred subsection of Model |
| D-4 | Imposing a rule from outside on a foreign function, and a rule registry file; the form of a reference to a rule in a comment | Shifts blame because the foreign function promises nothing; needs a doc-comment concept | Review decision 13; Open Questions 13, 16 | Hypothesis, Open Question 13; Racket `contract-in` as the precedent |
| D-5 | A contract that attaches conditions to a position, whatever the member name (a conditions-only contract), with its own distinct spelling | The short form is matched by call signature and a conditions-only header is an error in v0.1; named predicates cover reuse until a real need appears | Review decision 24 | EDR-098 Decision item 5, "Forms (24)" |

`old` (state at entry in postconditions) is deferred separately; see
[`CONTRACT_OLD_STATE.md`](CONTRACT_OLD_STATE.md).

## Triggers for reopening

| Item | Reopen when |
|------|-------------|
| D-1, D-2 | the function-type notation is accepted (`FUNCTION_RETURNING_FUNCTION.md`) and lambdas have an accepted syntax |
| D-3 | a property such as the Transitivity Invariant has to be checked rather than stated in prose, and `ORTHON-TEST` can generate inputs |
| D-4 | a rule has to be attached to a function the author does not own, and doc comments are part of the language |
| D-5 | named predicates prove insufficient for reuse across members with different names |

## Open Questions

1. Which of D-1 and D-2 should be decided together with lambda syntax, and which
   can precede it?
2. Does D-3 need its own keyword, or can a multi-instance property be a contract
   member with free parameters?

## Decision History

| Date | Event |
|------|-------|
| 2026-10-22 | Register created from EDR-098 (Proposed), as gate resolution R7 |
