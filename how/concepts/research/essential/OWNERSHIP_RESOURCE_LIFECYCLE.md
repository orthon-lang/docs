# Resource Lifecycle and Management

> **Status:** Open research hypothesis. This document does not amend an accepted EDR.
> **Parent map:** [Ownership](OWNERSHIP.md). **Version:** v0.1 decisions are open unless identified as deferred.

## Accepted Baseline and Scope

[EDR-013](../../../decision_records/architecture/EDR-013-semantic-model.md) adopts ownership, move, shared XOR exclusive borrowing, and scope-based lifetime. [EDR-030](../../../decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) requires compile-time ownership analysis. The [semantic model](../../../../what/SEMANTIC_MODEL.md) requires exclusive access for `proc`. [EDR-094](../../../decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md) specifies borrowed iteration and prohibits structural mutation during a live iterator borrow. These are constraints, not approval of the candidates below.

## Problem and Boundary

Define responsibility for acquiring, moving and releasing resources, including scope exit and failure paths. [EDR-085](../../../decision_records/architecture/EDR-085-execution-context-invocation.md) describes `using` through context, scope and destruction; whether every `Resource` requires such a context is not settled by this hypothesis. Do not infer a separate execution context for each resource.

## Alternatives

1. Ownership-driven deterministic cleanup, with an optional managed scope.
2. Require an explicit managed scope for resources needing cleanup, with defined ownership transfer.
3. Use explicit resource managers and compiler-checked lifetime contracts, without requiring one context per resource.

Compare leaks, transfer across scopes, early exits, fallible cleanup and interaction with accepted EDR-085. Candidates that alter accepted semantics require amendment review.

## Discriminating Scenarios

| Scenario | Expected observation | Status |
|---|---|---|
| Leave managed scope normally or on an error path | Release occurs according to the specified cleanup contract; no unintended duplicate release | Cleanup and error-order semantics open |
| Move a resource to a longer-lived owner | The original scope must not release the transferred resource; the new owner assumes responsibility | Covered by scenario |
| Keep a borrowed view after its resource is released | Reject unless validity is independently established | Covered by scenario |
| Foreign code retains an untracked pointer or mutates during a shared borrow | Safe interface must prevent or exclude this by a verified/trusted contract; conflicting safe call is rejected | FFI contract and trust boundary open |
| Resource stored in a container | Exactly one responsible cleanup path; field order and partial moves require a rule | Coverage partial |

## v0.1 Decisions and Dependencies

Specify resource protocols, destruction and ordering, explicit close, partial construction, error propagation, move versus managed-scope cleanup, and foreign ownership contracts. Compare with EDR-085 and the accepted declarative resource model. Depends on categories, transfer and borrow scopes.

## Review and Routing

This is a research input, not a completed Decision Pipeline, Design Review, validation gate, or human sign-off. Classify each question using the [post-acceptance gap procedure](../../../CONCEPT_PIPELINE.md#10-post-acceptance-gaps): Type B for an unresolved accepted-concept detail, Type C for a demonstrated cross-concept conflict, or Type A for a missing concept. Apply the relevant review and affected gates before recording any resolution. Nyquist coverage above identifies discriminating documented scenarios, not executable tests or proof of completeness.
