# EDR-072: Context-Limited Modules — Compiler-Enforced Capability-Based Module Access

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

How does a module system help an LLM (or a human) understand a module's behaviour without loading the entire codebase into context? Traditional module systems allow undeclared transitive dependencies to leak, making it impossible to understand a module's surface without loading its entire dependency graph.

Context-limited modules solve this by: explicit dependency declarations (no undeclared transitive access), effect isolation (compiler verifies declared effects), and context window budget diagnostics.

The module system affects name resolution, compilation units, and visibility — none of which can be expressed as a library function.

---

### Decision

Context-limited modules are a **Language** construct:
- Every module declares its public API and explicit dependencies in a header section.
- Only declared dependencies are accessible — no undeclared transitive imports.
- Side effects (I/O, allocation, mutation) are declared in the module header.
- The compiler verifies the module performs only declared effects.
- No effect leakage from dependencies — a module does not inherit dependency effects.
- Toolchain provides context window budget diagnostics (signature token counts).

---

### Consequences

- **Positive:**
  - LLM can understand a module from its header + direct dependency headers.
  - Effect isolation enables independent reasoning about module behaviour.
  - No undeclared transitive dependency access — full transparency.
- **Negative:**
  - Module headers are more verbose than traditional module systems.
  - Effect tracking adds compiler complexity.
  - Effect isolation may require refactoring of existing module structures.

---

### Compliance

- The compiler must enforce that only declared dependencies are accessible.
- The compiler must verify effect declarations against module body.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| File-scoped (C/Go) | No visibility control — transitive dependencies leak. |
| Separate interface files (OCaml `.mli`) | Manual maintenance — drift risk between signature and implementation. |
| No module system | Namespace-only — no isolation guarantees. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Understand a module from its header — direct LLM and human benefit. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Explicit dependency graph — no hidden access. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One module header format covers API, dependencies, and effects. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Module system is the foundation of compilation units — architectural centrality justified. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Module semantics independent of compilation model. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Explicit module boundaries improve long-term maintainability. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLM can see a module's entire surface in its header — critical for context window management. |
