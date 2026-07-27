# Error Handling

> **✅ ACCEPTED — [EDR-020](../how/decision_records/architecture/EDR-020-error-handling.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Ownership,
> [`GLOSSARY.md`](../GLOSSARY.md) § Result Type, Error Propagation,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How does a program react to failure without crashing or producing silent corruption?

Error handling has evolved through three eras:
1. **Return codes** (`errno` in C) — easy to ignore, no type safety, no composability.
2. **Exceptions** (`try-catch` in Java, C++, Python) — hidden control flow, invisible in signatures, no compiler guarantee of handling.
3. **Monadic types** (`Result<T, E>`, `Option<T>`, `Either<L, R>`) — error becomes part of the return type; the compiler enforces handling.

The core problem: **errors should be visible in the function contract** and **handling should be composable**. Orthon adopts the `Result` model — no exceptions, no unchecked fallibility.

## Principles

1. **Explicit fallibility** — Functions that can fail must declare this in their signature via `Result<T, E>` return type.
2. **Composability** — Error values support transformation (`map`, `and_then`, `or_else`), not just unwrapping.
3. **No silent propagation** — Unhandled `Result` values cause a compile-time error, not a runtime crash.
4. **Recovery before escape** — The language should encourage error recovery before propagation.
5. **No exceptions** — All fallibility is declared in the type system. There is no `try-catch` mechanism.
6. **Exhaustive handling** — Pattern matching on `Result` must cover both `Ok` and `Error` variants.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Error Propagation Policy | Determines that errors propagate explicitly via `?` operator (exception-style propagation rejected) |
| Error Type Policy | Errors are typed via `Result<T, E>` — the error type `E` is a first-class type parameter |
| Recovery Policy | Specifies minimum recovery actions before propagation is allowed; combinators like `or_else` enable recovery |

## Model (What)

Error handling uses a monadic type `Result<T, E>` where `T` is the success type and `E` is the error type. Functions return `Result`; the caller must handle it. There are no runtime exceptions outside of `Result`.

### Core Result Type

```orthon
# Functions that can fail return Result
fun divide(a: Int, b: Int) -> Result<Int, DivisionError>
    if b == 0 then Error(DivisionError.DivisionByZero)
    else Ok(a / b)

# Short-circuit propagation with ? operator
fun read_config(path: String) -> Result<Config, IOError>
    data = fs.read_file(path)?     # ? propagates Error upward
    parse_config(data)              # automatically wrapped in Ok

# Recovery with fallback combinator
config = read_config("app.toml").or_else(|e| default_config())
```

### All Canonical Forms

The `?` operator has equivalent canonical forms:

```orthon
# Form 1: ? operator (concise)
let value = fallible_operation()?

# Form 2: Pattern matching (explicit)
let value = match fallible_operation()
    Ok(v)  -> v
    Error(e) -> return Error(e)

# Form 3: Combinator-based (functional)
let value = fallible_operation().unwrap()
```

All three forms produce the same semantics. The `?` operator is the preferred canonical form for propagation.

### Exhaustive Handling Requirement

The compiler **must** check that pattern matches on `Result<T, E>` cover both `Ok` and `Error` variants.

```orthon
# Compile error: missing Error case
match divide(a, b):
    Ok(result) -> print(result)

# Correct: both variants covered
match divide(a, b):
    Ok(result)  -> print(result)
    Error(err)  -> print("Division failed: {err}")
```

### Result Combinators

| Combinator | Signature | Description |
|------------|-----------|-------------|
| `.map(fn)` | `Result<U, E>` where `fn: T -> U` | Transform success value |
| `.and_then(fn)` | `Result<U, E>` where `fn: T -> Result<U, E>` | Chain fallible operations |
| `.or_else(fn)` | `Result<T, F>` where `fn: E -> Result<T, F>` | Recover from error |
| `.unwrap()` | `T` | Unwrap or panic (use sparingly) |
| `.unwrap_or(default)` | `T` | Unwrap or return default |
| `.unwrap_or_else(fn)` | `T` where `fn: E -> T` | Unwrap or compute default |
| `.is_ok()` | `Bool` | Check if `Ok` |
| `.is_error()` | `Bool` | Check if `Error` |
| `.ok()` | `Option<T>` | Convert `Result` to `Option`, discarding error |
| `.error()` | `Option<E>` | Extract error value if present |

### Named Function Equivalents

Per the Named Before Symbolic principle, each operator has an equivalent named function:

```orthon
fallible_operation()?          # operator form
propagate(fallible_operation())  # named function form

result.map(fn)                  # method form
map(result, fn)                  # free function form

result.or_else(fallback)         # method form
or_else(result, fallback)        # free function form
```

### Interaction with Option

`Result` and `Option` are distinct concepts with distinct combinators:

| Concept | Meaning | Combinators |
|---------|---------|-------------|
| `Result<T, E>` | Operation may fail | `?`, `.or_else()`, `.map()`, `.and_then()` |
| `Option<T>` | Value may be absent | `?.`, `??`, `.or()`, `.map()`, `.and_then()` |

Both support `map` and `and_then` for uniform composition. The `?` operator works on `Result` only (failures must be diagnosed); `?.` works on `Option` only (absence is normal).

### Relationship with Error Union (ERROR_UNION)

The `Result<T, E>` model supports error types that are themselves union types, enabling multiple error sources to be composed. When a function calls multiple fallible operations with different error types, the combined error type is a union of those types. See EDR-020 § Related Concepts for the ERROR_UNION relationship.

```orthon
# Multiple error sources compose naturally
fun process_file(path: String) -> Result<Data, IOError | ParseError>
    raw = fs.read_file(path)?      # may produce IOError
    parsed = parse(raw)?           # may produce ParseError
    return Ok(parsed)

# The combined error type IOError | ParseError is exposed to callers
```

## Default Strategy

All errors use the `Result<T, E>` model. The `?` operator is the canonical propagation mechanism. The compiler rejects unhandled `Result` values. No runtime exceptions outside of `Result`. The default implementation uses stack-unwinding for `?` propagation (similar to Rust's `?` desugaring).

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Panic | Unrecoverable errors abort the program (for invariants, bounds checks) | Invariant violations, programmer errors |
| Checked exceptions | Java-style declared exception list in function signature | Rejected: hidden control flow, no composability |
| Dynamic errors | `Result<T, dyn Error>` — error type is erased, runtime dispatch | Interop layers, plugin systems |
| Error Union | Union of error types for multi-source error handling | Complex systems with multiple error domains |

## Open Questions

1. Should error types carry a mandatory human-readable message or just a machine-readable discriminant?
2. Should `try` blocks exist, or is pattern matching sufficient?
3. Interaction with ownership: does `?` consume or borrow on propagation?
4. How to handle errors from destructors and cleanup code?
5. Should there be a `Result`-specific `?` that differs from `Option`'s `?.`, or can they share a mechanism?

## Decision History

- **Result model over exceptions** adopted. Rationale: Explicit fallibility in signatures, compiler-enforced handling, composable combinator chains. Aligns with Declarative With Static Guarantees.
- **`?` operator for propagation** adopted over explicit match for ergonomics. Rationale: Common pattern (propagate on error) deserves concise syntax. The `?` is syntactically visible — no hidden control flow.
- **No `try-catch`** adopted. Rationale: Pattern matching on `Result` provides exhaustive handling without hidden control flow. Exceptions violate Explicit Semantics.
- **Combinators (map, and_then, or_else)** adopted. Rationale: Enables composable error transformation and recovery without nested match blocks.
- **Error Union relationship** noted for future specification. Rationale: Multiple error sources naturally compose; ERROR_UNION (Plan 04-02) defines the formal semantics.
- **Accepted via EDR-020** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
