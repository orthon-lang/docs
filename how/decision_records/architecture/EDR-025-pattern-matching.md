# EDR-025: Pattern Matching — Exhaustive, Expression-Oriented Structural Matching

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Conditional branching is fundamental to programming, but imperative cascades (`if-else if-else`) degrade readability as conditions grow. Nested conditionals create "arrow code." Missed `else` branches cause subtle bugs. The compiler cannot verify that all cases are handled.

Research in `how/concepts/research/essential/PATTERN_MATCHING.md` analysed pattern matching in modern languages:

- **Rust** — Exhaustive `match` with destructuring, guards, binding. Compiler-enforced exhaustiveness. Gold standard for expression-oriented matching.
- **OCaml/Haskell** — Algebraic pattern matching with deep destructuring. Compiler checks completeness.
- **Swift** — `switch` with pattern matching, `if case`, `guard case`. Flexible but less strict exhaustiveness.
- **Kotlin** — `when` expression with smart casts, sealed class exhaustiveness.

The core design tension: exhaustiveness is the primary value proposition (compiler-enforced completeness), but it imposes a strictness cost. Every pattern must be accounted for — including impossible combinations and growing match arms as types evolve.

Pattern matching depends on the trait system (EDR-019): exhaustive matching on enum-like types requires sealed trait hierarchies. Destructuring patterns require knowledge of a type's structural representation — traits define the interface; pattern matching decomposes the structure.

### Decision

Adopt **exhaustive, expression-oriented pattern matching** as a core Orthon language construct with the following design:

1. **`match` as expression:** `match value { case pattern => expr, ... }` produces a value. All arms must produce the same type (or coerce to it).
2. **Exhaustiveness required (compile-time error):** The compiler verifies that all cases are covered. Missing cases are a compile-time error, not a warning. Exhaustiveness applies to:
   - All variants of an enum/sum type
   - `Some`/`None` for `Option<T>`
   - `Ok`/`Error` for `Result<T, E>`
   - All cases of a sealed trait hierarchy
3. **Wildcard `_`:** A catch-all pattern for remaining cases. When `_` is used, the compiler requires it as the last arm but does not error on missing specific cases.
4. **Guards:** Additional predicates on matched values: `case pattern if condition => expr`. Guards are evaluated after the pattern shape matches.
5. **Destructuring:** Patterns decompose compound values — tuple elements, record fields, nested patterns, variant constructors.
6. **Binding:** Matched subvalues are automatically bound to variables within the arm body. `case Some(x) => process(x)` binds `x` to the inner value.
7. **`or` patterns:** `case A | B => expr` matches either pattern.
8. **Pattern order precedence:** First match wins. The compiler may warn about unreachable arms.
9. **Interaction with ownership:** Matching consumes the scrutinee by default; borrowing is explicit (`match &value { ... }`).

### Consequences

**Positive:**
- Compiler-enforced exhaustiveness eliminates a class of bugs — every data shape is accounted for
- `match` as expression composes naturally with assignment, return, and other expression contexts
- Destructuring eliminates manual field extraction boilerplate
- Guards provide conditional matching without nested `if` inside match arms

**Negative:**
- Exhaustiveness requires updating match arms when new enum variants are added — visible cost, but the compiler tells you exactly where
- Complex nested patterns can reduce readability; the compiler must limit reasonable nesting depth
- Interaction with ownership semantics (consuming vs. borrowing the scrutinee) requires clear rules

### Compliance

1. Every `match` expression must be exhaustive — verified at compile time with a specific error message for missing cases.
2. The compiler must not silently accept unreachable arms — a warning or error is required.
3. Destructuring patterns must match the structural representation of the type (pack/unpack decomposition).
4. The Schema Provider must expose sealed trait hierarchies and enum variants for LLM querying (enables exhaustiveness verification in generated code).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Non-exhaustive match (warnings only) | Loses the primary value — compiler-enforced completeness. Warnings are frequently ignored. |
| Statement-oriented match (no expression value) | Reduces composability. Expression-oriented matching is the modern standard (Rust, OCaml, Kotlin). |
| No guards (pure structural matching only) | Too restrictive — guards handle cases where matching depends on value content, not just shape. |
| No `or` patterns (requires separate arms) | Increases boilerplate for adjacent cases with identical bodies. `or` patterns are well-established (Rust, Scala). |
| Pattern matching via library (no language support) | Exhaustiveness checking and destructuring semantics require compiler support — not expressible via composition of primitives. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes `match result { case Ok(data) => process(data); case Error(err) => handle(err) }` and the compiler guarantees every path is handled. No more "unhandled case" runtime bugs. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Pattern matching follows from the sum type model: an enum with N variants requires N+1 cases (N variants + optional wildcard). Guards extend the pattern after structural matching. Or patterns are syntactic sugar for duplicate arms. First-match precedence is unambiguous. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Hypothesis: "Pattern matching replaces if-else chains, manual destructuring, and runtime type checking with a single expression." Verified by comparing code examples — the match variant is consistently shorter, more readable, and eliminates entire classes of bugs. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Pattern matching depends on the trait system (EDR-019) for sealed trait exhaustiveness and on the data model (pack/unpack) for destructuring. The match expression is a Core Language construct (Level 1). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Apparent contradiction: exhaustiveness checking seems tied to a specific compiler implementation. Separation: the *semantic model* (exhaustive matching, destructuring semantics, guards) is strategy-independent; the *decision tree compilation* (how the compiler generates efficient branching code) is a Strategy choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "Match values by structure, guaranteed complete by the compiler." Pattern matching is a mature, stable feature in Rust (2015+), OCaml (1996+), and Haskell (1990+). Exhaustiveness is a net maintainability gain — it prevents the most common enum/sum-type bugs. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | An LLM can reliably produce exhaustive match expressions when given the sealed trait hierarchy or enum variant list. The rule is simple: one arm per variant + wildcard. The Schema Provider makes variants and their types available for LLM querying. The compiler catches missing cases — safe to generate and iterate. |

**Gates not applied:** None — all seven gates are required for a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/PATTERN_MATCHING.md` — Full specification
- `what/concepts/TRAITS.md` (EDR-019) — Sealed trait exhaustiveness
- `what/concepts/PATTERN_MATCHING_DISPATCH.md` (EDR-026) — Multimethod dispatch via pattern matching
- `how/concepts/research/essential/SMART_CAST.md` — Type narrowing after match
- `how/concepts/research/essential/PATTERN_MATCHING.md` — Research analysis
- `how/DESIGN_PRINCIPLES.md` — Explicitness, Declarative With Static Guarantees

### Supersedes

*None* — this is a new decision.
