# Ownership Transfer

> **Status:** Open research hypothesis. This document does not amend an accepted EDR.
> **Parent map:** [Ownership](OWNERSHIP.md). **Version:** v0.1 decisions are open unless identified as deferred.

## Accepted Baseline and Scope

[EDR-013](../../../decision_records/architecture/EDR-013-semantic-model.md) adopts ownership, move, shared XOR exclusive borrowing, and scope-based lifetime. [EDR-030](../../../decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) requires compile-time ownership analysis. The [semantic model](../../../../what/SEMANTIC_MODEL.md) requires exclusive access for `proc`. [EDR-094](../../../decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md) specifies borrowed iteration and prohibits structural mutation during a live iterator borrow. These are constraints, not approval of the candidates below.

## Problem and Boundary

Define when responsibility transfers, the validity of the former binding, and transfer across calls and containers. The exact operator belongs to [ownership transfer syntax research](OWNERSHIP_TRANSFER_OPERATOR.md) and the syntax pipeline. Copyable data and independently owned copies must remain distinct from resource moves.

## Alternatives

1. Require an explicit transfer operation at every existing resource binding.
2. Use signature-driven transfer with an explicit marker only at selected sites.
3. Require explicit transfer except for freshly produced values, subject to a specified freshness rule.

All candidates must respect EDR-013's visible semantic transfer; whether candidate 2 satisfies it requires review. No syntax is selected here.

## Discriminating Scenarios

| Scenario | Expected observation | Status |
|---|---|---|
| Transfer a resource and read its old binding | Reject the read; the recipient owns the resource | Covered by scenario |
| Copy an independent regular value, mutate original | Copy retains its original value | Covered by scenario |
| Transfer a resource while a view into its storage remains active | Reject if transfer invalidates the view or validity cannot be proved | Covered; stable-storage exceptions open |
| Move a resource into a container then exit scope | Exactly one responsible owner follows the selected lifecycle rules | Cleanup order and partial moves open |

## v0.1 Decisions and Dependencies

Specify transfer at assignment, arguments, return, fresh values, branches, partially moved values, and reinitialisation. Determine treatment of copyable values and diagnostics. Depends on type categories; coordinates with borrowing and resource lifecycle.

## Review and Routing

This is a research input, not a completed Decision Pipeline, Design Review, validation gate, or human sign-off. Classify each question using the [post-acceptance gap procedure](../../../CONCEPT_PIPELINE.md#10-post-acceptance-gaps): Type B for an unresolved accepted-concept detail, Type C for a demonstrated cross-concept conflict, or Type A for a missing concept. Apply the relevant review and affected gates before recording any resolution. Nyquist coverage above identifies discriminating documented scenarios, not executable tests or proof of completeness.
