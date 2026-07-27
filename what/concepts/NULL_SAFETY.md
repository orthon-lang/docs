# Null Safety

> **✅ ACCEPTED — [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Ownership,
> [`GLOSSARY.md`](../GLOSSARY.md) § Option Type,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How does a language represent the absence of a value without introducing null pointer errors?

Tony Hoare called null references his "billion-dollar mistake." The core problem: `null` conflates "this value is absent" with "this is a valid value of type `T`." Every reference of type `T` can silently be `null`, turning every dereference into a potential runtime crash.

Orthon's solution: **absence must be encoded in the type**, not represented as a sentinel value.

## Principles

1. **No `null` sentinel** — There is no `null` value in the language. Absence is always represented as `None`, which is a value of type `None` — not assignable to any other type.
2. **Option type** — `Option<T>` is the canonical representation of optional values. The compiler enforces unwrapping before use.
3. **Syntactic convenience** — The `?.` (elvis / optional chaining) operator provides ergonomic access to optional values without explicit match.
4. **Fallback chaining** — `??` provides concise default-value fallback.
5. **Exhaustiveness** — Pattern matching on `Option` must cover both `Some` and `None`, checked by the compiler.
6. **Explicit unwrap** — Forced unwrap (`!`) is syntactically visible and intentional. No silent unwrapping.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Null Safety Policy | Determines that `null` does not exist — replaced by `Option` |
| Option Type Policy | Governs how `Option<T>` integrates with the type system (monadic, explicit unwrap) |
| Propagation Policy | Controls how `None` propagates through chains (`?.` vs. manual match) |

## Model (What)

There is **no `null`** in the language. Absence is represented by `Option<T>` — a sum type with variants `Some(T)` and `None`. `None` is of type `None`, which is assignable to `Option<T>` but not to any non-optional type `T`.

```orthon
# Functions that may not return a value use Option
user = db.find_user(42)         # returns Option<User>

# Elvis operator — unwrap or continue as None
display_name = user?.name       # returns Option<String>

# Fallback with default
name = user?.name ?? "Guest"    # unwrap or use default
# or using or combinator:
name = user.name.or("Guest")

# Pattern matching — compiler checks exhaustiveness
match user:
    Some(u) -> print(u.name)
    None    -> print("Guest")

# Forced unwrap (panics if None — use sparingly)
raw_user = user!                # explicit, not silent
```

### Operators and Combinators

| Syntax | Name | Behaviour |
|--------|------|-----------|
| `?.` | Elvis / optional chaining | If left side is `None`, expression short-circuits to `None`. Otherwise, unwraps and continues. |
| `??` | Unwrap or default | `a ?? b` evaluates to `a` if `a` is `Some`, otherwise `b`. |
| `!` | Forced unwrap | Unwraps `Option<T>` to `T`. Panics if `None`. Must be syntactically visible. |
| `.or(default)` | Combinator | Equivalent to `??`. Named function form. |
| `.map(fn)` | Combinator | Transform `Some(T)` via function, pass through `None`. |
| `.and_then(fn)` | Combinator | Chain operations that return `Option`. |

### Interaction with Error Handling

`Option` and `Result` are distinct concepts with distinct combinators:

| Concept | Meaning | Combinators |
|---------|---------|-------------|
| `Option<T>` | Value may be absent | `?.`, `??`, `.or()`, `.map()`, `.and_then()` |
| `Result<T, E>` | Operation may fail | `?`, `.or_else()`, `.map()`, `.and_then()` |

Both support `map` and `and_then`, enabling uniform composition. The `?` operator works on `Result` only (failures must be diagnosed); `?.` works on `Option` only (absence is normal).

### Exhaustiveness Requirement

The compiler **must** check that pattern matches on `Option<T>` cover both `Some` and `None` variants. A match that omits either variant is a compile-time error.

```orthon
# Compile error: missing None case
match user:
    Some(u) -> print(u.name)

# Correct: both variants covered
match user:
    Some(u) -> print(u.name)
    None    -> print("Guest")
```

## Default Strategy

`Option<T>` is the only representation of optional values. The compiler rejects direct assignment of `None` to non-optional types. `.unwrap()` has a named function equivalent but is not an operator — the `!` operator is the canonical forced unwrap. The default linter flags unnecessary uses of `!` where `??` or pattern matching would suffice.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Nullable types (C#/Kotlin `T?`) | Compiler tracks nullability but `null` still exists. Rejected: violates No `null` sentinel principle. |
| Null Object pattern | No language support; programmer creates sentinel instances per type. Rejected: ad-hoc, no compiler enforcement. |
| Java `Optional<T>` | Library type, not enforced by compiler. Values can still be `null`. Rejected: violates Exhaustiveness principle. |
| Python `None` + `typing.Optional` | No compiler enforcement. `None` is a valid value of any type at runtime. Rejected: violates Declarative With Static Guarantees. |

## Open Questions

1. Should `Option<T>` support `?` (Result-style propagation) or only `?.` and explicit combinators?
2. Should `None` carry a message or context (e.g., `None("user not found")`)?
3. Interaction with gradual typing: what happens when `Option<T>` crosses a typed/untyped boundary?
4. Should collections have `find`-like methods that return `Option`, or is that always explicit at the call site?

## Decision History

- **No `null` sentinel** adopted over nullable types. Rationale: Eliminates the root cause of null pointer errors at the language level. Aligns with Declarative With Static Guarantees.
- **`Option<T>` as sum type** adopted over nullable annotations. Rationale: Compiler-enforced exhaustiveness prevents the entire class of null-dereference bugs.
- **`?.` and `??` operators** adopted over explicit match for ergonomics. Rationale: Common patterns (optional chaining, fallback defaults) deserve concise syntax per Principle of Parsimony.
- **`!` for forced unwrap** adopted. Rationale: Explicitness principle — the panic risk must be syntactically visible.
- **Accepted via EDR-018** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/SEMANTIC_MODEL.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
