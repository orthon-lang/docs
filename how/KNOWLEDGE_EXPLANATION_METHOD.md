# Knowledge Explanation Method

> Status: working method for preparing future learning materials. Other
> methods, including lighter ones, may be selected when appropriate.

## Purpose

Turn a complex topic into an explanation that gives the reader a usable
understanding while preserving the conditions that make it correct. Use
the [Learning Model](LEARNING_MODEL.md) to identify the objective and
prerequisite skills; use this method to prepare and present the knowledge.

## Prepare: Crystallize the Knowledge

1. **Triangulate.** Gather distinct examples and relevant perspectives.
   First explore differences; then identify candidate common principles.
   Check a contrasting case or counterexample and the canonical sources.
   Agreement among examples alone does not prove a universal rule.
2. **Remove what is not needed.** Preserve the principle, its conditions
   and limits, relevant anti-patterns, and useful idioms. Move details
   that do not serve the learning objective to optional material.
3. **Build a causal model.** Explain how a condition leads through a
   mechanism to a consequence. Check that each link is supported; mark
   uncertainty and avoid presenting correlations as established causes.

The preparation may be abbreviated for a narrow, well-established topic.
The author need not expose every research step to the reader.

## Present: Build Understanding

1. **Situation.** Introduce a concrete problem familiar to the reader.
2. **Action.** Show a small example of what the concept lets the reader do.
3. **Contract and cause.** State the relevant guarantees and obligations,
   and explain the causal chain needed to understand them.
4. **Application and limits.** Compare another case, show a limitation or
   anti-pattern, and introduce implementation details when the reader's
   objective requires them.

Adapt the depth and order to the reader's prior knowledge. A contract can
hide implementation details, but it must not hide conditions required for
correct use. Invite readers to compare examples and articulate an invariant
when that supports the learning objective.

## Example: Optional Results

A lookup may find no matching record. An optional result makes that
possibility part of the interface; a caller handles both presence and
absence. The causal chain is: absence is possible → the return contract
represents it → the caller must account for it. Compare a missing record
with a missing configuration value to show the common principle. Explain
separately that absence and an operation failure are different concerns.

This example describes an explanation strategy. Language guarantees and
concrete examples must be checked against their canonical sources, such as
[the source-linked theses](../what/THESES.md), before publication.

## Choosing a Method

Use this method when the topic warrants comparison, boundary analysis,
and a causal explanation. A short definition or focused example may be
enough for a narrower objective. Evaluate the method on actual readers;
replace or supplement it if its cost or complexity exceeds its benefit.
