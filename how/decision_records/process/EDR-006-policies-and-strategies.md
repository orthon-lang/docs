# EDR-006: Implementation Policies & Strategies

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

**Supersedes:** TDR-005

---

### Context

Language semantics define *what* a program means. Implementation
details define *how* that meaning is realised. Without a systematic
separation, the two become entangled: semantics leak implementation
assumptions, and changing the implementation changes observable
behaviour.

Orthon's layered architecture (EDR-010) requires that the Language
Specification be implementable through multiple strategies. This
demands a clear, declarative interface between *what* and *how*.

### Decision

Adopt a **Policy/Strategy model**:

- A **Policy** is a declarative constraint or preference within a
  single domain-specific area (allocation, evaluation, concurrency,
  etc.). Policies state *what* is preferred, not *how* to achieve it.
- A **Strategy** is a named set of Policies assembled into a coherent
  profile (Default, Embedded, High-Performance).

Policies are building blocks; Strategies compose them. Both are
declarative, not procedural.

The Policy catalogue (`IMPLEMENTATION_POLICIES.md`) is populated
as language concepts are designed. Each concept's Policy footprint
is verified through `IMPLEMENTATION_INDEPENDENCE_GATE`.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Semantics-implementation coupling | High | Can't change allocator without changing program behaviour |
| Strategy lock-in | High | Only one implementation profile is possible |
| Concept-Policy ambiguity | Medium | Unclear which concepts drive which implementation choices |

### Consequences

- **Positive:**
  - Clean separation of what (language) from how (implementation)
  - Multiple strategies can coexist for different use cases
  - New strategies can be added without changing the language spec
- **Negative:**
  - Adds abstraction layer between spec and implementation
  - Policy catalogue must be maintained as concepts grow

### Evolution

- New Policies are added as new concepts are designed.
- New Strategies are assembled from existing Policies for new use cases.

### Compliance

Verified through `IMPLEMENTATION_INDEPENDENCE_GATE` (EDR-002) during
EDR-007 (Concept Design Review).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Hard-coded implementation choices in spec | Locks language to one implementation |
| No explicit Policy model | Ad-hoc implementation decisions; no composability |
| Procedural strategy selection | Violates declarative principle; tangled logic |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-010 (Layered Architecture) | Policies/Strategies are the mechanism for the Implementation layer |
| EDR-002 (Gate System) | `IMPLEMENTATION_INDEPENDENCE_GATE` verifies Policy separation |
