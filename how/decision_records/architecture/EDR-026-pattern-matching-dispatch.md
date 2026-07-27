# EDR-026: Pattern Matching Dispatch — Multimethod Dispatch via Definition-Site Pattern Matching

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

EDR-025 established exhaustive pattern matching as a Core Language construct. EDR-019 established the trait system. Between them, Orthon can match values structurally and express behavioural contracts. However, a gap remains: **multimethod dispatch** — dispatching different implementations based on the type or value of multiple arguments, not just a single receiver.

The imperative crutch analysis in `how/concepts/research/essential/PATTERN_MATCHING_DISPATCH.md` identified the "type-check-then-cast" pattern as a common source of bugs and boilerplate:

```
// Without multimethod dispatch — manual type checking
fun process(a: Expr, b: Expr)
    match a:
        case IntLiteral:
            match b: ...  // nested manual dispatch
```

The design question: does Orthon need dedicated multimethod dispatch syntax, or can this be naturally expressed via existing pattern matching on function arguments?

The argument-site pattern matching approach gives programmers the ability to write function declarations where each parameter is matched against a pattern, and the compiler dispatches to the most specific matching declaration at the call site. This is a natural extension of value-level pattern matching to the function definition level.

Cross-reference: COMMAND_PATTERN_VIA_DELEGATE (Plan 04-07) may provide an alternative mechanism for dynamic dispatch scenarios. Pattern Matching Dispatch should not duplicate what `delegate` can express.

### Decision

Adopt **pattern matching dispatch** — multimethod dispatch where function arguments are pattern-matched at definition site and resolved at call site — with the following design:

1. **`match` declaration form:** A function declaration uses `match` on its parameters to define multiple dispatch variants:

```orthon
fun process(match a: Expr, match b: Expr)
    case IntLiteral(x), IntLiteral(y) => x + y
    case StringLiteral(s), IntLiteral(n) => s.repeat(n)
    case _, _ => error.UnsupportedOperation
```

2. **Definition-site dispatch:** All dispatch variants are declared together in one function declaration. No separate declaration of dispatch variants in different modules.
3. **Call-site resolution:** At the call site, the compiler selects the most specific matching arm based on the runtime types of the arguments. If multiple arms match, either the first matching arm is selected (in source order) or a specificity rule determines the winner.
4. **Exhaustiveness:** The set of dispatch variants for a function must be exhaustive over the possible argument types. The compiler verifies that every combination of argument patterns is covered. A wildcard `_` arm satisfies exhaustiveness.
5. **Specificity rule:** When multiple arms match, the most specific arm wins (the one with the most concrete patterns, not just wildcards). Ties resolve to a compile-time error — the programmer must disambiguate.
6. **Overlap with traits:** When a trait-based dispatch is sufficient (single receiver), prefer trait methods. Pattern matching dispatch is for true multimethod scenarios (dispatch on multiple arguments of different types).
7. **Cross-reference with COMMAND_PATTERN_VIA_DELEGATE (Plan 04-07):** Pattern Matching Dispatch covers argument-based multimethod dispatch. COMMAND_PATTERN_VIA_DELEGATE covers deferred execution and command-like patterns. They are complementary, not overlapping.

### Consequences

**Positive:**
- Eliminates nested type-check-then-cast chains — one declaration form expresses all dispatch variants
- Exhaustiveness checking ensures all argument combinations are covered
- Natural extension of value-level pattern matching to function definition
- Compose with sealed trait hierarchies for type-safe extensibility

**Negative:**
- Specificity rules add compiler complexity — determining "most specific" requires structural comparison of patterns
- Exhaustiveness across multiple arguments multiplies the number of cases to verify
- May encourage type-based dispatch over trait-based polymorphism in cases where traits are more appropriate

### Compliance

1. Every `match` declaration form must be exhaustive over its argument patterns.
2. Specificity resolution must be deterministic — the same call must always resolve to the same arm.
3. Overlap with trait dispatch must be documented — when both a trait method and a pattern dispatch arm match, trait dispatch takes precedence for single-receiver calls.
4. The Schema Provider must expose dispatch arm signatures for LLM querying.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| No multimethod dispatch (traits only for all polymorphism) | Too restrictive — true multimethod dispatch on multiple arguments is not expressible via single-receiver trait dispatch |
| Separate function declarations per dispatch arm (Rust-like) | Scatters dispatch logic across multiple `impl` blocks; definition-site declaration provides local reasoning |
| Visitor pattern (manual dispatch) | Boilerplate-heavy; every new type requires updating the visitor interface. Pattern matching dispatch is the language-level solution |
| Full CLOS-style generic functions | Overly complex for Orthon's design; definition-site dispatch is simpler and more predictable |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer declares `fun process(match a: Expr, match b: Expr) { case IntLiteral(x), IntLiteral(y) => ... }` and the compiler generates the dispatch tree. No manual `instanceof` chains. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Pattern matching dispatch is a direct extension of value-level pattern matching (EDR-025): the same pattern semantics (destructuring, guards, wildcards) applied at the function argument level. Specificity resolution follows from structural pattern comparison. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Hypothesis: "Definition-site pattern matching on arguments replaces nested type checks with a single declaration." The mechanism is simple — patterns on arguments, arms on combinations — and follows established match semantics. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Pattern matching dispatch depends on pattern matching (EDR-025) for pattern semantics and on traits (EDR-019) for type hierarchy. It sits at the same Core Language level as pattern matching (Level 1). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Apparent contradiction: dispatch resolution seems tied to a specific compiler. Separation: the *semantic model* (argument patterns, specificity, exhaustiveness) is strategy-independent; the *dispatch tree compilation* (decision tree generation, vtable vs. static dispatch) is a Strategy choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "Dispatch by matching arguments, verified exhaustive." The multimethod pattern has been proven in CLOS, Julia, and Scala. Definition-site declaration provides local reasoning — all dispatch variants in one place. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | An LLM can generate a dispatch function by enumerating the argument type combinations and their corresponding outputs. The pattern syntax is consistent with value-level matching. The compiler catches coverage gaps. |

**Gates not applied:** None — all seven gates are required for a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/PATTERN_MATCHING_DISPATCH.md` — Full specification
- `what/concepts/PATTERN_MATCHING.md` (EDR-025) — Foundation pattern matching semantics
- `what/concepts/TRAITS.md` (EDR-019) — Trait-based dispatch complement
- `how/concepts/research/essential/PATTERN_MATCHING_DISPATCH.md` — Research analysis
- `how/DESIGN_PRINCIPLES.md` — Orthogonality, Explicitness

### Supersedes

*None* — this is a new decision.
