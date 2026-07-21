# Fitness Functions

> **Last updated:** 2026-07-20
>
> Architectural fitness functions guard against decay of the core
> design. Each function is a measurable check applied when a concept
> is revised or a new concept is introduced.

Fitness Functions form an extensible catalogue of architectural
metrics. They are checked as part of the design pipeline —
alongside [Gates](../gates/DECISION_VALIDATION.md) — before a
concept is approved.

---

## Concept Stability

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Transparency principle

A concept that requires more than one substantive semantic change
after its initial Gate approval triggers an architectural review.
Frequent semantic revision of a single concept indicates the concept
or its surrounding abstractions are poorly factored.

**Measured by:** Count of substantive semantic changes per concept
after Gate approval. Threshold: >1 change triggers review.

---

## Layered Isolation

**Source:** [EDR-010](../decision_records/architecture/EDR-010-layered-architecture.md) — Layered Architecture

A change to one layer (Core Language, Standard Library, Implementation
Strategy) must not ripple into another layer's semantics. If a strategy
change forces a concept-level redefinition, the layer boundary is
breached.

**Measured by:** Impact analysis in each EDR: does the change require
modifications in multiple layers? If yes, layer boundary is breached.

---

## Composition Surface

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Orthogonality principle

The number of documented canonical forms for a concept must not
decrease across revisions — composition should expand, not contract.

**Measured by:** Diff of canonical forms count between revisions.
Negative diff → composition decay.

---

## Data First

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Data First principle

Every new concept must operate on Data as the primary abstraction.
No concept may introduce behavior without a Data representation.

**Measured by:** Concept design review: verify every new construct
has an explicit Data input and Data output. A construct that operates
without Data (e.g., a purely side-effectful statement) must be
explicitly justified.

---

## Orthogonality

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Orthogonality principle

No two concepts overlap in responsibility. The composability matrix
must have no forbidden pairs — every pair of concepts must compose
meaningfully.

**Measured by:** Composability matrix review at each concept addition.
If concept A and concept B cannot be used together, the matrix shows
a forbidden pair, triggering an architectural review.

---

## Explicitness

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Explicitness / Explicit Semantics principles

Semantic changes must be syntactically visible. No hidden conversions,
implicit side-effects, or invisible ownership transfers.

**Measured by:** Syntax audit: for each semantic change (allocation,
mutability, ownership transfer), verify a syntactic marker exists.
Hidden semantics → fitness failure.

---

## Minimal Core

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Minimal Core principle; [EDR-010](../decision_records/architecture/EDR-010-layered-architecture.md)

Core Language changes must be justified against composition alternatives.
A new core concept must demonstrate that it cannot be expressed through
composition of existing primitives.

**Measured by:** Core Change Criterion check: for any proposal affecting
the Core, document why composition of existing primitives is insufficient.
If the justification is absent or weak, the proposal is rejected.

---

## Named Before Symbolic

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Named Before Symbolic principle

Every symbolic operator must have an equivalent named function.

**Measured by:** Operator inventory: for each symbolic operator in the
language specification, verify a named function equivalent exists in
the Standard Library or Core.

---

## LLM Generability

**Source:** [EDR-011](../decision_records/architecture/EDR-011-llm-generability-gate.md); [\_language-design.md](../gates/_language-design.md) — LLM generability criterion

Every construct must be expressible in a machine-readable schema with no
ambiguities that an LLM could misinterpret. The Static Analyser must be
able to detect LLM-generation errors for this construct.

**Measured by:** Schema completeness check: does the construct have a
grammar rule, type signature, and stdlib contract? Ambiguity audit: are
there any syntactic or semantic ambiguities that could lead an LLM to
generate incorrect code?

---

## Policy Independence

**Source:** [EDR-010](../decision_records/architecture/EDR-010-layered-architecture.md); [IMPLEMENTATION_POLICIES.md](../IMPLEMENTATION_POLICIES.md)

Core concepts must not depend on specific Policy values. A concept's
semantics must be valid under any valid Policy assignment.

**Measured by:** Policy footprint review: for each concept, verify
that its semantics are defined without reference to specific Policy
values (e.g., "values have managed lifetimes" is valid; "values use
reference counting" is not).

---

## Cross-Reference Validity

**Source:** [CONVENTIONS.md](../../.planning/codebase/CONVENTIONS.md) — Cross-reference conventions; [AGENTS.md](../../AGENTS.md) §10.8

All "See also" links between documents must resolve to existing files.

**Measured by:** Automated link checker run at each phase boundary.
Broken links are documented and fixed before phase completion.

---

## Document Freshness

**Source:** [DOCUMENTATION_PRINCIPLES.md](../DOCUMENTATION_PRINCIPLES.md) — Date-Stamp Requirement

Documents with `> **Last updated:**` more than 90 days in the past
trigger a freshness review.

**Measured by:** Date-stamp audit: compare each document's last_updated
date to current date. Documents exceeding 90 days are flagged for review
at the next phase boundary.

---

## Deterministic Behavior

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Deterministic Behavior principle

The same source code must produce the same observable behavior across
optimization levels and implementations.

**Measured by:** Cross-strategy test: compile and run the same program
under two different Implementation Strategies. Output must be identical.
If behavior differs, the concept or strategy violates determinism.

---

## Representation Symmetry

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Representation Symmetry principle

Reversible transformations of representations must use symmetric syntax
(prefix to create, postfix to restore).

**Measured by:** Symmetry audit: for each documented representation
transformation, verify that a symmetric inverse form exists. Asymmetric
transformations are flagged.

---

## Traceability

**Source:** [DESIGN_PRINCIPLES.md](../DESIGN_PRINCIPLES.md) — Transparency principle; [\_language-design.md](../gates/_language-design.md) — Traceability criterion

Every design decision must be traceable from origin through alternatives
to final decision, recorded in an EDR or gate entry.

**Measured by:** EDR coverage: every concept document with a Decision
History section must reference an EDR. Concepts without traceable
decisions are flagged.

---

*This catalogue is extensible. New fitness functions may be added as the language evolves, following the same pattern: source, description, and measurement criteria.*
