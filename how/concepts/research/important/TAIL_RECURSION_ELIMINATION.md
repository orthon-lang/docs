# Tail Recursion Elimination (TRE) (Hypothesis H1)

> **Status:** research/unverified.
> **Layer:** How — concept research, not accepted Orthon semantics.
> **Date:** 2026-11-17.
> **Source:** Author discussion, [Tail recursion in Lisp / Scheme](chatgpt-conversation://6afc34a5-cae8-83ed-a3fa-d5d7287d3e8b).
> **Tier:** important — candidate capability; core necessity remains unproved.
> No acceptance decision, human sign-off, or EDR is recorded here.

## Problem

Deep self-recursion may exhaust the stack even when the recursive result is returned directly. Investigate lowering eligible tail self-calls to iteration while preserving evaluation order, ownership, and failure behavior.

## Examples and Hypothesis

Pseudocode only; all forms below are provisional, not canonical Orthon syntax.

```text
count(n, acc):
    if n == 0: return acc
    return count(n - 1, acc + 1)
```

## Alternatives to Investigate

Compare automatic detection with best-effort optimization, `@tailrec` as an optimization request, `@tailrec` as a checked contract, and `guarantees stack: constant` as a resource requirement. Detection, permission to optimize, and enforcement of a guarantee are separate responsibilities. No candidate is selected.

Under the checked-contract candidate, inability to establish the promised property would produce a compile-time diagnostic, including in debug builds. The policy-request candidate may offer a weaker promise. Determine which interpretation is appropriate before choosing notation.

## Implications for Orthon

Separate the required property from analysis and implementation. Review
[Design Principles](../../../DESIGN_PRINCIPLES.md), especially Semantics
Before Optimization and Explicit Optimization. Automatic analysis alone
neither authorizes an implicit optimization nor establishes a resource bound.

## Classical PL Problems

This hypothesis is a concrete instance of several established programming
language problems:

- **Tail-position analysis:** the compiler must determine whether a call's
  continuation is exactly the caller's continuation, including branches,
  handlers, cleanup, and postcondition checks.
- **Operational equivalence:** replacing `call` with a loop must preserve
  evaluation order, failures, ownership events, and observable effects.
- **Space safety:** the intended claim concerns the control stack, not
  termination, heap growth, or total memory consumption.
- **Separate compilation:** a local transformation may be unable to prove
  anything about an indirect or foreign call.
- **Debugging observability:** removing frames changes stack traces unless
  logical call history is represented separately.

These are research dimensions, not resolved requirements for Orthon.

## Open Questions

1. Which recursive branches must be in tail position, and are ordinary exit branches allowed?
2. Does pending cleanup, a handler, or a postcondition prevent frame elimination even after a syntactic return?
3. Is checking a compiler obligation, an optimization/execution Policy, or language semantics?
4. Does a guarantee cover the self-recursive chain only, or also argument evaluation and non-tail callees?
5. How are indirect calls, dynamic dispatch, and unsupported targets diagnosed?

## Verification Plan

Compare recursive and iterative traces, evaluating new arguments before replacing parameters. Include non-tail recursion, multiple branches, pending cleanup, ownership transfers, and failures as negative cases. Specify depth-scaling stack measurements for debug/release and each Strategy using fixed-size counters. Any changed evaluation/cleanup order refutes the transformation; stack growth refutes the claimed bound within its defined scope.

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

The discussion distinguishes three decisions that are often conflated:
detecting a tail call, allowing an optimization, and promising bounded stack
use. H1 can be useful even if H2 is rejected: a compiler may lower direct
self-recursion while the language retains ordinary call semantics. Conversely,
`@tailrec` is not automatically equivalent to a general `guarantees` clause,
because a resource guarantee may be satisfied by a loop or state machine and
may cover more than self-recursion. The next step is evidence about semantic
equivalence and scope, not a choice of spelling.

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
