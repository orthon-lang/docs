# ADR Template — Architecture Decision Record

> **⚠️ DEPRECATED.** This template has been superseded by the
> [EDR Architecture template](_edr-architecture.md). ADRs are now a
> category of Engineering Decision Records (EDR) — see
> [EDR-001](../decision_records/process/EDR-001-edr-system.md).
>
> Do not use this template for new decisions. Use `_edr-architecture.md`
> instead and place records in `docs/how/decision_records/architecture/`.

---

## ADR-NNN: [Short Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-NNN]

**Date:** YYYY-MM-DD

**Driver:** [Who initiated or is responsible for this decision]

---

### Context

What is the problem or design tension that prompted this decision?

Describe the forces at play: technical constraints, philosophical principles, trade-offs, conflicting requirements. Include relevant references to `../../why/VISION.md`, `../../why/MANIFESTO.md`, `../../what/DESIGN_PRINCIPLES.md`, or other documents.

> *Anchor the context in the project's Why or How layer.*

---

### Decision

What did we decide?

State the decision clearly and concretely. This is the single outcome of this record.

---

### Consequences

What does this decision make easier and what does it make harder?

List both positive consequences (benefits, enabled capabilities) and negative consequences (constraints, trade-offs accepted).

- **Positive:**
  - ...
- **Negative:**
  - ...

---

### Compliance

How will we verify that this decision is followed in future design work?

For example: code review checklist item, gate entry, automated lint rule, or documentation convention.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| [Option A] | Why it was not chosen. |
| [Option B] | Why it was not chosen. |

---

### Affected Documents

- [ ] `VISION.md`
- [ ] `MANIFESTO.md`
- [ ] `ZEN.md`
- [ ] `ARCHITECTURE.md`
- [ ] `DESIGN_PRINCIPLES.md`
- [ ] `what/CORE_CONCEPTS.md`
- [ ] `DATA_MODEL.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `GLOSSARY.md`
- [ ] `AGENTS.md`
- [ ] Other: ...

---

### Gate Entry

If this decision requires explicit review, create a corresponding entry in `gates/_language-design.md`.

---

### Notes

Any additional commentary, follow-up tasks, or links to discussion.
