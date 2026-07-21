# Optimization Model

> **⚠️ DRAFT — Placeholder for Phase 7.**
> This document defines the boundary between language semantics and
> compiler optimisation — what is guaranteed by the language vs. what
> is an optimisation choice.
>
> **Status:** Placeholder — to be filled during Phase 7 of M1.
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 7,
> [`EXECUTION_MODEL.md`](EXECUTION_MODEL.md),
> [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md)
> (§ Semantics Before Optimization, Explicit Optimization)

---

## Principle

**Semantics Before Optimization.** The language specification defines
correct program behavior. Any transformation that preserves this
behavior is an optimisation. Any transformation that changes it is
either a language feature (if intended) or a bug (if unintended).

## Classification

<!-- To be filled during Phase 7 — one row per optimisation -->

| Optimisation | Category | Classification | Rationale |
|-------------|----------|----------------|-----------|
| Inlining | Code transformation | Optimisation | Preserves semantics; changes performance only |
| Constant folding | Compile-time evaluation | Optimisation | Preserves semantics; eager or lazy produces same result |
| Escape analysis | Memory allocation | Optimisation | Stack vs. heap is invisible to semantics |
| Dead code elimination | Code removal | Optimisation | Unused code has no observable effect |
| Loop unrolling | Loop transformation | Optimisation | Preserves loop semantics |
| Evaluation order | Expression sequencing | **Semantics** | Order affects observable behavior |
| Memory layout | Struct layout | Depends on strategy | Layout not exposed in safe code |
