# TDR-005: Implementation Policies & Strategies

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Policy

**Milestone:** 0

---

### Context

Language semantics define *what* a program means. Implementation
details define *how* that meaning is realised. Without a systematic
separation, the two become entangled: semantics leak implementation
assumptions, and changing the implementation changes observable
behaviour.

Orthon's layered architecture (TDR-009) requires that the Language
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

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| All implementation-variant aspects | Core language semantics |
| Allocation, evaluation, concurrency, lifetime, algorithm selection | Syntax definition |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-009 (Layered Architecture) | Policies/Strategies are the mechanism for the Implementation layer |
| TDR-001 (Gate System) | `IMPLEMENTATION_INDEPENDENCE_GATE` verifies Policy separation |

### Consequences

- **Positive:**
  - Clean separation of what (language) from how (implementation)
  - Multiple strategies can coexist for different use cases
  - New strategies can be added without changing the language spec
- **Negative:**
  - Adds abstraction layer between spec and implementation
  - Policy catalogue must be maintained as concepts grow

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Hard-coded implementation choices in spec | Locks language to one implementation |
| No explicit Policy model | Ad-hoc implementation decisions; no composability |
| Procedural strategy selection | Violates declarative principle; tangled logic |

### Evolution

- New Policy types are **added** as new language concepts require them.
- New Strategies are **added** as new use cases emerge.
- Policy values can be **extended** without breaking existing strategies.

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [x] `IMPLEMENTATION_POLICIES.md`
- [x] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [ ] `concept-design-review.md`
- [x] `strategies/DEFAULT_STRATEGY.md`
- [x] `strategies/EMBEDDED_STRATEGY.md`
- [x] `strategies/HIGH_PERFORMANCE_STRATEGY.md`
