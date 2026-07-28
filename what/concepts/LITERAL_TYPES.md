# Literal Types

> **✅ ACCEPTED — [EDR-043](../how/decision_records/architecture/EDR-043-literal-types.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`UNION_INTERSECTION_TYPES.md`](UNION_INTERSECTION_TYPES.md),
> [`TYPE_LEVEL_COMPUTATION.md`](TYPE_LEVEL_COMPUTATION.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Literal Type, Widening,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md),
> [`ALGEBRAIC_DATA_TYPES.md`](ALGEBRAIC_DATA_TYPES.md)

---

## Issue (Why)

A specific concrete value — a string like `"GET"`, a number like `42`, a boolean like `true` — can be a type unto itself, inhabited only by that exact value. These **literal types** compose with union types into closed sets: `type Method = "GET" | "POST" | "PUT"` serves as an alternative to ADTs for simple closed sets.

The core problem: should Orthon let literal values act as their own types, and if so, what is the widening rule (when does a literal type widen to its base type)?

With LITERAL_TYPES accepted, both ADTs (EDR-039) and literal type unions are available. ADTs provide stronger guarantees (named variant identity, compiler-enforced exhaustiveness). Literal type unions provide lighter-weight ergonomics for string/number literal sets.

## Principles

1. **Values are singleton types** — A literal value `"GET"` is its own type, inhabited only by that value.
2. **One explicit widening rule** — Immutable bindings preserve literal types; mutable bindings widen to base types. No context-dependent rules.
3. **Primitive scalars only** — Literal types are available for `String`, `Int`, `Float`, and `Bool`. No compound literal types in v0.1.
4. **Composable with union types** — Literal types compose via `|` (EDR-045) into closed sets.
5. **Input to type-level computation** — `KeyOf<T>` (EDR-046) produces a union of literal property-name types.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Definition Policy | Governs how literal types are recognized (parser-level: literal in binding position produces singleton type) |
| Widening Policy | Controls when literal types widen to their base type (mutable bindings only) |
| Narrowing Policy | Governs how literal types narrow in pattern matching (match, is checks) |
| Type Compatibility Policy | Determines when a literal type is assignable to its base type (always) vs. when widening is required |

## Model (What)

### Literal type annotation and inference

```orthon
# Explicit literal type annotation
let method: "GET" = "GET"

# Inferred literal type (immutable binding)
let x = "GET"           # type: "GET" (literal preserved)

# Widened to base type (mutable binding)
var y = "GET"           # type: String (widened — mutable)

# Integer literal type
let port: 80 | 443 = 80

# Boolean literal type
let flag: true = true
```

### Closed set via union composition

```orthon
type Method = "GET" | "POST" | "PUT" | "DELETE"
type Port = 80 | 443 | 8080
type Visibility = "public" | "private"
```

### Narrowing in pattern matching

```orthon
fun handle(method: "GET" | "POST" | "PUT")
    match method:
        case "GET"  => handle_get()
        case "POST" => handle_post()
        case "PUT"  => handle_put()
    # No exhaustiveness guarantee — unlike ADTs, literal type
    # unions can change without compiler warning at match sites.
```

### Coexistence with ADTs

Both mechanisms are available; the programmer chooses based on context:

| Use Case | Recommended Mechanism |
|---|---|
| Internal domain modelling (closed, named variants) | ADT (`type Status = Active | Inactive`) |
| External API boundary (string/number constants) | Literal type union (`type Method = "GET" | "POST"`) |
| Mix of both (internal variant mapped to external string) | ADT + conversion function |

## Default Strategy

Literal types are tracked as singleton types in the type system. Immutable bindings preserve the singleton type. Widening to base type occurs on mutable binding assignment. Narrowing via pattern matching follows the same flow-sensitive rules as TYPE_LEVEL_NULL_SAFETY (EDR-028).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Always widen (TypeScript `let` behaviour) | Every literal immediately widens to its base type. No literal type tracking. Rejected — loses the ergonomic benefit. |
| Never widen (all bindings preserve literal type) | Overly restrictive — mutable bindings would prevent reassignment with different literal values. |
| Context-dependent widening (TypeScript `const`/`let`/`as const`) | Violates LLM Generability Gate. One explicit rule is simpler. |

## Open Questions

1. Should literal types support arithmetic narrowing? (e.g., `let x: 1..10` for range literal types — deferred beyond v0.1.)
2. Should enum member literals (e.g., `Color.Red` as a literal type) be supported?

## Decision History

- **EDR-043 (2026-07-27):** Literal types accepted as Language feature. Single widening rule (`let` preserves, `var` widens). Primitive scalars only.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [x] `what/concepts/UNION_INTERSECTION_TYPES.md`
- [x] `what/concepts/TYPE_LEVEL_COMPUTATION.md`
