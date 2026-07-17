# Concept Template — Language Concept Document

> Use this template for every concept document in `docs/what/concepts/`.
> Each concept describes one orthogonal language construct — its motivation,
> constraints, design, and implementation strategies.
>
> Copy this file from `docs/how/templates/_concept.md` when starting a new concept.

---

# [Concept Name]

## Issue (Why)

What problem does this concept solve?

Describe the real-world problem that motivates this concept. Why is it necessary? What scenarios suffer without it?

## Principles

Which principles must not be violated?

Reference `DESIGN_PRINCIPLES.md` and other relevant documents. List the constraints that every implementation of this concept must satisfy.

## Policy Footprint

Which Policy Types does this concept involve?

| Policy Type | Role in the concept |
|---|---|
| _(e.g. Allocation Policy)_ | _(how this Policy Type relates to the concept)_ |
| _(e.g. Lifetime Policy)_ | _(second relation, if any)_ |

*The specific list is filled in during concept design and passes
verification through `IMPLEMENTATION_INDEPENDENCE_GATE`.*

## Model (What)

What does the language concept look like?

Describe the ideal model: syntax, semantics, the programmer's mental model. Include usage examples.

## Default Strategy

How is it implemented by default?

Describe the strategy the compiler or runtime chooses when the programmer does not specify an explicit strategy.

## Alternative Strategies

What alternative implementations are permitted?

List and briefly describe alternative strategies. For each, state when it should be used and what trade-offs it involves.

## Open Questions

What remains unresolved?

Questions, deferred decisions, and contentious points that require further discussion.

## Decision History

Why were alternatives rejected?

References to ADRs, logs of rejected approaches, and key trade-offs that were accepted.

---

### Affected Documents

- [ ] `CORE_CONCEPTS.md`
- [ ] `DATA_MODEL.md`
- [ ] `DESIGN_PRINCIPLES.md`
- [ ] `GLOSSARY.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] Other: ...
