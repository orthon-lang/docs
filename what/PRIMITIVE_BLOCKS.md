# Primitive Blocks

> **⚠️ DRAFT — Placeholder for Phase 3.**
> This document will identify the minimal orthogonal set of primitive
> building blocks from which all language features are composed.
>
> **Status:** Placeholder — to be filled during Phase 3 of M1.
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 3,
> [`SEMANTIC_MODEL.md`](SEMANTIC_MODEL.md),
> [`FOUNDATIONAL_ABSTRACTIONS.md`](../how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md)

---

## Purpose

Every language construct must decompose into primitive blocks. If a
derived feature cannot be decomposed, the primitive set is incomplete.
If two primitives overlap, the set is not orthogonal.

## Starting Hypothesis

The following is a candidate set, to be validated during Phase 3:

| Primitive | Description | Semantic Justification |
|-----------|-------------|----------------------|
| `identifier` | Named reference to a value | Identity — how values are named and referred to |
| `literal` | Inline value notation | Data — how values are written directly |
| `assignment` | Bind value to name | Evaluation — how values are stored for later use |
| `function` | Parameterized computation | Evaluation — how computations are abstracted |
| `call` | Invoke a function | Evaluation — how computations are triggered |
| `attribute access` | Access member of composite | Visibility — how composite values expose parts |
| `scope` | Lexical boundary | Visibility/Lifetime — how names are contained |
| `reference` | Indirection to a value | Ownership — how indirection is expressed |
| `pack` | Combine values into composite | Data — how composites are built |
| `unpack` | Decompose composite into parts | Data — how composites are destructured |
| `operator definition` | User-definable operator semantics | Explicitness — how syntax is extended |

## Validation

<!-- To be filled during Phase 3 — check each research concept for decomposition -->

## EDR

- **EDR-NNN:** Acceptance of Orthon Primitive Blocks
  <!-- Created during Phase 3 -->
