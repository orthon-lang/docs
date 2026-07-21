# EDR-012: Semantic Dependency Architecture

**Status:** Accepted

**Date:** 2026-07-21

**Category:** Architecture

**Scope:** Platform

**Supersedes:** *None*

---

### Context

The current architecture (EDR-010) defines a three-layer implementation
stack: Core Language, Standard Library, Implementation Strategy. This
describes *how the language is realised* — what gets compiled, what
the runtime provides, and what bridges them.

However, the Language Layer itself has no internal structure. Concepts
like `context`, `decorator`, `property`, `pattern matching`, and
`async`/`await` sit alongside primitives like variables, function call,
and attribute access without any formal distinction between them. The
Manifesto's "Composition over exceptions" and "Minimal core" principles
imply that some features should be compositions of others, but the
architecture does not encode this distinction.

This leads to several risks:

- **Feature bloat drift:** Without a formal layer boundary, there is
  no mechanism to prevent a Language Pattern from being treated as a
  primitive — gradually expanding the "Core" beyond what is minimal.
- **Hidden coupling:** A Language Pattern may accidentally depend on
  a higher-level construct (e.g., a library) without architectural
  enforcement, violating orthogonality.
- **Unclear change scope:** When a proposal changes a feature, there
  is no architectural guidance on whether it is changing a primitive
  (affecting everything above it) or a pattern (affecting only its
  own level and below).

The project needs an architectural model that makes the internal
structure of the Language Layer explicit — distinguishing what is
truly primitive from what is composed.

### Decision

Adopt a **6-level Semantic Dependency Architecture** as an orthogonal
refinement within the Language Layer of the existing implementation
stack:

| Level | Name | Description |
|-------|------|-------------|
| 5 | **Applications** | User programs composed from frameworks and libraries |
| 4 | **Frameworks** | Reusable structural patterns built on libraries |
| 3 | **Standard Library** | Higher-level abstractions built on language patterns |
| 2 | **Language Patterns** | Compositions of primitives — convenience, no new semantics |
| 1 | **Primitive Operations** | Atomic operations on data — cannot be decomposed |
| 0 | **Data Model** | What exists — Data, Data Modifiers, Representations |

**Dependency Flow invariant:** Each level depends only on levels below
it. No level may reference or rely on constructs from a level above it.

This architecture is documented in `ARCHITECTURE.md` § Semantic
Dependency Architecture. The classification of each existing concept
into its level is tracked as a cross-reference task (see § Compliance).

### Consequences

- **Positive:**
  - Clearly distinguishes primitive operations from composed patterns
    — prevents feature bloat drift.
  - Provides architectural criteria for "is this a new primitive or a
    new pattern?" — every proposal must be classified into a level.
  - Dependency Flow invariant enforces orthogonality by construction.
  - Maps directly to existing Manifesto principles (Composition,
    Minimal Core, Extensibility).
  - Future concept design reviews can use the pyramid to determine
    the appropriate level of abstraction for each proposal.
- **Negative:**
  - Adds a classification step to every language proposal — some
    proposals may require discussion to determine their level.
  - Some existing concepts may straddle levels and need re-evaluation
    during cross-cutting review (Milestone 3).
  - The Pyramid adds conceptual overhead — readers must now
    understand two orthogonal architectural axes (implementation
    stack + semantic dependency hierarchy).

### Compliance

Compliance is verified through:

1. **Language Design Gate** (`_language-design.md`) — new checklist
   item: "To which Semantic Layer does this concept belong?" Must be
   answered before acceptance.
2. **Fitness Functions** (`FITNESS_FUNCTIONS.md`) — the existing
   Layered Isolation fitness function is extended to cover the
   semantic dependency layers.
3. **Cross-cutting review (Milestone 3)** — all existing concept
   documents are audited for level classification. Any concept
   found to violate the Dependency Flow invariant is flagged for
   remediation.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| **Keep current architecture (no internal structure)** | No mechanism to enforce composition-over-exceptions; feature bloat risk is unaddressed |
| **Two-layer model: Primitives + Everything else** | Too coarse — does not distinguish patterns from libraries, losing the "no new semantics" invariant for patterns |
| **Four-layer model without Data Model** | Data Model (Level 0) is necessary to anchor the stack — operations operate on data, and without an explicit data foundation the bottom of the pyramid is undefined |
| **Merge pyramid into existing Layered Architecture doc** | Would conflate two orthogonal axes (implementation stack vs. semantic dependency) — cleaner as a separate section within ARCHITECTURE.md |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-010 (Layered Architecture) | Defined the implementation stack; this EDR refines the Language Layer of that stack |
| EDR-005 (Fitness Functions) | Layered Isolation fitness function extended to cover semantic layers |
| EDR-007 (Concept Design Review) | Concept design reviews must now classify proposals into a Semantic Layer |
