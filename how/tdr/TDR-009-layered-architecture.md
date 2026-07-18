# TDR-009: Layered Architecture

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Architecture

**Milestone:** 0

---

### Context

Programming languages often evolve with entangled concerns: syntax,
semantics, standard library, and implementation details mix in ways
that make it impossible to change one without affecting others.

Orthon's Vision states the language should be "architected like
well-designed software" — following SOLID principles. This demands
a clear architectural separation of concerns from the start.

### Decision

Adopt a **three-layer architecture**:

| Layer | Responsibility | Stability |
|-------|---------------|-----------|
| Core Language | Syntax, semantics, type system | Stable — changes rarely |
| Standard Library | Collections, I/O, concurrency primitives | Evolves — additions, deprecations |
| Implementation Strategy | Allocation, evaluation, concurrency model | Swappable — multiple profiles |

The Core Language and Standard Library define **interfaces and
contracts**; the Implementation Strategy **fulfills** them. User
programs depend on language interfaces, not implementation details.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Cross-layer coupling | High | Changing the allocator changes the type system |
| Implementation lock-in | High | Only one compiler/runtime is possible |
| Standard Library bloat | Medium | Library absorbs implementation concerns |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| All language design | External tooling (build system, formatter) |
| Standard Library design | |
| Implementation design | |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-005 (Policies & Strategies) | Policies are the mechanism for the Implementation Strategy layer |
| TDR-004 (Fitness Functions) | Layered Isolation guards layer boundaries |
| TDR-001 (Gate System) | `ARCHITECTURAL_INTEGRITY_GATE` verifies layer compliance |

### Consequences

- **Positive:**
  - Clean separation enables independent evolution of layers
  - Multiple implementation strategies can coexist
  - Standard Library can evolve without language changes
- **Negative:**
  - Requires discipline to maintain layer boundaries
  - Some designs are harder because they can't cross layers

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Monolithic language design | No separation — all concerns entangled |
| Two-layer (Language + Implementation) | Standard Library has no clear home |
| Microkernel + plugins | Overly complex for a language design phase |

### Evolution

- Layers can be **split** if a layer grows to cover two distinct
  concerns.
- New layers can be **added** if a new architectural concern emerges.
- Layer boundaries are stable — moving a concern across layers
  requires an ADR.

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [x] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [ ] `concept-design-review.md`
