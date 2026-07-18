# TDR-008: Tools Decision Records (TDR)

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Template

**Milestone:** 0

---

### Context

ADR (TDR-007) records decisions about language design. But the
process infrastructure itself — gates, methods, fitness functions,
policies, checklists — embodies decisions that are equally
consequential. Without documented rationale for these process tools,
they risk becoming ritual: followed without understanding *why*,
and therefore vulnerable to drift or abandonment.

The project needs a record type for process-tool decisions —
distinct from ADR to avoid conflating language decisions with
process decisions.

### Decision

Adopt **Tools Decision Records (TDR)** — an analogue of ADR for
process infrastructure decisions.

TDRs use the same lightweight format as ADRs, with two additional
sections specific to process tools:

- **Without It** — what specific risks materialise if the tool is
  removed or skipped (answers the question: "what do we lose?")
- **Scope** — explicit boundaries: when the tool applies and when
  it does NOT apply
- **Evolution** — can the tool be deprecated, extended, or replaced?
  Under what circumstances?

TDRs are stored in `docs/how/tdr/` with numeric prefixes. The
template is in `docs/how/templates/_tdr.md`.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Process becomes ritual | High | Tools followed without understanding why |
| Process drift | Medium | Tools gradually abandoned or altered without rationale |
| Onboarding friction | Medium | New contributors see process as arbitrary bureaucracy |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| All process infrastructure decisions | Language design decisions (use ADR) |
| Gates, methods, policies, fitness functions, checklists, procedures, templates | Implementation details of tools |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-007 (ADR System) | TDR is the process-tool analogue of ADR |
| TDR-010 (Process Inventory) | Process Inventory is the aggregated view of all TDRs |
| All TDR-001…TDR-009 | TDR-008 is the recursive TDR about the TDR system itself |

### Consequences

- **Positive:**
  - Process decisions have the same rigor as language decisions
  - "What do we lose?" is explicitly documented for every tool
  - New contributors can understand *why* the process exists
- **Negative:**
  - Additional record type to maintain
  - Recursive (TDR about TDR) — requires discipline to avoid
    infinite regress

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Use ADR for process decisions | Conflates language and process decisions; ADR lacks "Without It" and "Scope" sections |
| No records for process tools | Process becomes unexamined ritual |
| Inline rationale in each tool's doc | Scattered; not discoverable; inconsistent format |

### Evolution

- TDR format can be **extended** if new process-specific sections
  are needed.
- TDR system itself can be **deprecated** if the project adopts a
  different process-recording mechanism.
- The TDR directory may be **archived** for old milestones.

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [ ] `concept-design-review.md`
- [x] `templates/_tdr.md`
- [x] `PROCESS_INVENTORY.md`
