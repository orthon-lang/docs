# Fitness Functions

> Architectural fitness functions guard against decay of the core
> design. Each function is a measurable check applied when a concept
> is revised or a new concept is introduced.

Fitness Functions form an extensible catalogue of architectural
metrics. They are checked as part of the design pipeline —
alongside [Gates](../gates/DECISION_VALIDATION.md) — before a
concept is approved.

## Concept Stability

A concept that requires more than one substantive semantic change
after its initial Gate approval triggers an architectural review.
Frequent semantic revision of a single concept indicates the concept
or its surrounding abstractions are poorly factored. (See
[`what/DESIGN_PRINCIPLES.md`](../../what/DESIGN_PRINCIPLES.md) —
Transparency principle.)

## Layered Isolation

A change to one layer (Core Language, Standard Library, Implementation
Strategy) must not ripple into another layer's semantics. If a strategy
change forces a concept-level redefinition, the layer boundary is
breached.

## Composition Surface

The number of documented canonical forms for a concept must not
decrease across revisions — composition should expand, not contract.

---

*Additional fitness functions may be added here as the language evolves.*
