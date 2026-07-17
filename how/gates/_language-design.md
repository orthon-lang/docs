# Language Design Gate

> Gate checklist for language design decisions. Every proposal must
> satisfy these criteria before entering formal specification.

## Criteria

- [ ] **Vision alignment** — does this solve a real problem justified
      by the project's Vision?
- [ ] **Principle compliance** — does it violate any Manifesto or Design
      Principle? If yes, is the violation intentional and documented?
- [ ] **Orthogonality** — does it compose freely with existing concepts?
      Are there special cases or context-dependent behavior?
- [ ] **Minimality** — can this be expressed through composition of
      existing concepts instead?
- [ ] **Least Astonishment** — does the behavior match what a competent
      programmer would intuitively expect?
- [ ] **Explicitness** — are semantic changes syntactically visible?
- [ ] **Named equivalence** — if symbolic, does the named function form
      exist?
- [ ] **All canonical forms** — are all equivalent forms documented?
- [ ] **Impact analysis** — which existing concepts, documents,
      Policy types, and decisions are affected?
- [ ] **Policy footprint** — which Policy Types does the concept
      affect, and is compatibility confirmed for each:
      • concept does not depend on specific Policy values
        (see `IMPLEMENTATION_INDEPENDENCE_GATE`)
      • if the Policy Type does not yet exist — it is created
        in `IMPLEMENTATION_POLICIES.md`
      • if it exists — compatibility with existing concepts
        that share this Policy Type is verified
- [ ] **Traceability** — is the full path from origin to decision
      documented (alternatives considered, rationale, rejected
      approaches)?
- [ ] **Gate guard** — has a similar proposal been rejected before? If
      so, what circumstances have changed?
- [ ] **Decision journal** — is the decision recorded with date,
      concept, verdict, and rationale for future reference?

## Decision Journal

| Date | Concept | Verdict | Rationale |
|------|---------|---------|-----------|
|      |         |         |           |
