# Error Handling

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How does a program react to failure without crashing or producing silent corruption?

Error handling has evolved through three eras:
1. **Return codes** (`errno` in C) — easy to ignore, no type safety, no composability.
2. **Exceptions** (`try-catch` in Java, C++, Python) — hidden control flow, invisible in signatures, no compiler guarantee of handling.
3. **Monadic types** (`Result<T, E>`, `Option<T>`, `Either<L, R>`) — error becomes part of the return type; the compiler enforces handling.

The core problem: **errors should be visible in the function contract** and **handling should be composable**.

## Principles

1. **Explicit fallibility** — Functions that can fail must declare this in their signature.
2. **Composability** — Error values must support transformation (`map`, `and_then`, `or_else`), not just unwrapping.
3. **No silent propagation** — Unhandled errors cause a compile-time error, not a runtime crash.
4. **Recovery before escape** — The language should encourage (or require) error recovery before propagation.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Error Propagation Policy | Determines whether errors propagate automatically (exception-style), explicitly (`?` operator), or not at all |
| Error Type Policy | Governs whether errors must be typed (algebraic), stringly-typed, or open-ended |
| Recovery Policy | Specifies minimum recovery actions before propagation is allowed |

## Model (What)

Error handling uses a **monadic type** `Result<T, E>` where `T` is the success type and `E` is the error type. Functions return `Result`; the caller must handle it.

```
fun divide(a: Int, b: Int) -> Result<Int, DivisionError>
    if b == 0 then Error(DivisionError.DivisionByZero)
    else Ok(a / b)

fun read_config(path: String) -> Result<Config, IOError>
    data = fs.read_file(path)?   # ? propagates error upward
    parse_config(data)           # automatically wrapped in Ok

# Short-circuit propagation:
let result = divide(x, y)?      # unwraps Ok or early-returns Error

# Recovery with fallback:
config = read_config("app.toml").or_else(|e| default_config())
```

Key features:
- `?` operator for short-circuit propagation (visible in code).
- Pattern matching for exhaustive error handling.
- Combinators: `map`, `and_then`, `or_else`, `unwrap_or`, `unwrap_or_else`.
- **`.or_else(fallback)`** — recovery combinator. If `Result` is `Error`, calls the fallback function and returns its result. Makes error chains as flat as `try/catch` without hidden control flow.
- No unchecked exceptions — all fallibility is declared.

## Default Strategy

Errors are propagated via an explicit `?` operator. `Result` is the only error type. Unhandled `Result` values produce a compiler warning (configurable to error). No runtime exceptions outside of `Result`.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Panic | Unrecoverable errors abort the program (for invariants, bounds checks). |
| Checked exceptions | Java-style declared exception list in function signature. |
| Dynamic errors | `Result<T, dyn Error>` — error type is erased, runtime dispatch. |

## Open Questions

1. Should error types carry a mandatory human-readable message or just a machine-readable discriminant?
2. Should `try` blocks exist, or is pattern matching sufficient?
3. Interaction with ownership: does `?` consume or borrow on propagation?
4. How to handle errors from destructors and cleanup code?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
