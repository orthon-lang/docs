# Union and Intersection Types

> **✅ ACCEPTED — [EDR-045](../how/decision_records/architecture/EDR-045-union-intersection-types.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`ALGEBRAIC_DATA_TYPES.md`](ALGEBRAIC_DATA_TYPES.md),
> [`LITERAL_TYPES.md`](LITERAL_TYPES.md),
> [`TYPE_LEVEL_NULL_SAFETY.md`](TYPE_LEVEL_NULL_SAFETY.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Union Type, Narrowing,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Union types (`A | B`) and intersection types (`A & B`) are structural, set-theoretic type combinators. They operate over arbitrary types with no tag, discriminant, or prior variant declaration required.

This is distinct from Algebraic Data Types (EDR-039). ADTs require declaring a named type up front with named variants. Structural union types require no named-type declaration, no tag, and apply to any two existing types, including types the union's author does not own.

The core problem: should Orthon have a structural union/intersection type combinator distinct from ADTs, or should ADTs remain the sole "one of several" mechanism?

## Principles

1. **Union types accepted; intersection types NOT accepted for v0.1.** `A | B` is a structural, untagged union. `A & B` is redundant with product types.
2. **Named types only** — Union members must be named types or literal types (EDR-043). Anonymous structural shapes are not supported in v0.1.
3. **Untagged — no discriminant** — The runtime representation of a union value is the value itself. No tag, no boxing.
4. **Narrowing via match/is** — Narrowing follows the same flow-sensitive rules as TYPE_LEVEL_NULL_SAFETY (EDR-028).
5. **No exhaustiveness** — Unlike ADTs, the compiler does not guarantee exhaustive handling of union members.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Compatibility Policy | Determines assignment rules for union types (A is assignable to A|B; narrowing required for destructuring) |
| Narrowing Policy | Controls flow-sensitive type narrowing on union members (match, is checks) |
| Memory Layout Policy | Union type at runtime is just the value — no tag. Layout follows the active member's type. |

## Model (What)

### Union type declaration

```orthon
type ID = String | Int
type Method = "GET" | "POST" | "PUT"   # literal type union (EDR-043)
type MaybeError = Result | ErrorTag
```

### Narrowing via match

```orthon
fun print_id(id: String | Int)
    match id:
        case s: String => print("ID: $s")
        case i: Int    => print("ID: $(i.to_string())")
```

### Narrowing via is check

```orthon
if id is String:
    print("String ID: $(id)")      # id narrowed to String in this branch
```

### Function parameters

```orthon
fun connect(host: String | IPAddress) -> Result<Connection, Error>
    # Connect using either hostname or IP address
```

### Collection element types

```orthon
let items: List[String | Int] = ["hello", 42, "world"]
```

### Interaction with ADTs

Union types and ADTs serve different use cases:

```orthon
# ADT — named variants, exhaustive matching, compiler-enforced
type Payment = Cash | CreditCard(card_number: String) | Crypto(txid: String)

# Union type — ad-hoc composition at point of use, no declaration
type PaymentInput = String | Int     # Either a card number or a transaction ID
```

## Default Strategy

Union type values use the same memory representation as the active member — no boxing, no tag. Narrowing is flow-sensitive following EDR-028 rules. The compiler generates dispatch code at pattern match sites based on member types.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Tagged union (ADTs only) | All "one of several" types must be declared ADTs. No ad-hoc unions. Rejected — misses ergonomic benefit. |
| Open structural unions (TypeScript) | Any types, including anonymous shapes. Rejected — exhaustiveness impossible, LLM reasoning cost too high. |
| Restricted to StdLib types only | Artificial restriction. Union types are most useful with user-defined types. |

## Open Questions

1. Should union types support recursive narrowing (narrow a union member, then narrow its fields)?
2. Should there be a `T | None` shorthand (like TypeScript's `T | null`), or is `Option<T>` sufficient?
3. Does the no-exhaustiveness rule for union types need a compiler warning for matches that appear comprehensive?

## Decision History

- **EDR-045 (2026-07-27):** Union types accepted as Language feature. Intersection types NOT accepted (deferred post-v0.1). Named types only. No exhaustiveness guarantee.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [x] `what/concepts/LITERAL_TYPES.md`
- [x] `what/concepts/ALGEBRAIC_DATA_TYPES.md`
