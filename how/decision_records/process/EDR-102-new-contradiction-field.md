# EDR-102: New-Contradiction Field in the Decision Transparency Trail

**Status:** Proposed

**Date:** 2026-11-06

**Category:** Process

**Scope:** Project

---

### Context

`DESIGN_PRINCIPLES.md` § Transparency requires every design decision to
be traceable from origin to final decision, listing origin, considered
alternatives, rationale, gate review, and impact. What the trace does not
capture is the *forward* cost of a decision: the tension the decision
itself creates. The [dialectical method](../../process/DIALECTICAL_METHOD.md)
holds that every synthesis relocates a decision across a boundary and
pays for it with a new contradiction. Recording that contradiction makes
the trade-off explicit at the moment the decision is made, rather than
leaving it to resurface later as an unplanned phase or a surprise
interaction.

`DESIGN_PRINCIPLES.md` is closed for modification (Tier 1, EDR required).
This decision does not edit that locked text. It extends the Transparency
principle's Impact dimension through the EDR template and the
language-design gate — the operative instruments — leaving the
principle's wording unchanged.

### Decision

Add a mandatory **New contradiction** field to every EDR, in the
`Consequences` section, alongside Positive and Negative. The field names
the tension this decision creates — the relocated cost — as distinct
from the trade-off it accepts. The field is added via:

- the EDR templates (`how/templates/_edr.md`, `_edr-process.md`,
  `_edr-architecture.md`);
- a gate criterion in `how/gates/_language-design.md`.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Undocumented next tension | Medium | The cost of a decision is discovered later, as an unplanned phase or an interaction failure, instead of at the decision point |
| Conflated trade-off and tension | Low | A cost the decision *accepts* (Negative) is not distinguished from a tension the decision *creates* (New contradiction) |

### Consequences

- **Positive:** Trade-offs become forward-looking; each decision names the
  next tension it opens, feeding the ledger at
  [`what/DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md).
- **Negative:** One more mandatory field per EDR — a small, constant
  documentation overhead.
- **New contradiction:** Mandating a forward-looking field pressures the
  author to name a tension before it is fully understood; a premature
  "new contradiction" can ossify a guess into the record as if it were a
  finding.

### Evolution

Retire the field if future EDRs show it is routinely filled with a
placeholder rather than a real tension. Tighten it if the field proves
too weak to catch the tensions that later become phases.

### Compliance

Verified by the gate criterion and the templates: an EDR whose
`Consequences` section lacks a `New contradiction` entry fails the gate.
The field is a Tier 3 process decision under the
[Decision Process](../../process/DECISION_PROCESS.md#decision-tiers).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Edit `DESIGN_PRINCIPLES.md` § Transparency | Rejected — the principle is closed for modification; the template and gate are the operative instruments and suffice. |
| Keep the field only in the ledger, not in EDRs | Rejected — the field belongs at the point of decision; a separate ledger loses the per-decision trace. |
| Fold "new contradiction" into the existing "Negative" bullet | Rejected — a trade-off accepted (Negative) and a tension created (New contradiction) are distinct; conflating them hides the dialectical cost. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Names the cost of a decision at the point of decision, preventing it from resurfacing as an unplanned phase. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Extends the Transparency Impact dimension through template and gate; does not change the locked principle text. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One added field and one gate criterion; no new artifact type. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | The field has a defined retirement condition if it degrades to placeholder. |

Architectural integrity, implementation independence, and LLM
generability gates are not applied to this process decision; no language
construct or implementation strategy is changed.

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| [EDR-102](EDR-102-new-contradiction-field.md) | This record. |
| [`DIALECTICAL_METHOD.md`](../../process/DIALECTICAL_METHOD.md) | The method that motivates the field and defines its acceptance bar. |
| [`DIALECTICAL_TRADEOFFS.md`](../../../what/DIALECTICAL_TRADEOFFS.md) | The ledger the field feeds. |
