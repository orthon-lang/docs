# Borrowing and Exclusive Access

> **Status:** Open research hypothesis. This document does not amend an accepted EDR.
> **Parent map:** [Ownership](OWNERSHIP.md). **Version:** v0.1 decisions are open unless identified as deferred.

## Accepted Baseline and Scope

[EDR-013](../../../decision_records/architecture/EDR-013-semantic-model.md) adopts ownership, move, shared XOR exclusive borrowing, and scope-based lifetime. [EDR-030](../../../decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) requires compile-time ownership analysis. The [semantic model](../../../../what/SEMANTIC_MODEL.md) requires exclusive access for `proc`. [EDR-094](../../../decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md) specifies borrowed iteration and prohibits structural mutation during a live iterator borrow. These are constraints, not approval of the candidates below.

## Problem and Boundary

Specify temporary access and the proof obligations for `proc`, without conflating access with ownership transfer. Scope syntax and returned-view contracts are addressed separately in [borrow scopes](OWNERSHIP_BORROW_SCOPES.md).

## Alternatives

1. Conservative whole-owner access checking.
2. Field-sensitive checking where disjoint access is statically provable.
3. Whole-owner checking initially with explicitly designed, proven splitting operations.

Every candidate enforces shared XOR exclusive access; precision, supported APIs, and implementation complexity differ. No candidate is selected.

## Discriminating Scenarios

| Scenario | Expected observation | Status |
|---|---|---|
| Two shared reads in one scope | Permit both when no exclusive access conflicts | Covered by scenario |
| Shared view active, call `proc` on the same data | Reject conflicting mutation until the explicit borrow scope ends | Covered by scenario |
| Two exclusive accesses to overlapping data | Reject the second conflicting access | Covered by scenario |
| Distinct fields used for reading and writing | Sound acceptance depends on provable disjointness; conservative rejection is safe | Coverage partial; precision open |
| `for` borrows collection; body removes element | Reject structural mutation during the live iterator borrow per EDR-094 | Covered by scenario |

## v0.1 Decisions and Dependencies

Define how access is granted, checked, reborrowed, suspended and restored; owner access while borrowed; field overlap; `proc` argument aliasing; diagnostics and precise borrow scopes. Depends on transfer and type categories; coordinates with iteration and FFI.

## Review and Routing

This is a research input, not a completed Decision Pipeline, Design Review, validation gate, or human sign-off. Classify each question using the [post-acceptance gap procedure](../../../CONCEPT_PIPELINE.md#10-post-acceptance-gaps): Type B for an unresolved accepted-concept detail, Type C for a demonstrated cross-concept conflict, or Type A for a missing concept. Apply the relevant review and affected gates before recording any resolution. Nyquist coverage above identifies discriminating documented scenarios, not executable tests or proof of completeness.
