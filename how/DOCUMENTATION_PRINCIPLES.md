# Documentation Principles

> **Last updated:** 2026-07-20

## Document Status

Every document carries a status indicator at the top. The format is:

```markdown
> **⚠️ DRAFT — This document is a preliminary draft.**
> **Status:** [Proposed | Accepted | Deprecated | Superseded by EDR-NNN]
> **Last updated:** YYYY-MM-DD
```

DRAFT documents are works in progress. Accepted documents are stable.
Deprecated documents are retained for historical reference only.

## Section Structure (8-Section Concept Template)

All concept documents in `docs/what/concepts/` follow a uniform template
defined in `docs/how/templates/_concept.md`.

Every concept must include these sections **in order**:

1. **Issue (Why)** — what problem the concept solves
2. **Principles** — constraints that cannot be violated
3. **Policy Footprint** — which Policy Types the concept involves
4. **Model (What)** — the language model and semantics
5. **Default Strategy** — default implementation behaviour
6. **Alternative Strategies** — permitted implementation variants
7. **Open Questions** — unresolved issues
8. **Decision History** — rejected alternatives and rationale

> **Note:** Previous versions of this document erroneously listed 7 sections
> (missing Policy Footprint). The correct count is **8 sections**.

This uniform structure ensures consistency across concepts and makes
navigation predictable for readers.

## Writing Principles

### Show All Canonical Forms

When documenting a language feature, always present **all** canonical
ways to use it, not just the most common one.

If a feature has multiple equivalent forms, they should be shown together
in the same section. Example from [CORE_CONCEPTS.md](../what/concepts/CORE_CONCEPTS.md):

```orthon
emit value          # emit a single value from a generator
return sequence(v)  # return a sequence from a function
return value ->     # shorthand: return value as sequence tail
```

All three forms compile to the same semantics and must be documented
together to give the reader the complete picture.

### Policy Footprint Requirement

Every concept must declare which Policy Types it involves and how. This
is documented in the **Policy Footprint** section as a table:

| Policy Type | Role in the concept |
|---|---|
| _(e.g. Allocation Policy)_ | _(how this Policy Type relates to the concept)_ |
| _(e.g. Lifetime Policy)_ | _(second relation, if any)_ |

The Policy Footprint must pass the Implementation Independence Gate:
no concept may depend on specific Policy values. It may only declare
*which* Policy Types it interacts with.

### Affected Documents Checklist

Every concept must include an **Affected Documents** checklist at the
end, listing all documents that may need updating when the concept changes:

- [ ] `CORE_CONCEPTS.md`
- [ ] `DATA_MODEL.md`
- [ ] `DESIGN_PRINCIPLES.md`
- [ ] `GLOSSARY.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] Other: ...

This checklist ensures that cross-document impact is never overlooked
and serves as a verification step during design review.

### Date-Stamp Requirement

Every concept document and architecture document must carry a
`> **Last updated:** YYYY-MM-DD` metadata line, placed after the
document title (or after the DRAFT header block if present).

The date-stamp enables freshness tracking: documents with
`last_updated` more than 90 days in the past should be flagged for
review at the next phase boundary.

### Model Before Implementation

Code examples should precede semantic explanation. Readers should see
what the concept looks like before learning the theory behind it.

Within each concept document, the **Model (What)** section should open
with concrete examples showing the concept in action, followed by the
formal semantic description. This follows the principle of *concrete
before abstract*.

### Named Before Symbolic in Documentation

When documenting operators that have both symbolic and named forms,
present the named form first (or at least at the same level of
prominence). This reinforces the [Named Before Symbolic](../what/DESIGN_PRINCIPLES.md) design principle
and makes the documentation more accessible to readers who may not
be familiar with symbolic operators.

## Document Types

| Document Type | Location | Template |
|---|---|---|
| Concept | `docs/what/concepts/` | `docs/how/templates/_concept.md` |
| EDR | `docs/how/decision_records/{category}/` | `docs/how/templates/_edr*.md` |
| Strategy | `docs/how/strategies/` | Free-form |
| Architecture | `docs/how/architecture/` | Free-form |
| Gate | `docs/how/gates/` | Free-form |
| Process | `docs/how/` | Free-form |
