# EDR-041: Collection Literal Syntax

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module (Syntax)

---

### Context

Creating a collection and then manually adding each element is verbose, hard to read, and encourages mutation where a literal would suffice. Most modern languages provide compact, readable collection literals as a basic syntax element — `[1, 2, 3]` for lists, `{"a": 1}` for maps, `{1, 2, 3}` for sets — eliminating the create-then-mutate pattern.

Orthon needs to decide:

1. Whether collection literals are Language (new syntax with compiler semantics) or StdLib (syntactic sugar over existing collection constructors).
2. What literal syntax to support (lists, maps, sets).
3. Whether literals are immutable by default with explicit mut opt-in.
4. When to specify the concrete syntax (Phase 5 vs. now).

The research document at `how/concepts/research/important/COLLECTION_LITERAL_SYNTAX.md` explores this in depth.

---

### Decision

**1. Collection literals are classified as StdLib (syntactic sugar).** A collection literal like `[1, 2, 3]` desugars to a constructor call — e.g., `List(1, 2, 3)` — invoking a standard library constructor. No new compiler-level semantics are introduced; the language already supports the necessary primitives (`literal`, `pack`, `call`).

**2. Concrete syntax is deferred to Phase 5 (Syntax).** The following syntax forms are reserved as candidates:
- `[expr, expr, ...]` — list literals
- `{key: value, ...}` — map literals
- `{expr, expr, ...}` — set literals (or `[expr, expr, ...]` with set context)

The exact syntax and disambiguation rules (e.g., `{}` for empty maps vs. scope blocks) will be specified during Phase 5. The semantic model is established here; syntax is a Phase 5 concern.

**3. Immutable by default.** Collection literals produce immutable collections. A mutable variant requires explicit `mut` qualification: `mut[1, 2, 3]` or `MutableList(1, 2, 3)`. This aligns with Orthon's data-first philosophy — data is immutable unless explicitly declared otherwise.

**4. No arbitrary size limits.** Collection literals of any size are supported. The compiler desugars to the appropriate constructor, which may be a variadic function or a builder pattern depending on Implementation Strategy.

---

### Consequences

- **Positive:**
  - No new compiler-level semantics — collection literals desugar to existing StdLib constructors.
  - Syntax deferral to Phase 5 avoids premature commitment to concrete syntax while reserving the semantic space.
  - Immutable-by-default aligns with Orthon's data-first philosophy.
  - No arbitrary size limits — large literals compile efficiently.

- **Negative:**
  - The syntax `{}` for both maps and scope blocks requires disambiguation rules (deferred to Phase 5).
  - Without literal syntax in v0.1, early adopters must use constructor calls (`List(1, 2, 3)` instead of `[1, 2, 3]`).
  - Set literal syntax needs distinct notation from list literals unless type context disambiguates.

---

### Compliance

1. Collection literals must desugar to StdLib constructor calls — no special-case compiler code generation.
2. Immutable-by-default: the desugared constructor must produce an immutable collection.
3. The `mut` qualifier on a literal must select a mutable collection constructor.
4. No additional `import` statement is required for basic literal forms — the desugaring resolves to well-known types in the prelude.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Language feature (new compiler semantics) | Adds unnecessary complexity. Collection construction is a StdLib concern; the compiler should not have special knowledge of collection types. |
| Mutable by default | Violates Orthon's data-first philosophy. Immutable-by-default with explicit `mut` opt-in is more consistent. |
| No literal syntax at all | Acceptable for v0.1 (constructor calls suffice), but collection literals are a universal ergonomic expectation. Deferred to Phase 5. |
| Library-only construction (no special syntax) | Collection literals provide significant ergonomic benefit over constructor calls for common cases. The minimal syntactic overhead is justified. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Collection literals solve a universal ergonomic pain point — no programmer wants to write `list.add(1); list.add(2)` when `[1, 2]` suffices. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Desugaring to constructor calls is internally consistent — no special cases, no new type rules. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | StdLib classification keeps the language core minimal. Literal syntax is syntactic sugar, not new semantics. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | StdLib classification places collection literals in the correct architectural layer (syntax desugaring to StdLib, not core language). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Collection semantics (immutable by default, desugaring to constructor calls) are independent of any specific collection implementation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Collection literals are a universal, stable language feature. The desugaring approach minimizes long-term maintenance burden. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Collection literals are among the most LLM-generable constructs — `[1, 2, 3]` is unambiguous and universally understood. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-041 for per-gate reasoning trail.
