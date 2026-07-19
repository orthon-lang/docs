# Unpacking (Destructuring Assignment)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via ECR (Architecture category).

## Issue (Why)

How do you concisely extract values from compound data structures without verbose indexing?

Explicit indexing (`list[0]`, `tuple[1]`) is:
- **Verbose** — repeated index access for multi-element extraction.
- **Brittle** — changing structure order breaks every access site silently.
- **Non-obvious** — `result[0]` reveals nothing about semantics; a destructured binding (`(ok, err)`) does.

The core problem: **binding names to structural positions** should be syntactic, not procedural.

## Principles

1. **Pattern-based binding** — Destructuring follows the same pattern syntax as matching.
2. **Symmetry** — Construction and destruction follow the same syntax.
3. **Exhaustive by default** — Unused elements must be explicitly ignored (`_` or `*rest`).
4. **Nested unpacking** — Compound structures can be destructured recursively in one expression.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Binding Policy | Determines which binding modes are allowed (let, var, mutable) |
| Ignore Policy | Governs syntax for ignoring unused elements (`_`, `*rest`, `..`) |
| Function Argument Policy | Controls destructuring in function parameters |

## Model (What)

Destructuring assignment lets the programmer decompose tuples, records, and arrays into individual bindings using pattern syntax.

```
// Tuple destructuring
let (x, y) = point
let (first, ..rest) = list

// Record destructuring
let {name, age} = person
let {name: n, age: a} = person  // rename bindings

// Nested destructuring
let {address: {city, country}} = user

// Function parameter destructuring
func distance({x, y}: Point) -> Float
```

Key features:
- Tuple, record, and array patterns in assignment context.
- Rest pattern (`..rest` or `*rest`) for remaining elements.
- Nested patterns for recursive decomposition.
- Ignore pattern (`_`) for unused positions.
- Rename syntax for record field destructuring.

## Default Strategy

All compound data types support destructuring assignment. The syntax mirrors pattern matching. Rest binding uses `..rest` syntax. Unused positions require explicit `_` or are a compile-time warning.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Positional only | Only tuples support destructuring; records use `.field` access. |
| Named only | Only records support destructuring; tuples use indexed access. |
| Explicit unpack function | User-defined `unpack` method on types (Python `__iter__`). |

## Open Questions

1. Should destructuring be allowed in `for` loop headers? (e.g. `for {name, age} in people`)
2. Should mutable destructuring (`var {x, y} = point`) be allowed?
3. How does destructuring interact with ownership — does it move or borrow?
4. Should function parameter destructuring support default values?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
