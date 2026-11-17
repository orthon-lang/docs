# Proper Tail Calls (PTC) (Hypothesis H2)

> **Status:** research/unverified.
> **Layer:** How — concept research, not accepted Orthon semantics.
> **Date:** 2026-11-17.
> **Source:** Author discussion, [Tail recursion in Lisp / Scheme](chatgpt-conversation://6afc34a5-cae8-83ed-a3fa-d5d7287d3e8b).
> **Tier:** important — candidate capability; core necessity remains unproved.
> No acceptance decision, human sign-off, or EDR is recorded here.

## Problem

Self-recursion elimination does not cover mutual recursion or tail calls to unrelated functions. Investigate a guarantee that eligible arbitrary tail-call chains do not accumulate return frames.

## Examples and Hypothesis

Pseudocode only; all forms below are provisional, not canonical Orthon syntax.

```text
even(n): if n == 0: return true;  return odd(n - 1)
odd(n):  if n == 0: return false; return even(n - 1)
forward(x): return target(x)
```

## Alternatives to Investigate

Compare universal language semantics, an explicit checked contract, an opt-in execution Policy, and best-effort optimization. These provide different promises. H1 alone remains a valid narrower outcome.

Investigate native tail transfers, state-machine lowering, and trampolines as mechanisms, including allocation and dispatch costs. Bounded tail-chain storage does not imply termination, bounded total memory, or constant stack within arbitrary non-tail callees.

## Implications for Orthon

Separate the required property from analysis and implementation. Review
[Design Principles](../../../DESIGN_PRINCIPLES.md), especially Semantics
Before Optimization and Explicit Optimization. Automatic analysis alone
neither authorizes an implicit optimization nor establishes a resource bound.

## Classical PL Problems

PTC connects to the classical problems of **continuation semantics**,
**calling conventions**, and **bounded-space execution**. A proper tail call
must preserve the caller's continuation without retaining a redundant return
frame. This interacts with first-class continuations, exceptions, dynamic
dispatch, foreign-function interfaces, stack inspection, and separate
compilation. The phrase "constant stack" therefore needs a defined boundary:
native stack only, or also runtime frames, contract checks, and heap-allocated
continuation state. None of these boundaries is selected by this hypothesis.

## Open Questions

1. What are tail contexts when cleanup, result conversion, handlers, and postconditions remain pending?
2. Can the guarantee cover mutual recursion, higher-order calls, closures, generics, and dynamic dispatch?
3. How do separate compilation, calling conventions, foreign boundaries, and unsupported targets constrain it?
4. Can useful logical stack traces be preserved without unbounded hidden storage?
5. Is the guarantee language-wide or explicitly scoped by an execution contract?

## Verification Plan

Specify long mutual-recursive and unrelated tail-call chains, with indirect targets and changing argument layouts. Compare cleanup, failures, and return conversion with ordinary call traces. Prepare a target/Strategy feasibility matrix with fallback costs. A silently lost guarantee refutes the universal candidate. Measure retained heap continuations as well as native stack; moving an unbounded return chain to the heap does not establish proper space behavior.

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

PTC is broader than TRE. TRE can be a local lowering for `f -> f`; PTC is a
semantic property of arbitrary eligible `f -> g` chains, including mutual
recursion. That wider promise explains why mainstream languages often leave it
as an optimization or omit it: it reaches into ABI, cleanup, debugging, and
runtime design. Scheme demonstrates that the guarantee is possible when it is
chosen as a language foundation, but Scheme's choice is evidence for a design
option, not an argument that Orthon must adopt it.

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
