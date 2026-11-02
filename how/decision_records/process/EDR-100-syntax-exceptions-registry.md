# EDR-100: Syntax Exceptions Registry

**Status:** Accepted

**Date:** 2026-11-02

**Category:** Process

**Scope:** Project

---

### Context

The Manifesto and Syntax Principles require one syntactic element to have one
responsibility. Existing decisions nevertheless contain bounded mirror
exceptions, unresolved collisions, and rejected reuses. Without a single
register, a reviewer can miss a prior exception or accept a new
context-dependent role as an undocumented convention. The result is silent
dilution of the syntax principle and design debt that is hard to audit.

The Syntax Pipeline already has a Coupling & Overload Check. It needs a
durable input and output for the exceptions that the check finds; creating a
new validation gate would duplicate existing process machinery.

### Decision

1. Establish [`how/SYNTAX_EXCEPTIONS.md`](../../SYNTAX_EXCEPTIONS.md) as the
   **Syntax Exceptions Registry**. It is a How-layer process-control index,
   not a canonical syntax specification.
2. The registry records **ACCEPTED EXCEPTION**, **OPEN CONFLICT**,
   **RESOLVED**, and **REJECTED** entries. Each entry identifies the element,
   roles, boundary, rationale, canonical sources, deciding EDR, and review or
   sunset condition.
3. [Syntax Pipeline](../../SYNTAX_PIPELINE.md) Stage 3 must consult the
   registry and choose one outcome: no collision; an existing accepted
   exception applies; an open conflict must be resolved; or a new exception
   must be justified and registered through the existing acceptance path.
4. An unregistered exception is a design defect, not an implicit allowance.
   An Architecture EDR and applicable accepted syntax record must link to an
   entry whenever they accept, change, resolve, reject, or deprecate one; the
   entry must link back to those canonical records.
5. This is a **Tier 3 Process** decision. It changes neither language
   semantics nor the set of validation gates.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Undocumented overload | High | A context-dependent role is accepted without a visible exception decision. |
| Repeated rejected proposal | Medium | A withdrawn glyph reuse returns because its rejection is hard to find. |
| Provenance drift | Medium | Syntax records, Architecture EDRs, and collision history disagree. |
| Excess process overhead | Low | A parallel gate repeats work already performed by the Syntax Pipeline. |

### Consequences

- **Positive:** one discoverable process home, explicit Stage 3 treatment, and reciprocal provenance without duplicating syntax semantics.
- **Negative:** a syntax decision with a collision must update one more small artifact; the registry needs periodic review.

### Evolution

Keep the registry while the language has accepted exceptions or unresolved
syntax conflicts. Normally an obsolete entry becomes **RESOLVED** or
**REJECTED** rather than disappearing. If the registry grows beyond a
reviewable table, split it by status while retaining this entry contract.

### Compliance

1. Every Stage 3 review consults the registry and records one outcome in the
   syntax hypothesis or decision record.
2. Every accepted exception has reciprocal links among the registry, its
   Architecture EDR, and its accepted syntax record.
3. Every open conflict has an owner decision, a deferred status, or a sunset
   condition before Phase 5 completion.
4. `how/syntax/README.md` remains the hypothesis-and-decision queue; it does
   not duplicate registry entries.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep exceptions only in individual EDRs | A collision sweep cannot reliably discover distributed, differently worded exceptions. |
| Add a new validation gate | Stage 3 and existing gates already supply the needed review path. |
| Make the registry canonical syntax specification | It would duplicate What-layer records and blur process evidence with language semantics. |
| Automatically forbid every context-sensitive role | It would lose the documented Representation Symmetry exception and its boundary. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | Reviewers need a complete way to find exceptions before accepting syntax. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | The registry is a Stage 3 control while canonical meaning remains in What-layer records. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | One bounded registry extends an existing check without a parallel acceptance path. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | Statuses, reciprocal links, and review triggers make exceptions discoverable and revisitable. |

**Gates not applied:**
| Gate | Rationale |
|------|-----------|
| `ARCHITECTURAL_INTEGRITY_GATE` | This Tier 3 process decision does not change language architecture. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | The registry governs project review, not implementation strategy. |
| `LLM_GENERABILITY_GATE` | The registry does not create program syntax or code. |

**Detailed reasoning:** This process EDR's context, decision, compliance
rules, and gate table are the complete validation record for this Tier 3 decision.
