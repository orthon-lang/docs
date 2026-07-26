# Null Safety

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language represent the absence of a value without introducing null pointer errors?

Tony Hoare called null references his "billion-dollar mistake." The core problem: `null` conflates "this value is absent" with "this is a valid value of type T." Every reference of type `T` can silently be `null`, turning every dereference into a potential runtime crash.

Languages have tried various mitigations:
- **Nullable annotations** (C# `T?`, Kotlin `T?`) — compiler-checked at call sites, but the underlying issue remains: `null` is a special value that does not obey the type's contract.
- **`Optional` / `Maybe` types** (Rust `Option<T>`, Haskell `Maybe<T>`) — absence is a first-class concept in the type system. No special `null` value.

The winner is clear: **absence must be encoded in the type**, not represented as a sentinel value.

## Principles

1. **No `null` sentinel** — There is no `null` value in the language. Absence is always represented as `None`, which is a value of type `None` — not assignable to any other type.
2. **Option type** — `Option<T>` is the canonical representation of optional values. The compiler enforces unwrapping before use.
3. **Syntactic convenience** — The `?.` (elvis / optional chaining) operator provides ergonomic access to optional values without explicit match.
4. **Fallback chaining** — `??` or `or` provides concise default-value fallback.
5. **Exhaustiveness** — Pattern matching on `Option` must cover both `Some` and `None`, checked by the compiler.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Null Safety Policy | Determines whether `null` exists at all, or is replaced by `Option` |
| Option Type Policy | Governs how `Option<T>` integrates with the type system (monadic, auto-unwrap, or explicit) |
| Propagation Policy | Controls how `None` propagates through chains (`?.` vs manual match) |

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

Key features:
- **`?.`** — optional chaining: if left side is `None`, the expression short-circuits to `None`.
- **`??`** — unwrap or default: `a ?? b` evaluates to `a` if `a` is `Some`, otherwise `b`.
- **`.or(default)`** — combinator equivalent of `??`.
- **`!`** — explicit forced unwrap (panics on `None`). Visible and intentional.
- **`match` exhaustiveness** — the compiler rejects matches that do not cover both `Some` and `None`.

### Interaction with error handling

`Option` and `Result` are distinct concepts with distinct combinators:

| Concept | Meaning | Combinators |
|---------|---------|-------------|
| `Option<T>` | Value may be absent | `?.`, `??`, `.or()`, `.map()`, `.and_then()` |
| `Result<T, E>` | Operation may fail | `?`, `.or_else()`, `.map()`, `.and_then()` |

Both support `map` and `and_then`, enabling uniform composition. The `?` operator works on `Result` only (failures must be diagnosed); `?.` works on `Option` only (absence is normal).

## Default Strategy

`Option<T>` is the only representation of optional values. The compiler rejects direct assignment of `None` to non-optional types. `.unwrap()` is available but discouraged — the default linter flags it.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Nullable types (C#/Kotlin `T?`) | Compiler tracks nullability but `null` still exists. Interop with legacy code is easier. |
| Null Object pattern | No language support; programmer creates sentinel instances per type. |
| Java `Optional<T>` | Library type; not enforced by compiler. Values can still be `null`. |
| Python `None` + `typing.Optional` | No compiler enforcement. None is a valid value of any type at runtime. |

## Open Questions

1. Should `Option<T>` support `?` (Result-style propagation) or only `?.`?
2. Can `None` carry a message? (e.g., `None("user not found")`)
3. Interaction with gradual typing: what happens when `Option<T>` crosses a typed/untyped boundary?
4. Should collections have `find`-like methods that return `Option`, or is that always explicit?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] Other: `how/concepts/research/ERROR_HANDLING.md`, `how/concepts/research/PATTERN_MATCHING.md`
