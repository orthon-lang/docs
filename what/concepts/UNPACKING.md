# Unpacking — Destructuring Assignment with Pattern Syntax

> **✅ ACCEPTED — [EDR-055](../how/decision_records/architecture/EDR-055-unpacking.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`PATTERN_MATCHING.md`](PATTERN_MATCHING.md),
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md),
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Identity,
> [`GLOSSARY.md`](../GLOSSARY.md) § Destructuring, Unpacking

---

## Issue (Why)

How do you concisely extract values from compound data structures without verbose indexing?

Explicit indexing (`list[0]`, `tuple[1]`) is:
- **Verbose** — repeated index access for multi-element extraction.
- **Brittle** — changing structure order breaks every access site silently.
- **Non-obvious** — `result[0]` reveals nothing about semantics; `let (ok, err) = result` does.

The core problem: **binding names to structural positions** should be syntactic, not procedural.

Orthon's PRIMITIVE_BLOCKS (EDR-016) establishes `pack`/`unpack` as the fundamental value composition/decomposition primitive. Destructuring assignment is the syntactic expression of `unpack` — decomposing values into named bindings.

## Principles

1. **Symmetry with `pack`/`unpack`** — Construction and destruction use the same syntax. `(x, y)` creates a tuple; `let (x, y) = ...` decomposes one. This follows Representation Symmetry (DESIGN_PRINCIPLES.md).

2. **Pattern-based binding** — Destructuring follows the same pattern syntax as pattern matching (EDR-025).

3. **Consistency with PATTERN_MATCHING** — The same patterns (tuple, record, rest, ignore, nested) work in both `match` and destructuring contexts.

4. **Exhaustive by default** — Unused elements must be explicitly ignored (`_` or `..rest`).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Binding Policy | Determines binding modes (let, var, mutable) for destructured values |
| Ignore Policy | Governs syntax for unused elements (`_`, `..`, `..rest`) |
| Function Argument Policy | Controls destructuring in function parameters |
| Ownership Policy | Determines move vs. borrow semantics for destructured elements |

## Model (What)

Destructuring assignment lets the programmer decompose tuples, records, and arrays into individual bindings using pattern syntax.

### Tuple Destructuring

```orthon
let point = (10, 20)
let (x, y) = point               # x = 10, y = 20

# Rest pattern
let list = (1, 2, 3, 4, 5)
let (first, ..rest) = list       # first = 1, rest = (2, 3, 4, 5)

# Ignore pattern
let (x, _, z) = triple           # middle value ignored
```

### Record Destructuring

```orthon
let person = {name: "Alice", age: 30, city: "Zurich"}
let {name, age} = person          # name = "Alice", age = 30
let {name: n, age: a} = person    # rename bindings: n = "Alice", a = 30
let {name, ..} = person           # only extract name
```

### Nested Destructuring

```orthon
let user = {
    name: "Alice",
    address: {city: "Zurich", country: "Switzerland"}
}
let {address: {city, country}} = user   # city = "Zurich"
let ((a, b), c) = nested_tuple
```

### Function Parameter Destructuring

```orthon
fun distance({x, y}: Point) -> Float
    return sqrt(x * x + y * y)

fun process_first((x, _): Pair) -> x

fun handle_users({name, age}: Person)
    print("{name} is {age} years old")
```

### Destructuring in `for` Loops

```orthon
for {name, age} in people:
    print("{name} is {age} years old")

for (index, value) in items.enumerate():
    print("Index {index}: {value}")
```

### Mutable Destructuring

```orthon
var (x, y) = point      # x and y are mutable
let {mut name, age} = person  # name is mutable, age is immutable
```

### Desugaring

All destructuring forms desugar to `pack`/`unpack` primitives and field access:

```orthon
# Tuple desugaring:
let (x, y) = point
# → let tmp = point; let x, y = unpack(tmp)

# Record desugaring:
let {name, age} = person
# → let name = person.name; let age = person.age

# Nested desugaring:
let {address: {city, country}} = user
# → let address_tmp = user.address; let city = address_tmp.city; let country = address_tmp.country
```

## Default Strategy

All compound data types support destructuring assignment. The syntax mirrors PATTERN_MATCHING (EDR-025). Rest binding uses `..rest` syntax. Unused positions require explicit `_` (compile-time warning if unused binding is not prefixed with `_`).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Positional only | Only tuples support destructuring; records use `.field` access. Simpler but asymmetric. |
| Named only | Only records support destructuring; tuples use indexed access. |
| Explicit unpack function | User-defined `unpack` method on types (Python `__iter__`). More flexible but less predictable. |

## Open Questions

1. Should mutable destructuring (`var {x, y} = point`) be allowed in all contexts?
2. How does destructuring interact with ownership — does it move or borrow by default?
3. Should function parameter destructuring support default values (`fun f({x = 0, y = 0}: Point)`)?
4. Can destructuring be composed with nullable types — `let (x, y) = optional_point`?

## Decision History

- **2026-07-27** — Accepted via EDR-055. Classification: Language. Destructuring syntax matching pack/unpack symmetry (PRIMITIVE_BLOCKS). All forms desugar to `pack`/`unpack` primitives — no new runtime semantics.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/PATTERN_MATCHING.md` (referenced)
- [ ] `what/SYNTAX.md`
- [ ] `what/SEMANTIC_MODEL.md`
