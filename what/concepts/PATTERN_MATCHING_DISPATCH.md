# Pattern Matching Dispatch

> **✅ ACCEPTED — [EDR-026](../how/decision_records/architecture/EDR-026-pattern-matching-dispatch.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`PATTERN_MATCHING.md`](PATTERN_MATCHING.md),
> [`TRAITS.md`](TRAITS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Pattern Matching Dispatch, Multimethod,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Manual type checking with `isinstance` / `instanceof` followed by explicit casting duplicates effort, risks runtime errors, and violates polymorphism. The two-step pattern (check + cast) is both verbose and error-prone.

When a function's behaviour depends on the types of **multiple arguments** simultaneously, single-receiver trait dispatch is insufficient. The programmer must write nested pattern matching chains:

```orthon
// Without multimethod dispatch — cascading checks
fun process(a: Expr, b: Expr)
    match a:
        case IntLiteral(x):
            match b:
                case IntLiteral(y): return x + y
                case StringLiteral(s): return s.repeat(x)
                case _: return error.TypeMismatch
        case StringLiteral(s):
            match b:
                case IntLiteral(n): return s.repeat(n)
                case StringLiteral(t): return s ++ t
                case _: return error.TypeMismatch
```

The core problem: **N-way dispatch on multiple arguments requires exponential nested checks**, making code fragile and unreadable.

## Principles

1. **Definition-site declaration** — All dispatch variants for a multimethod are declared together in one function declaration. No separate `impl` blocks for different argument combinations.
2. **Exhaustiveness across arguments** — The set of dispatch variants must cover all possible argument type combinations (or use a wildcard for the default).
3. **Specificity resolution** — When multiple arms match, the most specific arm (with the most concrete patterns) wins. Ties are a compile-time error.
4. **Complement to traits** — Prefer trait-based dispatch when behaviour depends on a single receiver. Pattern matching dispatch is for true multimethod scenarios.
5. **Pattern syntax consistency** — The pattern syntax in dispatch arms is identical to value-level pattern matching (EDR-025): destructuring, wildcards, guards.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Dispatch Policy | Governs how the compiler resolves which arm to dispatch at each call site |
| Exhaustiveness Policy | Controls verification that all argument type combinations are covered |
| Specificity Policy | Defines the "most specific" comparison between patterns |

## Model (What)

### Match Declaration Form

A function with pattern matching dispatch declares all variants together using the `match` keyword on its parameters:

```orthon
fun process(match a: Expr, match b: Expr)
    case IntLiteral(x), IntLiteral(y) => x + y
    case StringLiteral(s), IntLiteral(n) => s.repeat(n)
    case IntLiteral(x), StringLiteral(s) => s.repeat(x)
    case StringLiteral(s), StringLiteral(t) => s ++ t
    case _, _ => error.UnsupportedOperation
```

### Single Dispatch

A single `match` parameter behaves like a type-based switch:

```orthon
fun handle(match event: Event)
    case Click(x, y) => on_click(x, y)
    case KeyPress(key) => on_key(key)
    case Resize(w, h) => on_resize(w, h)
    case _ => ignore()
```

### Guards in Dispatch Arms

Guards work identically to value-level pattern matching:

```orthon
fun classify(match n: Int)
    case n if n > 0 => "positive"
    case n if n < 0 => "negative"
    case _ => "zero"
```

### Specificity Rules

When multiple arms match, the most specific arm wins. Specificity is determined by:

1. **Pattern specificity** — A concrete pattern (e.g., `IntLiteral(x)`) is more specific than a wildcard (`_`).
2. **Depth specificity** — A deeper nested pattern is more specific than a shallower one.
3. **Guard specificity** — A pattern with a guard is more specific than an unguarded pattern.

If two arms have equal specificity and both match, the compiler reports an ambiguity error.

### Exhaustiveness

The compiler verifies that the dispatch arms cover all possible argument type combinations:

```orthon
fun process(match a: Bool, match b: Bool)
    case true, true => "both true"
    case true, false => "first true"
    case false, true => "second true"
    // Error: (false, false) is not covered
```

Using a wildcard satisfies exhaustiveness:

```orthon
fun process(match a: Bool, match b: Bool)
    case true, true => "both true"
    case _, _ => "not both true"   # covers (true, false), (false, true), (false, false)
```

### Relationship with Trait Dispatch

- **Single-receiver dispatch:** Use trait methods with `impl Trait for Type`.
- **Multi-argument dispatch:** Use pattern matching dispatch.
- **Overlap resolution:** When both a trait method and a dispatch arm match, trait dispatch takes precedence for single-receiver calls.

```orthon
// Prefer trait dispatch for single-receiver behaviour
impl Processable for IntLiteral
    fn process(self) -> Int = self.value

// Use pattern matching dispatch for multi-argument dispatch
fun combine(match a: Expr, match b: Expr) -> Expr
    case IntLiteral(x), IntLiteral(y) => IntLiteral(x + y)
    case _, _ => error.UnsupportedOperation
```

## Default Strategy

Definition-site `match` declaration form with all dispatch variants in one function. First-match precedence as tiebreaker after specificity comparison. Exhaustiveness checked as compile-time error. Wildcard for default cases.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Separate function declarations per arm (Rust-like `impl` blocks) | Scatters dispatch logic across multiple impl blocks. More modular but less local reasoning |
| Full CLOS-style generic functions | Multiple dispatch with method combinations (before/after/around). More powerful but more complex |
| Visitor pattern (manual) | Boilerplate-heavy; every new type requires updating the visitor interface. Pattern matching dispatch is the language-level solution |
| Only trait dispatch (no multimethods) | Insufficient for N-way dispatch — trait dispatch is single-receiver only |

## Open Questions

1. Should dispatch arms support default implementations (like trait default methods) for reusability?
2. How does pattern matching dispatch interact with the Metadata Protocol (`@`)?
3. Should dispatch arms support named function equivalents (per Named Before Symbolic)?
4. What is the interaction between dispatch specificity and generic type parameters in function arguments?

## Decision History

- **Definition-site `match` declaration** adopted over separate `impl` blocks. Rationale: All dispatch variants in one place for local reasoning.
- **Specificity resolution with compile-time ambiguity error** adopted. Rationale: Deterministic dispatch — no silent ambiguity.
- **Exhaustiveness required** adopted over optional coverage. Rationale: Consistency with value-level pattern matching (EDR-025).
- **Accepted via EDR-026** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/PATTERN_MATCHING.md`
- [x] `what/concepts/TRAITS.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
