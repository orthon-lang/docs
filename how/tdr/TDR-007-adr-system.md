# TDR-007: Architecture Decision Records (ADR)

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Template

**Milestone:** 0

---

### Context

Design decisions made during language development have long-term
consequences. Without documented rationale, future contributors
cannot understand *why* a decision was made — only *what* was
decided. This leads to repeated debates, accidental reversals,
and design drift.

The project needs a lightweight, structured format for recording
significant decisions with their context, alternatives, and
consequences.

### Decision

Adopt **Architecture Decision Records (ADR)** — a proven lightweight
format for documenting design decisions.

Each ADR captures:
- Context (what problem prompted the decision)
- Decision (what was decided, concretely)
- Consequences (positive and negative)
- Compliance (how to verify the decision is followed)
- Alternatives Considered (and why they were rejected)

ADRs are stored in `docs/how/adr/` with numeric prefixes. The
template is in `docs/how/templates/_adr.md`.

ADRs are created during Milestone 2 as the final step (Step 11)
of the Concept Design Review.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Design amnesia | High | "Why did we choose X over Y?" — no one remembers |
| Repeated debates | Medium | Same alternatives re-litigated without new information |
| Accidental reversals | Medium | Decision reversed without understanding original rationale |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| All language design decisions | Trivial implementation notes |
| Significant architectural choices | Work-in-progress ideas |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-006 (Concept Design Review) | Step 11 produces an ADR |
| TDR-008 (TDR System) | TDR is the process-tool analogue of ADR |

### Consequences

- **Positive:**
  - Decisions survive beyond the people who made them
  - Alternatives are documented — prevents re-litigation
  - Structured format ensures consistency
- **Negative:**
  - Writing overhead per decision
  - ADRs must be maintained (deprecated, superseded)

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Comments in specification | Not discoverable; mixed with spec content |
| Wiki / shared doc | Unstructured; no template; decays over time |
| No records | Unacceptable for a project with multi-year scope |

### Evolution

- ADR format can be **extended** with new sections if needed.
- ADRs can be **deprecated** or **superseded** as decisions evolve.
- The ADR directory may be **archived** for old milestones.

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [x] `concept-design-review.md`
- [x] `templates/_adr.md`
