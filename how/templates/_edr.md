# EDR Template — Engineering Decision Record

> Use this template to record any engineering decision in the Orthon project.
> Place each EDR as `docs/how/decision_records/{category}/EDR-NNN-slug.md`.
> Use this template from `docs/how/templates/_edr.md`.
>
> Category-specific variants:
> - `_edr-architecture.md` — Architecture decisions
> - `_edr-process.md` — Process decisions
> - (Other lenses use this base template; category-specific sections added on demand)

---

## EDR-NNN: [Short Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded by EDR-NNN]

**Date:** YYYY-MM-DD

**Category:** [Architecture | Technology | Tooling | Process | Delivery | Operations | Quality | Security | Governance | Data | AI | Documentation | Knowledge | Collaboration | Product]

**Scope:** [Platform | Subsystem | Module | Team | Project | Policy]

---

### Context

What problem or tension prompted this decision?

Describe the forces at play: technical constraints, philosophical principles,
trade-offs, conflicting requirements. Include references to relevant project
documents (VISION.md, MANIFESTO.md, DESIGN_PRINCIPLES.md, etc.).

> *Anchor the context — why does this decision matter?*

---

### Decision

What did we decide?

State the decision clearly and concretely. This is the single outcome of this record.

---

### Consequences

What does this decision make easier and what does it make harder?

- **Positive:**
  - …
- **Negative:**
  - … (trade-offs, overhead)

---

### Compliance

How will we verify that this decision is followed?

For example: code review checklist item, gate entry, automated lint rule,
documentation convention, or fitness function.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| … | … |

---

<!-- Category-specific sections (add below this line as needed) -->
