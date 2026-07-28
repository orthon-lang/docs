# Type-Level Computation

> **✅ ACCEPTED — [EDR-046](../how/decision_records/architecture/EDR-046-type-level-computation.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`LITERAL_TYPES.md`](LITERAL_TYPES.md),
> [`AST_MACROS.md`](AST_MACROS.md),
> [`COMPILE_TIME_EXECUTION.md`](COMPILE_TIME_EXECUTION.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Type-Level Computation, Type Intrinsic,
> [`ALGEBRAIC_DATA_TYPES.md`](ALGEBRAIC_DATA_TYPES.md)

---

## Issue (Why)

Deriving new types from existing types — creating a DTO type that omits sensitive fields, making all fields optional, extracting property names as a type — is a common pattern in practical programming. TypeScript demonstrates this through conditional types, mapped types, and utility types like `Partial<T>`, `Pick<T, K>`, and `Omit<T, K>`.

However, TypeScript's type-level language is Turing-complete, producing documented failure modes (compiler hangs on recursive conditional types, stack overflows). This directly conflicts with Orthon's minimal-core and LLM-generability pillars.

The core problem: does Orthon want a Turing-complete type-level language, a closed set of built-in intrinsics, or no type-level computation beyond ordinary generics?

## Principles

1. **Closed set of compiler intrinsics** — Orthon provides exactly 8 built-in type-level operations. No user-extensible type-level programming language.
2. **Non-recursive** — No type-level recursion. This eliminates the Turing-completeness failure mode.
3. **No `infer`** — TypeScript-style pattern matching at the type level is not supported. Type manipulation uses fixed, documented intrinsics.
4. **Derive/macro escape hatch** — For custom type-level operations beyond the intrinsic set, use `@macro` functions (EDR-029) that execute at compile time via comptime (EDR-031).
5. **Composable** — Intrinsics compose freely: `Partial<Omit<User, "password">>` is valid.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Computation Policy | Determines which type-level operations are available (the closed set of intrinsics) |
| Type Safety Policy | Ensures type intrinsics produce valid types — no malformed type generation |
| Recursion Policy | Enforces non-recursive evaluation — type-level recursion is rejected by the compiler |

## Model (What)

### Intrinsic reference

```orthon
# KeyOf<T> — union of literal property-name types
KeyOf<User>           # "id" | "name" | "email"

# Pick<T, K> — type with only keys K from T
Pick<User, "id" | "name">   # { id: Int, name: String }

# Omit<T, K> — type with all keys except K from T
Omit<User, "password">      # { id: Int, name: String, email: String }

# Partial<T> — all keys optional
Partial<User>               # { id?: Int, name?: String, email?: String, password?: String }

# Required<T> — all keys required (inverse of Partial)
Required<PartialUser>

# Record<K, V> — type with keys K and values V
Record<"a" | "b", Int>     # { a: Int, b: Int }

# Readonly<T> — all keys read-only
Readonly<User>

# ElementOf<T> — element type of collection
ElementOf<List<Int>>       # Int
ElementOf<Array<String>>   # String
```

### Composition

```orthon
# Compose intrinsics
type CreateUserPayload = Omit<User, "id" | "created_at">
type UpdateUserPayload = Partial<CreateUserPayload>

# With literal types (EDR-043)
KeyOf<User>               # union of string literal types
```

### Macro escape hatch

For operations not covered by the intrinsic set, use a `@macro` function:

```orthon
@macro
fun from_type(source: TypeDef, target: TypeDef) -> ImplBlock
    # Custom type derivation logic at compile time
    # This is a comptime function (EDR-031), not type-level computation

@derive(From<User>)
type CreateUserRequest(name: String, email: String)
```

## Default Strategy

Intrinsics are evaluated by the compiler during type checking. Each intrinsic has a fixed implementation with no user-configurable behaviour. Composition is evaluated left-to-right: `Omit<Partial<User>, "id">` applies `Partial` first, then `Omit`.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Full type-level language (TypeScript) | Turing-complete, user-extensible. Rejected — contradicts Minimal Core and LLM Generability. |
| No type-level computation (macros only) | Acceptable but less ergonomic for simple DTO derivation. Intrinsics provide lighter-weight syntax. |
| Traits-based (Rust `typenum`) | Less ergonomic. Struct-level DTO derivation via traits requires significant ceremony. |
| Comptime-based (Zig) | Already available via EDR-031. Intrinsics provide lighter-weight mechanism for common DTO operations without comptime awareness. |

## Open Questions

1. Should additional intrinsics be added for array/sequence type operations (e.g., `ElementOf<T>`, `LengthOf<T>`)?
2. Should `Omit<T, K>` and `Pick<T, K>` support union-typed keys, or only single literal keys?
3. Should there be a syntax for user-defined type aliases over intrinsics (e.g., `type PublicUser = Omit<User, "password">`)?

## Decision History

- **EDR-046 (2026-07-27):** Type-level computation accepted as Language feature (closed set of 8 compiler intrinsics). Non-recursive. No user-extensible type-level language. Derive/macro escape hatch for custom operations.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [x] `what/concepts/LITERAL_TYPES.md`
- [ ] `what/concepts/AST_MACROS.md`
