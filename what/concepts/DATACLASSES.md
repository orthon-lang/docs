# Dataclasses

> **✅ ACCEPTED — [EDR-042](../how/decision_records/architecture/EDR-042-dataclasses.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`AST_MACROS.md`](AST_MACROS.md) (derive mechanism),
> [`EQUALITY.md`](EQUALITY.md) (structural equality),
> [`GLOSSARY.md`](../GLOSSARY.md) § Dataclass, Derive,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

A large fraction of types in any codebase are passive data carriers — configuration objects, API response DTOs, database rows, value objects. Without syntactic support, each such type requires many lines of mechanically identical code: constructors, accessors, equality, hashing, and string representation.

Orthon eliminates this boilerplate through the existing `@derive` mechanism (EDR-029). A "dataclass" is simply a type annotated with `@derive(init, eq, repr, hash)`.

## Principles

1. **No dedicated keyword** — Dataclasses are a pattern expressed through the existing `@derive` mechanism, not a new language construct.
2. **Immutability by default** — Fields in a dataclass-style type are immutable by default (read-only access). Explicit `var` fields are available for mutable carriers.
3. **Structural equality by derive** — The `eq` derive generates structural `===` (EDR-017) comparison of all fields.
4. **Copy-with-modify** — The `with` expression creates a new instance with specified fields changed. No mutation of the original.
5. **StdLib classification** — Derive implementations are standard library macros registered in the derive registry, not compiler built-ins.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Derivation Policy | Controls which derives are automatically applied to dataclass-like types |
| Comparison Policy | Determines how equality derives use `===` (EDR-017) for recursive field comparison |
| Representation Policy | Controls how dataclass fields are laid out in memory (inline, packed) |
| Mutability Policy | Governs immutable-by-default semantics for dataclass fields |

## Model (What)

A dataclass is a type declaration annotated with `@derive(init, eq, repr, hash)`:

```orthon
@derive(init, eq, repr, hash)
type Point(x: Float, y: Float)

# Usage
p1 = Point(x: 1.0, y: 2.0)       # named construction
p2 = Point(1.0, 2.0)             # positional shorthand
p1.x                              # field access: 1.0

# Structural equality (from eq derive)
p1 === p2                         # true (same field values)

# String representation (from repr derive)
print(p1)                         # "Point(x: 1.0, y: 2.0)"

# Copy with modification (from init derive)
p3 = Point(x: 3.0, y: 2.0)       # new instance

# Hashing (from hash derive)
hash_map.insert(p1, "origin")
```

### The `with` expression

The `with` expression creates a new instance with specified fields changed:

```orthon
p2 = Point(x: 3.0, y: 4.0)
p3 = p2 { y = 5.0 }              # copy with y changed
# p3 is Point(x: 3.0, y: 5.0)
# p2 is unchanged
```

Semantics: `with` evaluates the original expression, then creates a new value of the same type with the specified fields replaced. This is a compiler-recognized intrinsic (not syntactic sugar) — it generates a field-by-field copy with selective reassignment.

### Custom derives

Users can define custom derive macros (EDR-029) and compose them with standard dataclass derives:

```orthon
@derive(init, eq, repr, hash, FromJson, ToJson)
type User(name: String, email: String)
```

## Default Strategy

The compiler generates field-by-field comparison for `eq`, field-by-field copy for `with`, and introspection-based string generation for `repr`. All derives produce type-safe, monomorphised code with no reflection overhead.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Manual implementation | Programmer writes `impl` blocks manually for full control. No derives. |
| Selective derives | `@derive(eq)` only — opt in to specific derivations. |
| No `with` expression | Copy-with-modify via manual construction: `Point(x: p2.x, y: 5.0)`. |

## Open Questions

1. Should `@derive(init, eq, repr, hash)` have a shorthand alias like `@dataclass`?
2. Should the `with` expression support nested field updates (`p2 { address.city = "Paris" }`)?
3. How do dataclass derives interact with generic types — does `@derive(eq)` on `type Pair<T>(a: T, b: T)` require `T: Eq`?

## Decision History

- **EDR-042 (2026-07-27):** Dataclasses accepted as StdLib pattern via `@derive`. No dedicated keyword. `with` expression accepted as compiler intrinsic.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/concepts/AST_MACROS.md`
- [ ] `what/concepts/EQUALITY.md`
- [ ] `what/SYNTAX.md` (with expression syntax)
