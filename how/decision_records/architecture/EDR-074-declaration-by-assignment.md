# EDR-074: Declaration by Assignment — Variable Introduction via First Assignment

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

How does a variable come into existence? Two approaches: explicit declaration (declare then assign) or declaration by assignment (first assignment creates the variable). Orthon balances conciseness with safety through first-assignment-is-declaration combined with compile-time definite assignment analysis.

The semantic decisions (how variables are created, how type is determined, what constitutes a read-before-write error) are language-level. The concrete syntax (keyword choice, annotation style) belongs to Phase 5 (Syntax).

---

### Decision

Declaration by Assignment is a **Language** construct, processed here with a clear Phase 5 boundary:

**Semantic decisions (this EDR):**
- First assignment is declaration — no separate declaration keyword.
- Assigning to an undeclared variable is a compile-time error.
- Type is inferred from the initializer; explicit annotations available.
- Read before write is a compile-time error (definite assignment analysis).
- Variables are immutable by default (`mut` required for reassignment).
- Shadowing permitted with explicit re-declaration keyword.

**Deferred to Phase 5 (Syntax):**
- Concrete syntax for type annotations (`: Type` vs `as Type`).
- Exact keyword for shadowing (`let` vs `var`).
- Whether `mut` is a keyword, modifier, or annotation.
- Concrete syntax for `const` (compile-time constants).

---

### Consequences

- **Positive:**
  - Concise — no `let`/`var` for the common case (first assignment).
  - Safe — definite assignment analysis catches read-before-write errors.
  - Immutable by default — mutation requires explicit `mut`.
  - Shadowing visible — requires an explicit keyword.
- **Negative:**
  - Definite assignment analysis adds compiler complexity.
  - Phase 5 must resolve concrete syntax — risk of design re-opening.

---

### Compliance

- The compiler must implement definite assignment analysis.
- `mut` must be required for variable reassignment.
- Shadowing must require an explicit keyword.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Keyword declaration (Rust `let`) | Adds ceremony for every variable — redundant with first-assignment pattern. |
| Mandatory type annotation (Java) | Verbose — contradicts type inference design (EDR-027). |
| No shadowing | Forces awkward naming (`count`, `count_2`, `count_final`) — poorer code quality. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Write `x = 1` and it works — minimum ceremony. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Definite assignment analysis is well-defined — no ambiguity. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One rule — first assignment declares — covers all cases. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with type inference (EDR-027) and immutability-by-default. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Declaration semantics independent of concrete syntax. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Proven from Python and Julia — well-understood semantics. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs already generate `x = 1` patterns — no new syntax to learn. |
