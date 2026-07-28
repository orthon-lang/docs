# Equality Operators: `==` vs `===`

> Summary of the three-operator equality model in Orthon (EQUALITY.md, EDR-017).
> Discussion from 2026-07-28 — `==` vs `===` distinction and advantages.
>
> **See also:** [`EQUALITY.md`](../what/concepts/EQUALITY.md)

## The Three Operators

| Operator | Name | Semantics | Overridable |
|----------|------|-----------|-------------|
| `===` | Value equality | Structural — recursive field-by-field comparison | No (built-in) |
| `==` | Semantic equality | Domain-specific — defined via `Eq` trait | Yes (via `impl Eq`) |
| `is` | Identity equality | Reference equality — same memory object | No (built-in) |

## Key Difference

- `===` compares **data content** — always structural, never overridden.
- `==` compares **business-logic equivalence** — user-defined via trait, falls back to `===` if not overridden.
- `is` compares **memory location** — only meaningful for mutable/reference-counted data.

```orthon
alice = Person(id=42, name="Alice")
bob   = Person(id=42, name="Bob")

alice == bob     # true  — same ID (custom Eq trait logic)
alice === bob    # false — different fields (name differs)
```

## `===` on Mutable Nested Data

When `===` compares a tuple containing mutable objects (e.g., `list`):

- It performs **recursive structural comparison** — it compares the current contents of each nested object.
- Returns `false` **only when the actual values differ** at the moment of comparison, not trivially because the objects are mutable.

```orthon
a = ([1, 2, 3], "hello")
b = ([1, 2, 3], "hello")
a === b   # true — current contents match

b.0.push(4)
a === b   # false — contents now differ
```

**Compiler warning:** The compiler may warn when `===` is used on a mutable type whose content could change between the comparison and the use of its result. This is a *staleness* risk, not a correctness issue — `===` is always correct at the moment of evaluation.

## Advantages Over Mainstream Languages

1. **Explicitness** — three distinct operators, three distinct semantics. The programmer never guesses which comparison is being made (unlike Python where `==` silently changes meaning depending on `__eq__`).

2. **No type bifurcation** — `===` works identically for all types. No primitive-vs-object split (Java), no type-coercion confusion (JavaScript).

3. **Clean separation** — structural equality (`===`) and semantic equality (`==`) never collide. You can `==` two Person objects by ID while still using `===` to detect field drift.

4. **Guaranteed transitivity** — both `===` and `==` must be transitive. NaN is deferred to the Standard Library (use `Float.isNaN()`) so it cannot violate the invariant.

5. **Named function equivalents** — every operator has an `eq()`, `equal()`, `same()` form per the Named Before Symbolic principle.

## Transitivity Invariant

For any types `T` where `==` or `===` is defined:

```
if a == b and b == c, then a == c must hold
```

The compiler may insert runtime checks in debug mode to detect violations. `is` is transitive by construction.
