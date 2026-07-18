# EDR-010: Layered Architecture

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Architecture

**Scope:** Platform

**Supersedes:** TDR-009

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

### Consequences

- **Positive:**
  - Clean separation enables independent evolution of layers
  - Multiple implementation strategies can coexist
  - Standard Library can evolve without language changes
- **Negative:**
  - Requires discipline to maintain layer boundaries
  - Some designs are harder because they can't cross layers

### Evolution

The three-layer model is fundamental to Orthon's architecture. Changes
to the layer structure itself require an Architecture-category EDR.

### Compliance

Verified through `ARCHITECTURAL_INTEGRITY_GATE` (EDR-002) and the
Layered Isolation fitness function (EDR-005).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Monolithic language design | No separation — all concerns entangled |
| Two-layer (Language + Implementation) | Standard Library has no clear home |
| Microkernel + plugins | Overly complex for a language design phase |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-006 (Policies & Strategies) | Policies are the mechanism for the Implementation Strategy layer |
| EDR-005 (Fitness Functions) | Layered Isolation guards layer boundaries |
| EDR-002 (Gate System) | `ARCHITECTURAL_INTEGRITY_GATE` verifies layer compliance |
