# Execution Contracts (Hypothesis H4)

> **Status:** research/unverified.
> **Layer:** How — concept research, not accepted Orthon semantics.
> **Date:** 2026-11-17.
> **Source:** Author discussion, [Tail recursion in Lisp / Scheme](chatgpt-conversation://6afc34a5-cae8-83ed-a3fa-d5d7287d3e8b).
> **Tier:** important — candidate capability; core necessity remains unproved.
> No acceptance decision, human sign-off, or EDR is recorded here.

## Problem

A stack requirement describes an execution property independently of the technique implementing it. Investigate expressing it alongside existing behavioral contracts and effect declarations without conflating their responsibilities.

## Examples and Hypothesis

Pseudocode only; all forms below are provisional, not canonical Orthon syntax.

```text
count(n, acc) -> result
    requires n >= 0
    ensures result == acc + n
    effects none
    guarantees stack: constant
```

## Alternatives to Investigate

| Candidate category | Research meaning |
|---|---|
| `requires` | Caller obligations before entry |
| `ensures` | Relations over successful exit state and result |
| `effects` | Permitted observable actions or computational effects |
| `guarantees` | Properties maintained during execution, including resource bounds |

The accepted Contracts model already provides `requires` and `ensures`. A separate `guarantees` category and a unified declaration system remain hypotheses. `effects none` illustrates a category, not an accepted effect vocabulary; reconcile it with Effect Footprint before adding a declaration mechanism.

Compare a dedicated `@tailrec`, inference only, `ensures stack: constant`, and `guarantees stack: constant`. A stack bound concerns execution throughout the call, rather than an ordinary exit predicate. Tail position is a structural property, not itself a side effect.

Could `@tailrec` be sugar for a scoped stack guarantee? Equivalence is unverified: loops or trampolines might satisfy the guarantee without recursion, while eliminated self-recursion does not bound arbitrary non-tail callees. This record does not change accepted contract-check elision or settle Trait/Contract Unification.

## Implications for Orthon

Separate the required property from analysis and implementation. Review
[Design Principles](../../../DESIGN_PRINCIPLES.md), especially Semantics
Before Optimization and Explicit Optimization. Automatic analysis alone
neither authorizes an implicit optimization nor establishes a resource bound.

## Classical PL Problems

H4 intersects **Hoare logic and design by contract** (`requires`/`ensures`),
**effect systems** (`effects`), and **resource-aware or refinement types**
(`guarantees`). It raises familiar questions about soundness, composition,
substitutability, higher-order functions, separate compilation, and the
difference between a statically proved property and a runtime check. A stack
bound is a temporal resource invariant, so it does not naturally behave like a
postcondition evaluated only at return. It must also state what happens when a
callee, cleanup action, or contract check falls outside the claimed scope.

## Open Questions

1. Is constant stack relative to input size, recursion depth, or a named chain? Does it include callees, contract checks, and runtime helpers?
2. Do guarantees enter function types, higher-order bounds, and substitutability, or remain separate declarations?
3. How do guarantees compose through unknown callees, dynamic dispatch, and separate compilation?
4. Must unproved promises be rejected, or can an explicit runtime mechanism enforce them? How are unsupported and violated promises distinguished?
5. Can enforcement survive debug/release and Strategy changes without inheriting ordinary runtime-contract elision?
6. Can postconditions themselves retain frames and invalidate an otherwise tail-recursive stack bound?

## Verification Plan

Define the resource, scope, assumptions, and permitted enforcement outcomes first. Compare loops, self-recursion, mutual recursion, and bounded callers with unbounded non-tail callees. Check higher-order substitution and effect composition against existing models. Compare annotation, inference, and guarantee clauses for diagnostics and duplicate concepts. Silent non-enforcement refutes a checked guarantee. Termination, bounded heap, and total memory remain independent claims.

External comparison targets from the discussion: Scheme tail-call semantics,
Kotlin's `tailrec`, and Python/Java recursive-versus-iterative examples.
Consult primary specifications and separate language guarantees from
implementation behavior before using these comparisons as evidence.
They have not been independently verified in this capture.

## Decision Pipeline — Preliminary Triage

Initial application of the [Decision Pipeline](../../../process/DECISION_PIPELINE.md):

| Question | Research assessment |
|---|---|
| 1. Problem | Defined above; establish practical need with verification cases |
| 2. Language or library? | Compare semantic guarantees with library/state-machine composition |
| 3. Existing primitives? | Investigate calls, iteration, contracts, and contexts first |
| 4. Principle conflict? | Review implicit optimization and duplicate declarations |
| 5. Semantics or sugar? | Required resource bounds add obligations; spelling may be sugar |
| 6. Composition? | Demonstrate or refute sufficiency of existing mechanisms |
| 7. Sugar? | Annotation/contract equivalence remains unverified |
| 8. Optimization or semantics? | Best-effort lowering differs from a required bound |
| 9. Compatibility? | Review contracts, effects, calls, and Strategy independence |
| 10. Worth adding? | Unverified; inclusion awaits evidence and review |

## Related Hypotheses and Existing Models

- [H1 — Tail Recursion Elimination (TRE)](TAIL_RECURSION_ELIMINATION.md)
- [H2 — Proper Tail Calls (PTC)](PROPER_TAIL_CALLS.md)
- [H3 — Control Transfer](CONTROL_TRANSFER.md)
- [H4 — Execution Contracts](EXECUTION_CONTRACTS.md)

H1 concerns self-recursion; H2 covers arbitrary tail calls. H3 asks whether
a shared mechanism helps either. H4 investigates declaring and checking
required properties. None requires acceptance of the others: H1 may suffice,
H2 may stand independently, and H3 may remain internal or prove unnecessary.

## Reasoning Summary

The contract vocabulary should describe the promised observable property, not
the compiler technique used to obtain it. `@tailrec` is narrower and structural:
it can mean that a particular recursive shape is checked. `guarantees stack:
constant` is broader and resource-oriented: it could be met by TRE, PTC, a
loop, or a state machine. `requires`, `ensures`, `effects`, and `guarantees`
should therefore remain candidates for separate responsibilities until scope,
composition, and enforcement are demonstrated. This hypothesis extends the
accepted Contracts research surface; it does not revise that concept.

- [Accepted Contracts](../../../../what/concepts/CONTRACTS.md)
- [Effect Footprint hypothesis](../essential/EFFECT_FOOTPRINT.md)
- [Trait/Contract Unification hypothesis](../essential/TRAIT_CONTRACT_UNIFICATION.md)
- [Execution Context Invocation](../essential/EXECUTION_CONTEXT_INVOCATION.md)
- [Execution Model](../../../../what/EXECUTION_MODEL.md)
- [Optimization Model](../../../../what/OPTIMIZATION_MODEL.md)
- [Syntax Decision Queue](../../../syntax/README.md#decision-queue)

## Gate Evidence Still Required

These are pending checks under the
[Language Design Gate](../../../gates/_language-design.md), not passed criteria:

- [ ] Establish necessity and semantic-layer classification; prefer sufficient composition.
- [ ] Define precise scope and interactions with evaluation, ownership, cleanup, failures, and contracts.
- [ ] Verify portability and distinguish semantic obligations from Policy choices.
- [ ] Resolve syntax through the Syntax Pipeline; no canonical form is accepted here.
- [ ] Review counterexamples, costs, and new contradictions before human sign-off and any acceptance EDR.
