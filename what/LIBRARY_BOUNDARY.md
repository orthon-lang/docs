# Library Boundary

> **⚠️ DRAFT — Placeholder for Phase 4.**
> This document defines the boundary between the language core, the
> standard library, and external libraries — the canonical answer to
> "Should this be in the language or a library?"
>
> **Status:** Placeholder — to be filled during Phase 4 of M1.
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 4,
> [`DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md)

---

## Classification Criteria

| Classification | Definition | Examples |
|----------------|------------|----------|
| **Language** | Requires new syntax, semantics, or compiler support | `function`, `call`, `scope`, `assignment` |
| **Standard Library** | Built on language primitives; ships with every implementation | `vector`, `json`, `filesystem`, `regex`, `time` |
| **External Library** | Domain-specific; third-party maintained | `http`, `gui`, `crypto` |

## Decision Rule

Q2 from the Decision Pipeline: *"Is this a language problem or a library
problem?"* A feature belongs in the language only if:

1. It **requires new syntax** that cannot be expressed through existing primitives.
2. It **introduces new semantics** that cannot be implemented as a library.
3. It is **universally required** by the target audience for every program.

Otherwise, it belongs in the Standard Library or an external library.

## Classified Features

<!-- To be filled during Phase 4 — one row per feature -->

| Feature | Classification | Rationale | EDR |
|---------|---------------|-----------|-----|
| *(example)* Scope | Language | Requires compiler scope chain | EDR-NNN |
| *(example)* JSON | Standard Library | Pure library — built on strings and maps | EDR-NNN |
| *(example)* HTTP | External Library | Domain-specific protocol handling | — |
