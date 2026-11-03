# Borrow Scopes and Returned Views

> **Status:** Open research hypothesis. This document does not amend an accepted EDR.
> **Parent map:** [Ownership](OWNERSHIP.md). **Version:** v0.1 decisions are open unless identified as deferred.

## Accepted Baseline and Scope

[EDR-013](../../../decision_records/architecture/EDR-013-semantic-model.md) adopts ownership, move, shared XOR exclusive borrowing, and scope-based lifetime. [EDR-030](../../../decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) requires compile-time ownership analysis. The [semantic model](../../../../what/SEMANTIC_MODEL.md) requires exclusive access for `proc`. [EDR-094](../../../decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md) specifies borrowed iteration and prohibits structural mutation during a live iterator borrow. These are constraints, not approval of the candidates below.

## Problem and Boundary

Determine how scope-bounded views cross function boundaries and how the compiler connects a result, including aggregates and iterators, to its possible owners. Working decisions in [Ownership](OWNERSHIP.md) permit returned borrows with static proof, set explicitly bounded lexical borrow scopes, and infer an unambiguous relationship. Public result types are declared; precise full syntax is open. `-> buffer` is a non-public candidate deferred beyond v0.1.

## Alternatives

1. Infer a single unambiguous source; require an explicit relationship for ambiguity.
2. Infer all relationships from function bodies and expose the resulting contract in interfaces.
3. Require an explicit relationship for all returned borrows.

Candidate 1 follows the working inference decision. Candidates 2 and 3 remain comparisons and would require revisiting that decision where they conflict. Full public result types remain explicit in every candidate.

## Discriminating Scenarios

| Scenario | Expected observation | Status |
|---|---|---|
| Return a view of a caller-owned input | Accept only if the relation and both scope and access obligations are provable | Covered by scenario |
| Return a view of a function-local buffer | Reject: the view would outlive the owner | Covered by scenario |
| Choose a view from two inputs using a runtime condition | Track all possible owners or reject when the contract cannot be proved | Covered; representation open |
| Return iterator or closure that captures a borrow | Preserve dependency through the returned aggregate; prevent escape and conflicting mutation | Covered; type representation open |
| Use a view outside its explicit scope or move its owner while it invalidates the view | Reject | Covered by scenario |

## v0.1 Decisions and Dependencies

Specify unambiguous inference, explicit ambiguous contracts, permitted scopes, nested reborrows, aggregate and iterator dependencies, error paths, and cross-module checking. Determine essential interface notation via the syntax pipeline; conveniences are post-v0.1. Depends on borrowing and transfer; coordinates with iterator protocol and FFI.

## Review and Routing

This is a research input, not a completed Decision Pipeline, Design Review, validation gate, or human sign-off. Classify each question using the [post-acceptance gap procedure](../../../CONCEPT_PIPELINE.md#10-post-acceptance-gaps): Type B for an unresolved accepted-concept detail, Type C for a demonstrated cross-concept conflict, or Type A for a missing concept. Apply the relevant review and affected gates before recording any resolution. Nyquist coverage above identifies discriminating documented scenarios, not executable tests or proof of completeness.
