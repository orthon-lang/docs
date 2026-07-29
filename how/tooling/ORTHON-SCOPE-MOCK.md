# Tooling: Orthon Scoped Function Mocking

> **Status:** Open · **Priority:** P3 · **Target:** M3 (Tooling)
> **Source:** [BAML gap analysis](../../notes/baml-concepts-orthon-gap-analysis.md) § Tier 2 (#13)

---

## Description

A mechanism to replace dangerous or non-deterministic functions
within a lexical scope — enabling sandboxed execution of LLM-generated
code without risking side effects (network calls, file system writes,
etc.).

Analogous to BAML's planned scoped function mocking feature.

## Problem

When LLM agents generate Orthon code dynamically, the generated code
may invoke functions with side effects (e.g., HTTP calls, filesystem
access). Running such code requires sandboxing. Scoped mocking lets
the caller replace dangerous functions with safe stubs within a
controlled scope.

## Spec Impact

### If this tool exists

- LLM-generated code can be executed safely in a sandboxed scope.
- Enables agent self-correction loops: generate → run in sandbox →
  observe → fix.
- Foundation for testing and evaluation infrastructure.

### If this tool does NOT exist

- Every execution of LLM-generated code carries side-effect risk.
- Sandboxing must be implemented externally (container, WASM sandbox).
- Agent self-correction loops are more complex and slower.

## Language Feature Dependencies

- **Scope model** — Must support lexical scope as a first-class
  boundary for function replacement. See `CONTEXT_LIMITED_MODULES`
  (EDR-062) for capability-based access control.
- **Function model** — Function references must be replaceable within
  a scope (dependency injection at the language level).
- **Ecosystem boundary** — Likely a **Standard Library** (M2) feature
  rather than language core, with a CLI tool wrapper (M3).

## Spec Documents Affected

- [`what/SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md) — The
  visibility and scope dimensions should define whether scoped
  function replacement is semantically valid.
- [`what/LIBRARY_BOUNDARY.md`](../../what/LIBRARY_BOUNDARY.md) —
  Scoped mocking is a stdlib candidate. The boundary rationale
  should document this classification.
- [`how/concepts/research/CONTEXT_LIMITED_MODULES.md`](../concepts/research/CONTEXT_LIMITED_MODULES.md) —
  The capability model may already provide the scope for mocking.

## Notes

- BAML's version is planned (not yet shipped). Monitor their BEPs.
- Potential overlap with capability-based security model (see
  `CONTEXT_LIMITED_MODULES` EDR-062).
- Two design approaches: (1) language-level `mock { ... }` scope,
  or (2) runtime-level scope via tooling. The template should capture
  this design choice.
- For M1 spec purposes, the key requirement is that the semantic model
  does not *prevent* scoped function replacement.
