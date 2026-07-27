# EDR-028: Type-Level Null Safety — Option&lt;T&gt; Tracking at the Type Level with Compiler Narrowing

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

EDR-018 established Orthon's null safety model: no `null` sentinel, `Option<T>` as a sum type with `Some(T)` and `None` variants, operators `?.` (optional chaining), `??` (fallback), and `!` (forced unwrap). However, EDR-018 left open the question of **type-level narrowing** — after a check that a value is `Some(T)`, can the compiler track that the value is definitely non-null in the subsequent code path?

Research in `how/concepts/research/essential/TYPE_LEVEL_NULL_SAFETY.md` analysed the distinction between:
1. **Option type at the declaration level** — `Option<T>` vs. `T` is a type-system distinction at declaration time.
2. **Narrowing at the control-flow level** — After `if value is Some(x)`, the compiler knows `value` is `T` inside the branch.

The design tension: narrowing requires flow-sensitive type analysis — the compiler must track type information across control flow edges. This adds compiler complexity but is essential for ergonomic null safety (otherwise every `match` on `Option` would require explicit unboxing even when the programmer knows the value is present).

Type-level null safety builds on the foundation of NULL_SAFETY (EDR-018) by adding the narrowing dimension. It ensures that:
- `Option<T>` and `T` are distinct types — assigning `None` to a non-optional `T` is a compile error.
- After a pattern match or guard that establishes the value is `Some(T)`, the compiler narrows the type to `T` in the matching branch.
- After an explicit check (`if value != None`), the compiler narrows the type within the true branch.
- The narrowing is tracked per-variable — assigning to the variable resets the narrowing.

### Decision

Adopt **type-level null safety with compiler narrowing after checks** with the following design:

1. **`Option<T>` as the null-safety mechanism:** No `null` sentinel. `Option<T>` is a sum type with `Some(T)` and `None` variants. `T` and `Option<T>` are distinct, incompatible types.
2. **Compiler narrowing after pattern match:** After `match value { case Some(x) => ... }`, the compiler narrows `value`'s type to `T` within the `Some` arm.
3. **Compiler narrowing after check:** After `if value != None { ... }`, the compiler narrows `value` to `T` within the true branch.
4. **Compiler narrowing after `?.` chain:** After a `?.` chain, intermediate values that pass the check are narrowed.
5. **Narrowing is per-variable and flow-sensitive:** Narrowing follows control flow. If a variable is reassigned, narrowing resets. Narrowing does not cross function call boundaries (unless inlined).
6. **`?T` syntactic sugar is deferred to Phase 5.** The semantic model of type-level null safety is settled here. If `?T` is adopted as shorthand for `Option<T>`, the concrete syntax is determined in Phase 5 (Syntax).
7. **Narrowing for non-null assertions:** The `!` operator (unwraps with panic if `None`) produces type `T` at the usage site — it does not narrow the original variable.

### Consequences

**Positive:**
- Eliminates null pointer errors entirely — every potential `None` dereference is a compile-time error
- Narrowing after checks eliminates manual unboxing in the common case — ergonomic null safety
- Flow-sensitive tracking preserves safety while reducing boilerplate
- Separate narrowable `Option` from non-narrowable `Result` — distinct roles with distinct semantics

**Negative:**
- Flow-sensitive type analysis adds compiler complexity
- Narrowing across complex control flow (loops, callbacks) may be conservative — some safe programs may require explicit annotation
- The deferred `?T` syntax means Phase 5 must settle the shorthand — until then, `Option<T>` is the only form

### Compliance

1. Assigning `None` to a non-optional variable must be a compile-time error.
2. After a pattern match with `Some(x)`, the scrutinee must be known to be non-null in the matching arm.
3. Narrowing must be conservative: if the compiler cannot prove a value is non-null, it remains `Option<T>` — even if the programmer knows it is safe.
4. The `!` operator must produce type `T` at the call site (it is the explicit "I know this is non-null" escape hatch).
5. Narrowing must reset on variable reassignment.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| No narrowing (always `Option<T>` after match) | Requires explicit `!` unwrap after every `Some` check — defeats the ergonomic benefit of pattern matching |
| Global flow analysis (function-call aware narrowing) | Too complex; cross-function narrowing is fragile and context-dependent. Local narrowing (within a function body) is sufficient |
| Flow typing (TypeScript-style) | Narrowing on type predicates rather than concrete patterns — less precise than pattern-match-based narrowing |
| Non-null types without `Option` (Kotlin `T?` only) | Loses sum-type semantics — `T?` is a nullable reference, not a true optional. `Option<T>` provides pattern matching and exhaustive handling |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes `match opt { case Some(x) => process(x) }` and the compiler knows `x` is non-null inside the arm. No manual unwrap call, no risk of dereferencing `None`. The `!` escape hatch is available for cases where the programmer knows more than the compiler. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Narrowing follows from the semantics of pattern matching: if a value matched `Some(x)`, it cannot be `None` in that branch. Flow-sensitive tracking is well-defined — narrowing is bounded by control flow edges and variable reassignment. No self-referential paradoxes. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Hypothesis: "Narrowing after match reduces unboxing boilerplate without sacrificing safety." Tested by comparing code with and without narrowing — narrowing eliminates explicit `unwrap` calls after every `Some` check. The mechanism is simple: pattern match on `Option` → compiler narrows the type. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Type-level null safety builds on NULL_SAFETY (EDR-018) by adding narrowing semantics. It depends on the type system's ability to track types per-variable with flow sensitivity. It is a Core Language type-system service (Level 1). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Apparent contradiction: narrowing seems tied to a specific type-system implementation. Separation: the *semantic specification* (narrowing after match, per-variable, flow-sensitive) is strategy-independent; the *narrowing algorithm* (def-use analysis, SSA-based, constraint-based) is a Strategy choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "After matching `Some(x)`, the type is narrowed — no manual unboxing needed." Conservative narrowing is safe by construction — the compiler never incorrectly narrows. Flow-sensitive tracking in Rust and Kotlin has proven production stability. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | An LLM generating a match on `Option<T>` can rely on the narrowed type inside the `Some` arm without annotating the narrowing — the compiler handles it. The rule is simple: "match on Some — the inner value is non-null." The Schema Provider exposes optionality per type. |

**Gates not applied:** None — all seven gates are required for a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/TYPE_LEVEL_NULL_SAFETY.md` — Full specification
- `what/concepts/NULL_SAFETY.md` (EDR-018) — Foundation null safety model
- `what/concepts/PATTERN_MATCHING.md` (EDR-025) — Pattern match narrowing
- `how/concepts/research/essential/TYPE_LEVEL_NULL_SAFETY.md` — Research analysis
- `how/DESIGN_PRINCIPLES.md` — Declarative With Static Guarantees, Explicitness

### Supersedes

*None* — this is a new decision complementing EDR-018.
