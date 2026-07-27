# EDR-038: Representation Modifiers — PRIMITIVE_BLOCKS Correction for Type Storage Annotations

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Primitive Blocks)

---

### Context

The research document [`REPRESENTATION_MODIFIERS.md`](../../concepts/research/essential/REPRESENTATION_MODIFIERS.md) proposes a family of representation modifiers (`struct(T)`, `boxed(T)`, `shared(T)`, `atomic(T)`, `ffi(T)`, `packed(T)`) that decouple *what a type is* from *how it is stored*. The core insight: `struct(User)` and `boxed(User)` represent the *same semantic type* — they differ only in memory layout and ownership model.

The question for this EDR: Where does this concept belong in Orthon's architecture?

**Per D-03 classification rules**, the Decision Pipeline was run (see `DECISION_PIPELINE.md` § Essential — Derived Features, Representation Modifiers entry). The pipeline reveals:

- **Q3 (Existing primitives?):** Representation modifiers are orthogonal annotations on the `pack` primitive (for inline/struct layout) and the `reference` primitive (for indirection/boxed). They do not add new primitives — they are modifiers on existing primitives.
- **Q4 (Violate principles?):** No — they align with Explicit Semantics (representation modifier syntax is visible) and Orthogonality (modifiers compose with any type).
- **Q5 (New semantics?):** The modifiers add *annotation semantics* (storage strategy selection), but these are not new primitive operations — they are constraints on how existing primitives are materialised.

---

### Decision

**Classification: PRIMITIVE_BLOCKS correction.** Representation modifiers are orthogonal annotations on existing primitives, not new primitives or a standalone Language feature.

**Rationale:**

1. **Annotations on primitives.** `struct(T)` is an annotation on the `pack` primitive indicating "pack without runtime metadata." `boxed(T)` is an annotation on the `reference` primitive indicating "heap-allocate with full runtime info." The primitives themselves (`pack`, `reference`) remain unchanged.
2. **Not new primitives.** Each modifier describes *how* an existing primitive is realised, not *what* operation is performed. Adding them as primitives would violate orthogonality — they would overlap with `pack` (struct vs. pack) and `reference` (boxed vs. reference).
3. **Correction to PRIMITIVE_BLOCKS.md.** The primitive blocks document should note that representation annotations are orthogonal modifiers on the `pack` and `reference` primitives — not new primitives themselves. This clarifies the design space without adding new specifications.
4. **No separate concept doc needed.** Being a PRIMITIVE_BLOCKS correction, the specification lives as a section update, not a standalone `what/concepts/REPRESENTATION_MODIFIERS.md`.

**Specifically:** PRIMITIVE_BLOCKS.md § 3.1.3 (`pack`/`unpack`) will note that representation modifiers (struct, boxed, shared, packed) are orthogonal annotations on pack operations. PRIMITIVE_BLOCKS.md § 4 (Exclusions and Decomposition) will note that representation modifiers are excluded from the primitive set and explain their decomposition.

---

### Consequences

- **Positive:**
  - No new concept document — correction lives within existing PRIMITIVE_BLOCKS.md.
  - Primitive set remains minimal (9 primitives) — representation modifiers are annotations, not primitives.
  - Clarifies the boundary between semantics (what a type is) and storage (how it is materialised).
  - Keeps CORE_CONCEPTS.md lean — no entry for a correction-level concept.
- **Negative:**
  - Full specification of representation modifier syntax and semantics is deferred beyond v0.1.
  - The correction does not resolve which modifiers (struct, boxed, shared, atomic, ffi, packed) Orthon will support — it only establishes that they are annotation-level, not primitive-level.
  - Per-use-site representation selection (caller overrides type's default storage) remains an open design question.

---

### Compliance

PRIMITIVE_BLOCKS.md will include a note in § 3.1.3 (`pack`/`unpack`) and § 4 (Exclusions) acknowledging representation modifiers as orthogonal annotations on primitives, not new primitives.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Language feature — full concept doc in `what/concepts/` | Over-specification. The modifiers are annotations on existing primitives, not standalone semantics. A full concept doc would repeat primitive decomposition. |
| New primitives — add `struct_mode`, `boxed_mode`, etc. | Violates orthogonality — each modifier overlaps with either `pack` (struct) or `reference` (boxed). The primitive set would grow without adding new operations. |
| Reject — not needed | Representation modifiers solve a genuine problem (decoupling type semantics from storage strategy). The design space should be preserved via the PRIMITIVE_BLOCKS note. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Flag | Representation modifiers provide significant value (storage control without type duplication), but correction-level treatment defers full specification. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Modifiers as annotations on primitives is logically consistent: they add constraints to existing operations without creating new operations. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Annotation model is simpler than adding 6 new primitives. The correction correctly identifies the minimal solution. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Annotation pattern fits Orthon's existing modifier system (e.g., `fun`/`proc`/`new` are declaration kind modifiers on the `function` primitive). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Representation modifiers describe storage intent — implementation is independent of how the compiler realizes that intent (inline, boxed, arena, etc.). |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Keeping representation modifiers as annotations rather than primitives ensures the primitive set remains stable as new modifiers are added. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs can add `struct`/`boxed` annotations — they are syntactic modifiers with clear semantics. |

**Gates not applied:** None — all applicable given semantic scope.
