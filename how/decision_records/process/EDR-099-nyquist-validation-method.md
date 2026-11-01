# EDR-099: Nyquist Validation as a Supplemental Method

**Status:** Accepted

**Date:** 2026-11-01

**Category:** Process

**Scope:** Project

---

### Context

Orthon's seven Decision Validation gates examine proposals from
different perspectives, and fitness functions check specific design
properties. A passing gate or a documented example can still miss a
semantic violation if the selected evidence cannot distinguish the
valid outcome from the violating one. The risk is highest at concept
boundaries, across event orders, and across Implementation Strategies.

### Decision

Adopt [Nyquist Validation](../../gates/methods/NYQUIST_VALIDATION_METHOD.md)
as a supplemental process method. For a significant semantic invariant,
the reviewer records its canonical source, plausible violation
scenarios, an observation that distinguishes each violation, and the
uncovered or deferred cases. Apply the audit while preparing or
reviewing validation evidence after the Decision Pipeline's initial
filter. It audits evidence across existing gates and fitness functions;
it does not introduce an eighth gate or change language semantics.

The name is an analogy to sampling: insufficient observations can hide
meaningful changes. It does not import a Nyquist frequency, a numerical
coverage threshold, or a proof of completeness. In this documentation
repository, specified outcomes and counterexamples are design evidence,
not passing executable tests. GSD's similarly named mapping from phase
requirements to automated tests is a different mechanism.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Undetectable invariant violation | High | All selected examples look valid even when an important interaction breaks |
| Overstated assurance | Medium | A gate verdict is presented as broader evidence than its scenarios support |

### Consequences

- **Positive:** Reviewers can identify missing discriminating scenarios
  before treating gate evidence as sufficient.
- **Negative:** The audit adds documentation work for significant
  invariants. It is targeted to consequential cases and records its
  coverage limit to avoid an unbounded completeness exercise.

### Evolution

Simplify or retire the method if repeated concept reviews show that it
only duplicates existing gate evidence. Extend its scenario catalogue
when implementation tests become available, without treating those
tests as proof of universal correctness.

### Compliance

For each selected invariant, a review trail identifies the canonical
rule, at least one meaningful violation scenario, the discriminating
observation, and any uncovered or deferred cases. The
[Concept Design Review](../../concept-design-review.md#validation)
and [Decision Validation](../../gates/DECISION_VALIDATION.md#supplemental-method-nyquist-validation)
point to the method. This is a Tier 3 process decision under the
[Decision Process](../../process/DECISION_PROCESS.md#decision-tiers).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Add an eighth validation gate | Detectability is about the adequacy of evidence across existing gates, not an independent language quality attribute. |
| Use only GSD-style requirement-to-test mapping | A documentation-only design has no compiler tests; mapping requirements to checks would not establish that the checks reveal semantic violations. |
| Keep the existing process unchanged | Existing gates specify what to assess but do not explicitly test whether chosen observations distinguish valid and invalid outcomes. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Reviewers need to know whether significant semantic failures would be visible in their evidence. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | The method audits evidence across gates without changing their criteria or verdict rules. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Four bounded steps reuse existing invariants, scenarios, and review artifacts. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | The audit can be simplified or retired if it duplicates gate evidence. |

Architectural integrity, implementation independence, and LLM
generability gates are not applied to this process decision; no
language construct or implementation strategy is changed.

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| [EDR-003](EDR-003-validation-methods.md) | Retains the methods assigned to validation gates; this method supplements their evidence. |
| [EDR-002](EDR-002-gate-system.md) | Keeps the gate system and gate count unchanged. |
| [EDR-005](../quality/EDR-005-fitness-functions.md) | Fitness functions can supply invariants and observations for the audit. |
