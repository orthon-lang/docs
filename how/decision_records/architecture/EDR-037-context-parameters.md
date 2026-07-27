# EDR-037: Context Parameters — SEMANTIC_MODEL Correction for Implicit Context Flow

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Semantic Model)

---

### Context

The research document [`CONTEXT_PARAMETERS.md`](../../concepts/research/essential/CONTEXT_PARAMETERS.md) proposes a Dual Parameter Model that separates function parameters into two independent spaces: **data** ("what is being computed") and **context** ("in what environment is the computation happening"), with automatic resolution via a `given`/`using` mechanism (analogous to Scala 3).

The question for this EDR: Where does this concept belong in Orthon's architecture?

**Per D-03 classification rules**, the Decision Pipeline was run (see `DECISION_PIPELINE.md` § Essential — Derived Features, Context Parameters entry). The pipeline reveals:

- **Q2 (Language or Library?):** Context resolution requires compiler support for implicit parameter matching, type-directed `given` resolution, and lexical scoping rules. This is not implementable as a library.
- **Q3 (Existing primitives?):** No — implicit parameter threading is not expressible via the 9-primitive set. It requires compiler-level parameter resolution.
- **Q5 (New semantics?):** Yes — context parameters add implicit-passing semantics with static resolution.

However, the *nature* of the semantics is a correction to the Evaluation and Visibility dimensions of the Semantic Model, not a standalone language feature. Context flow is a cross-cutting concern about how parameters interact with scope resolution — it is a missing piece of the Evaluation dimension (when are context values resolved?) and Visibility dimension (where are `given` instances visible?).

---

### Decision

**Classification: SEMANTIC_MODEL correction.** Context parameters are not a standalone Language feature — they are a correction to the Semantic Model's Evaluation and Visibility dimensions.

**Rationale:**

1. **Cross-cutting concern.** Context flow affects how function parameters are resolved, which is an aspect of Evaluation (when is context supplied?) and Visibility (which `given` instances are in scope?). It does not introduce a new semantic dimension or primitive.
2. **Not a new language construct.** The `using` block and `given` resolution are mechanisms for parameter passing — they refine how existing `function` + `call` primitives interact with scope, not add new primitives.
3. **Already partially covered.** The SEMANTIC_MODEL.md § Evaluation describes parameter evaluation: "Function arguments are evaluated before the call executes." Context parameters add an implicit resolution pathway alongside explicit arguments — this is a refinement, not a contradiction.
4. **No separate concept doc needed.** Being a SEMANTIC_MODEL correction, the specification lives as a section update, not a standalone `what/concepts/CONTEXT_PARAMETERS.md`.

**Architectural impact:** A brief note will be added to SEMANTIC_MODEL.md § Evaluation acknowledging that parameter resolution may include implicit context flow as a cross-cutting concern, with a forward reference to the future language feature specification.

---

### Consequences

- **Positive:**
  - No new concept document required — correction lives within existing SEMANTIC_MODEL.md.
  - SEMANTIC_MODEL.md gains a more complete treatment of Evaluation (parameters may be resolved implicitly).
  - Avoids premature commitment to syntax (`using`, `given` keywords deferred).
  - Keeps CORE_CONCEPTS.md lean — no entry for a correction-level concept.
- **Negative:**
  - Context parameters are less discoverable as a design concept.
  - Full specification (`given`/`using` syntax, resolution priority, ambiguity rules) is deferred to a later phase.
  - The correction is minimal — it flags the concern without specifying the mechanism.

---

### Compliance

SEMANTIC_MODEL.md § Evaluation will include a brief subsection on implicit context flow as a cross-cutting concern. The full context parameter mechanism is deferred beyond v0.1.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Language feature — full concept doc in `what/concepts/` | Over-specification for v0.1. Context parameters are important but not essential for the first version. A SEMANTIC_MODEL correction preserves the design space without committing to syntax. |
| StdLib — dependency injection library | Context resolution requires compiler support (type-directed resolution, scoping) — not implementable as a library. Classification as StdLib would be incorrect per D-03. |
| Reject — not needed | Context parameters solve a real problem (plumbing through call chains). The design space should be preserved via the SEMANTIC_MODEL note, even if implementation is deferred. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Flag | Context parameters provide significant ergonomic value (eliminate manual plumbing), but correction-level treatment defers full value. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Context flow as a cross-cutting concern of Evaluation and Visibility is logically consistent with the existing semantic dimensions. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | A brief note in SEMANTIC_MODEL.md is the simplest correct treatment — adds awareness without commitment. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Cross-cutting concern pattern fits the Semantic Model's existing design (each dimension already has cross-cutting references). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Context resolution semantics (compile-time, scope-directed) are implementation-independent. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Preserving the design space as a SEMANTIC_MODEL note avoids future contradictions. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Context parameters have LLM Generability concerns (implicit resolution can produce non-obvious results), but correction-level treatment defers resolution. |

**Gates not applied:** None — all applicable given semantic scope.
