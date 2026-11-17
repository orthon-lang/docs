# Control Transfer (Hypothesis H3)

> **Status:** research/unverified.
> **Layer:** How — concept research, not accepted Orthon semantics.
> **Date:** 2026-11-17.
> **Source:** Author discussion, [Tail recursion in Lisp / Scheme](chatgpt-conversation://6afc34a5-cae8-83ed-a3fa-d5d7287d3e8b).
> **Tier:** important — candidate capability; core necessity remains unproved.
> No acceptance decision, human sign-off, or EDR is recorded here.

## Problem

TRE and PTC may share a control-flow operation. Investigate whether an independent control-transfer mechanism simplifies their model or adds an unnecessary primitive.

## Examples and Hypothesis

Pseudocode only; all forms below are provisional, not canonical Orthon syntax.

```text
Conceptual IR only:
transfer target(arguments, inherited_continuation)
```

## Alternatives to Investigate

Compare composition of existing call/control-flow operations, an internal IR operation, and a user-visible primitive. A transfer without retaining the caller's return frame might explain both H1 and H2. No user keyword is proposed as accepted syntax.

Investigate continuations and selected concurrency transitions as possible extensions, not established equivalences. Tail transfer alone does not provide continuation capture/resumption, scheduling, actor isolation, ownership transfer, or message semantics. Existing Execution Context Invocation remains authoritative.

## Implications for Orthon

Separate the required property from analysis and implementation. Review
[Design Principles](../../../DESIGN_PRINCIPLES.md), especially Semantics
Before Optimization and Explicit Optimization. Automatic analysis alone
neither authorizes an implicit optimization nor establishes a resource bound.

## Classical PL Problems

This hypothesis touches **continuation-passing style (CPS)**, abstract
machines such as CEK-style evaluators, first-class continuations, coroutines,
exceptions, and actor-style message transfer. A control-transfer primitive must
specify what happens to the current continuation, live bindings, resource
cleanup, handlers, and ownership. It must also avoid confusing a local jump
with scheduling, isolation, cancellation, or communication. These boundary
questions are the classic reason that control transfer is usually modelled in
the operational semantics or IR before it is exposed as user syntax.

## Open Questions

1. Is this needed in language semantics, only in IR, or at neither level?
2. What happens to the current continuation, local resources, references, and error handlers?
3. Are first-class continuations needed, and would they be one-shot or multi-shot?
4. Can any concurrency application preserve context ownership, cancellation, and result obligations?
5. Can H1/H2 emerge from existing operations without exposing backend constraints?

## Verification Plan

Describe operational traces for call/return, self-transfer, mutual transfer, and cleanup boundaries. Compare primitive counts and proof obligations against existing composition. Seek escaping references, duplicate destruction, lost handlers, and double resumption. For each concurrency claim, map scheduling, isolation, cancellation, and result handling separately. Failure of that unification need not invalidate an IR-only mechanism.

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

H3 is the possible common denominator behind H1 and H2, but it is not yet a
justification for a new language primitive. If existing call semantics plus IR
lowering explain the needed behavior, exposing `transfer` would violate
minimality. If mutual tail calls, continuations, or execution contexts require
a shared continuation model, an IR-level mechanism may still be valuable even
when no user-facing keyword is accepted. The research question is therefore
where the abstraction belongs, not whether a powerful primitive can be named.

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
