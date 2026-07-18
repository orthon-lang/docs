# TDR-006: Concept Design Review (11-Step Procedure)

**Status:** Superseded by [EDR-007](../decision_records/process/EDR-007-concept-design-review.md)

**Date:** 2026-07-18

**Domain:** Process

**Milestone:** 0

---

### Context

Without a uniform design procedure, each concept risks being designed
through a different ad-hoc process. Some may start from the problem,
others from the solution. Some may check principles, others may skip
that step. The result is inconsistent quality across the language
specification.

The project needs a single, repeatable procedure that every concept
goes through — ensuring problem-first design, principle compliance,
and full traceability.

### Decision

Adopt an **11-step Concept Design Review** procedure:

1. **Problem** — state the real-world problem
2. **Use Cases** — concrete scenarios
3. **Necessity** — is this concept required?
4. **Sufficiency** — does it fully solve the problem?
5. **Alternatives** — what else was considered?
6. **Principles Check** — does it violate any principle?
7. **Interactions** — which other concepts does it touch?
8. **Rationale** — why this design over alternatives?
9. **Examples** — illustrative usage
10. **Open Questions** — unresolved issues
11. **ADR** — Architecture Decision Record

The procedure is applied during Milestone 2 for every concept in the
Language Inventory. The output is evaluated through the Decision
Validation Gates (TDR-001) and the Language Design Checklist (TDR-003).

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Inconsistent concept quality | High | Some concepts are well-designed, others are shallow |
| Solution-first design | High | Technically interesting ideas without validated problems |
| Missing traceability | Medium | Decisions lack documented rationale |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| Every concept in LANGUAGE_INVENTORY.md | Vision/philosophy documents |
| | Cross-cutting review (Milestone 3) |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-001 (Gate System) | Gates evaluate the 11-step output |
| TDR-003 (Checklist) | Checklist records the outcome |
| TDR-007 (ADR System) | Step 11 produces an ADR |

### Consequences

- **Positive:**
  - Uniform quality across all concepts
  - Problem-first prevents solution-first bias
  - Full traceability from problem to ADR
- **Negative:**
  - 11 steps per concept is significant overhead
  - Risk of mechanical execution without genuine analysis

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Lighter process (3-5 steps) | Insufficient depth for language design decisions |
| Ad-hoc per-concept process | Inconsistent; no quality guarantee |
| RFC-style process | Good for community input, poor for principled design |

### Evolution

- Steps can be **reordered** if a better sequence is discovered.
- Steps can be **split** if one step grows to cover two distinct
  concerns.
- The procedure is stable — changes require evidence that the
  current sequence produces worse outcomes.

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [x] `concept-design-review.md`
