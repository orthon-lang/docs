# Collection Literal Syntax

> **✅ ACCEPTED — [EDR-041](../how/decision_records/architecture/EDR-041-collection-literal-syntax.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`GLOSSARY.md`](../GLOSSARY.md) § Collection Literal,
> [`SYNTAX.md`](../SYNTAX.md) (Phase 5 — concrete syntax),
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Creating a collection and then manually adding each element is verbose, hard to read, and encourages mutation where a literal would suffice. The create-then-mutate pattern — instantiating an empty collection and calling `.add()` or `.put()` for each element — is a universal imperative crutch.

Orthon eliminates this pattern by providing compact, readable collection literals as syntactic sugar over standard library constructors.

## Principles

1. **StdLib, not Language** — Collection literals desugar to standard library constructor calls. No new compiler-level semantics.
2. **Immutable by default** — Collection literals produce immutable collections. Mutable variants require explicit `mut` qualification.
3. **No arbitrary size limits** — Collection literals of any size are supported.
4. **Syntax deferred to Phase 5** — The semantic model is established here; concrete syntax is a Phase 5 concern.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Representation Policy | Determines how collection literals are materialised (statically sized array vs. dynamic collection) |
| Allocation Policy | Controls where literal data is allocated (static data section, stack, heap) |
| Mutability Policy | Governs immutable-by-default semantics and `mut` opt-in |

## Model (What)

A collection literal is syntactic sugar that desugars to a standard library constructor call:

```orthon
# List literal (concrete syntax deferred to Phase 5)
[1, 2, 3]              # desugars to List(1, 2, 3)

# Map literal
{"a": 1, "b": 2}       # desugars to Map("a" -> 1, "b" -> 2)

# Set literal
{1, 2, 3}              # desugars to Set(1, 2, 3)

# Mutable variant (explicit opt-in)
mut[1, 2, 3]           # desugars to MutableList(1, 2, 3)
```

### Candidate syntax forms for Phase 5

The following syntax forms are reserved as candidates, subject to Phase 5 agreement:

| Collection | Candidate Syntax | Notes |
|---|---|---|
| List | `[expr, expr, ...]` | Unambiguous — brackets not used elsewhere |
| Map | `{key: value, ...}` | Disambiguation from scope `{}` required |
| Set | `{expr, expr, ...}` or inferred from context | May share `[]` syntax disambiguated by element type |

### Desugaring rules

- `[a, b, c]` → `List(a, b, c)` — constructor with variadic arguments.
- `{k1: v1, k2: v2}` → `Map(k1 -> v1, k2 -> v2)` — pair constructor.
- `{a, b, c}` → `Set(a, b, c)` — or `HashSet(a, b, c)` depending on ordered/unordered semantics.
- `mut[1, 2, 3]` → `MutableList(1, 2, 3)` — mutable variant.

## Default Strategy

The desugared constructor call uses the default Implementation Strategy's collection types. Lists desugar to `List` (immutable, ordered), maps to `Map` (immutable, key-value), sets to `Set` (immutable, unordered).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Builder desugaring | Large literals desugar to a builder pattern for efficiency, not a single constructor call with many arguments. |
| Static allocation | Literals with constant elements are allocated in the static data section (no runtime construction cost). |
| Inline array | List literals of known size compile to stack-allocated arrays. |

## Open Questions

1. How does `{}` disambiguation work between empty maps and empty scope blocks? (Phase 5 decision.)
2. Should set literals use `{}` syntax or should context-based inference from `[]` be sufficient?
3. What is the maximum literal size before the compiler switches from constructor to builder desugaring?

## Decision History

- **EDR-041 (2026-07-27):** Collection literals accepted as StdLib (syntactic sugar). Syntax deferred to Phase 5. Immutable by default.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [ ] `what/SYNTAX.md` (Phase 5)
- [x] `what/GLOSSARY.md`
- [ ] `how/concepts/research/essential/DATA_MODEL.md`
- [ ] `how/concepts/research/essential/MUTABILITY.md`
