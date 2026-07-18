# TDR Template — Tools Decision Record

> **⚠️ DEPRECATED.** This template has been superseded by the
> [EDR Process template](_edr-process.md). TDRs are now a category of
> Engineering Decision Records (EDR) — see
> [EDR-001](../decision_records/process/EDR-001-edr-system.md).
>
> Do not use this template for new decisions. Use `_edr-process.md`
> instead and place records in `docs/how/decision_records/process/`.

---

## TDR-NNN: [Tool / Approach Name]

**Status:** [Proposed | Accepted | Deprecated | Superseded by TDR-NNN]

**Date:** YYYY-MM-DD

**Domain:** [Gate | Method | Policy | Fitness Function | Process | Template | Architecture]

**Milestone:** [0 | 1 | …]

---

### Context

What problem in the design process does this tool solve?
What failure mode does it prevent?
What triggered the need for this tool?

> *Anchor in the project's process philosophy — why is this tool
> necessary for the Orthon design workflow?*

---

### Decision

What tool or approach did we adopt?

State the decision clearly: its structure, its method, its outputs.

---

### Without It

What do we lose if this tool is removed or skipped?

| Risk | Severity | Manifestation |
|------|----------|---------------|
| … | … | … |

---

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| … | … |

---

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| … | … |

---

### Consequences

- **Positive:**
  - …
- **Negative:**
  - … (trade-offs, overhead)

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| … | … |

---

### Evolution

- Can this tool be deprecated? Under what circumstances?
- Can it be extended? How?
- What would signal that this tool is no longer needed?

---

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [ ] `concept-design-review.md`
- [ ] Other: …
