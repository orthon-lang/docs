# Documentation Principles

## Show All Canonical Forms

When documenting a language feature, always present all canonical ways to use it, not just the most common one.

If a feature has multiple equivalent forms, they should be shown together in the same section.

## Concept Document Template

All concept documents in `docs/what/concepts/` follow a uniform template
defined in `docs/how/templates/_concept.md`.

Every concept must include these sections in order:

1. **Issue (Why)** — what problem the concept solves
2. **Principles** — constraints that cannot be violated
3. **Model (What)** — the language model and semantics
4. **Default Strategy** — default implementation behaviour
5. **Alternative Strategies** — permitted implementation variants
6. **Open Questions** — unresolved issues
7. **Decision History** — rejected alternatives and rationale

This uniform structure ensures consistency across concepts and makes
navigation predictable for readers.
