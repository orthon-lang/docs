# Ownership Type Categories and Composition

> **Status:** Open research hypothesis. This document does not amend an accepted EDR.
> **Parent map:** [Ownership](OWNERSHIP.md). **Version:** v0.1 decisions are open unless identified as deferred.

## Accepted Baseline and Scope

[EDR-013](../../../decision_records/architecture/EDR-013-semantic-model.md) adopts ownership, move, shared XOR exclusive borrowing, and scope-based lifetime. [EDR-030](../../../decision_records/architecture/EDR-030-compiler-as-static-analyzer.md) requires compile-time ownership analysis. The [semantic model](../../../../what/SEMANTIC_MODEL.md) requires exclusive access for `proc`. [EDR-094](../../../decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md) specifies borrowed iteration and prohibits structural mutation during a live iterator borrow. These are constraints, not approval of the candidates below.

## Problem and Boundary

Determine how a declared type's copyability and responsibility interact with fields, generic parameters, and containers. This research does not define move syntax or cleanup protocols. Working terminology: regular types are copyable; resource types are move-only; `Resource`, `ResourceContainer`, and `Copyable` are candidate markers, with explicit `Resource` declaration. Their interaction is not accepted specification.

## Alternatives

1. Explicitly declare each container's classification; reject incompatible fields.
2. Derive a container's capabilities from its fields and generic arguments; require an explicit resource declaration where appropriate.
3. Use conditional generic capabilities with explicit declarations and compiler-checked constraints.

Compare declaration burden, predictable public contracts, soundness, and generic usability. No alternative is selected.

## Discriminating Scenarios

| Scenario | Expected observation | Status |
|---|---|---|
| Copy a regular record of copyable fields | Independent values; later mutation does not change the copy | Baseline; covered by scenario |
| Copy a record containing a move-only file | Reject unless an independently justified copy operation exists; never implicitly duplicate responsibility | Safety requirement; propagation rule open |
| Instantiate `Box<T>` with copyable and move-only `T` | Classification and allowed operations must follow the chosen generic rule | Uncovered: candidate-specific outcomes required |
| Explicit marker contradicts field capabilities | Reject an unsound declaration; decide how missing declarations are diagnosed | Partially covered |

## v0.1 Decisions and Dependencies

Define marker meaning and compiler recognition, user-defined types, copy versus explicit duplication, generic capability propagation, and invalid declarations. Determine whether `ResourceContainer` is essential or redundant. Depends on accepted ownership and type/trait rules; informs transfer, borrowing and lifecycle.

## Review and Routing

This is a research input, not a completed Decision Pipeline, Design Review, validation gate, or human sign-off. Classify each question using the [post-acceptance gap procedure](../../../CONCEPT_PIPELINE.md#10-post-acceptance-gaps): Type B for an unresolved accepted-concept detail, Type C for a demonstrated cross-concept conflict, or Type A for a missing concept. Apply the relevant review and affected gates before recording any resolution. Nyquist coverage above identifies discriminating documented scenarios, not executable tests or proof of completeness.
