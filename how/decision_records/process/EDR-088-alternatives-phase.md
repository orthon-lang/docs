# EDR-088: Concept Design Review — Alternatives Phase (7→8 steps)

**Status:** Accepted

**Date:** 2026-08-20

**Category:** Process

**Scope:** Project

---

### Context

The Concept Design Review procedure (`how/concept-design-review.md`) was
purely convergent. It moved from Idea/Problem directly to Minimal
Solution, so alternative candidate solutions were never generated
before selection:

- **Missing rational-decision stage.** The classical rational
  decision-making model has six stages, including "generate possible
  alternatives" (stage 3) before "evaluate alternatives" (stage 4). The
  review procedure covered stages 1–2 and 4–6 but had no divergent
  generation phase.
- **Retrospective-only alternatives.** The EDR template's "Alternatives
  Considered" table was filled after the decision was made. Recording
  alternatives post-choice is vulnerable to choice-supportive bias —
  the chosen option appears obviously best and rejected options appear
  worse than they actually were.
- **Decision Pipeline is convergent, not generative.** The 10-question
  Decision Pipeline (`how/process/DECISION_PIPELINE.md`) checks whether
  a proposal can be expressed as sugar or composition (Q5–Q7), but it
  never requires enumerating a solution space.

### Decision

1. **Insert Step 2 "Alternatives"** into the Concept Design Review,
   between Idea/Problem and Minimal Solution. The procedure becomes
   8 steps: Idea/Problem → **Alternatives** → Minimal Solution →
   Principle Check → Examples → Tooling Implications → Convergence
   Check → EDR.
2. **Canonical candidate set.** The step requires at least three
   candidate solutions drawn from the set: (A) language construct,
   (B) syntactic sugar, (C) library / composition, (D) do nothing —
   which maps directly to the Decision Pipeline exit paths. At most one
   additional "wild" alternative is permitted (5 candidates max).
3. **Fixed evaluation criteria.** Every candidate is scored against the
   same criteria the validation gates use (Design Principles,
   minimality, orthogonality, LLM Generability Gate) in a comparison
   table with a Selection row.
4. **Minimal Solution becomes selection.** Step 3 selects and refines
   the winner of Step 2 rather than inventing a solution from scratch.
5. **EDR fed verbatim.** The Step 2 comparison table is carried verbatim
   into the EDR's "Alternatives Considered" field — the field is no
   longer filled retrospectively.
6. **Convergence Check enforcement.** The Convergence Check gains a
   sixth sub-check "Alternatives documented" (≥3 candidates with a
   Selection row) so the phase cannot be silently skipped.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Choice-supportive bias | High | Rejected alternatives recorded after selection are remembered as worse than they were; rationale becomes rationalisation |
| No divergent search | High | Design space collapses to the first idea; competing solutions never considered |
| EDR becomes decoration | Medium | "Alternatives Considered" filled as an afterthought with no real generation behind it |
| Rational-model gap | Medium | Process claims to validate decisions but skips the generation stage of the decision cycle |

### Consequences

- **Positive:**
  - Closes the missing "generate alternatives" stage of the rational decision model.
  - EDR "Alternatives Considered" becomes a forward artifact, fed by a real generation phase.
  - Bounded overhead: 3–5 candidates, checklist step, no new files.
  - Reuses existing machinery (Decision Pipeline exit paths, validation gate criteria).
- **Negative:**
  - One additional step per concept review.
  - Risk of formulaic "checkbox" alternatives if the table is filled without genuine divergence — mitigated by the Convergence Check sub-check.

### Evolution

This phase should be reviewed for removal or simplification if:
- Concept reviews show the comparison table adds no value (every
  decision ends up identical anyway); or
- The overhead consistently outweighs the bias protection for solo
  authorship.

### Compliance

- The Convergence Check sub-check "Alternatives documented" must pass
  before the EDR is filed.
- EDR "Alternatives Considered" tables should match the Step 2
  comparison table (verbatim carry-over).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Split Step 2 into "2a Generate / 2b Select" without renumbering | Weaker as an explicit phase; renumbering is cheap and makes the divergence explicit in every cross-reference |
| Add only a Convergence Check sub-check (no new step) | Generation would still be retrospective; the step must precede selection to prevent choice-supportive bias |

### Gate Validation

> Process decisions affect *how* the project operates, not *what* the
> language is. They must balance rigour against overhead — a process
> that is not followed is worse than no process at all.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | Solves a real failure mode (bias, no divergent search) at bounded cost |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | Consistent with Decision Pipeline exit paths and existing gate criteria; contradicts no process |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | One checklist step; reuses existing criteria; no new artifacts |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | Enforced by the Convergence Check; low maintenance surface |

**Gates not applied:**

| Gate | Rationale |
|------|-----------|
| `ARCHITECTURAL_INTEGRITY_GATE` | Process decisions don't affect language architecture. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Process decisions are about project operations, not implementation strategies. |
| `LLM_GENERABILITY_GATE` | Process decisions don't produce code — LLM generability is not applicable. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for this EDR for per-gate reasoning trail.

---

> **Human Sign-off:** `Reviewed-by: mniedre · Date: 2026-08-20 · Verdict: LOCKED`
> Required before the EDR is finalized (AGENTS.md §7.4 — non-automatable).
> Only the solo author's explicit confirmation counts.
