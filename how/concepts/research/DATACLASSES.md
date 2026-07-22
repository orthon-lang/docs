# Dataclasses

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

How does a language reduce the boilerplate of defining a type whose primary purpose is to hold data — a simple record — without forcing the programmer to manually write constructors, accessors, equality, hashing, string representation, and copy methods?

A large fraction of types in any codebase are **passive data carriers**: configuration objects, API response DTOs, database rows, command/event messages, value objects. In languages without syntactic support for this pattern, each such type requires many lines of mechanically identical code:

```java
// Java — ~30 lines for a 2-field data carrier
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() { return x; }
    public int y() { return y; }

    @Override
    public boolean equals(Object o) { ... }
    @Override
    public int hashCode() { ... }
    @Override
    public String toString() { return "Point(x=" + x + ", y=" + y + ")"; }
}
```

Python's `@dataclass` decorator (introduced in 3.7) solves this by automatically synthesising `__init__`, `__repr__`, `__eq__`, `__hash__`, and optionally `__lt__`/`__le__`/`__gt__`/`__ge__` and `__del__` from field declarations:

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

# __init__, __repr__, __eq__, __hash__ auto-generated
```

| Language | Mechanism | Auto-init | Auto-eq | Auto-repr | Auto-hash | Immutability | Copy with changes |
|---|---|---|---|---|---|---|---|
| **Python** | `@dataclass` decorator | Yes | Yes (opt-out) | Yes (opt-out) | Yes (by policy) | `frozen=True` | `dataclasses.replace()` |
| **Java** | Records (Java 16+) | Yes | Yes (component-based) | Yes | Yes | Yes — implicitly final | Manual construction |
| **Kotlin** | `data class` | Yes | Yes (structural) | Yes | Yes | `val` vs `var` per field | `.copy()` built-in |
| **C#** | Records (C# 9+) / `record struct` | Yes | Yes (structural) | Yes (with `ToString`) | Yes | `record class` positional | `with { }` expression |
| **Rust** | `#[derive(...)]` on struct | No (manual `fn new`) | `#[derive(PartialEq)]` | `#[derive(Debug)]` | `#[derive(Hash)]` | Default (no `mut`) | Manual with `..` spread |
| **Scala** | `case class` | Yes | Yes (structural) | Yes | Yes | Yes (fields are `val`) | `.copy()` built-in |

The core problem: **should Orthon include a syntactic form for declaring data-aggregate types that automatically derives the common structural methods, and if so, what set of methods should be provided?**

## Principles

1. **Minimal core, maximal stdlib** (from `why/MANIFESTO.md`) — The mechanism for deriving common type operations should be a library concept (like a macro or derive attribute), not a hard-coded language keyword.
2. **Explicitness** (from `DESIGN_PRINCIPLES.md`) — The programmer should be able to see which methods are auto-derived and override any of them individually. No hidden generation.
3. **Immutability as a first-class axis** — Data carriers are often value objects that should be immutable. The language should make it easy to declare an immutable data type without boilerplate.
4. **Copy-with-modify** — A common pattern with data carriers is creating a copy with one or two fields changed. The language should support this naturally.

## Examples

### Python `@dataclass`

```python
from dataclasses import dataclass

@dataclass(frozen=True, order=True)
class Point:
    x: float
    y: float
    label: str = ""

# Auto-provided: __init__, __repr__, __eq__, __hash__, __lt__, __le__, __gt__, __ge__
# p = Point(1.0, 2.0, "origin")
# p2 = Point(1.0, 2.0, "origin")
# p == p2  # True — structural equality
```

### Java Records (Java 16+)

```java
record Point(double x, double y, String label) {}
// Auto-provided: canonical constructor, accessors (x(), y(), label()),
// equals(), hashCode(), toString()
```

### Kotlin `data class`

```kotlin
data class Point(val x: Double, val y: Double, val label: String = "")
// Auto-provided: component1()..componentN(), copy(), equals(), hashCode(), toString()
// val p2 = p.copy(x = 3.0)  // copy with modification
```

### Rust `#[derive]`

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: f64,
    y: f64,
    label: String,
}
// Manual construction only — but traits are opt-in per field
```

## Implications for Orthon

1. **Derive mechanism over keywords** — Orthon should provide a general derive/macro mechanism that synthesises structural methods from a type declaration. A "dataclass" is then just a specific application of derives: `derive(init, eq, repr, hash)` on a type. This is more orthogonal than a dedicated `data class` keyword.
2. **Structural equality as a derive** — Since Orthon already defines a data model (see [`DATA_MODEL.md`](DATA_MODEL.md)), the equality operation for data-aggregate types should derive structural comparison of all fields. A `derive(eq)` would generate this.
3. **Immutability as a modifier** — Rather than a `frozen=True` argument, Orthon could use an immutable-by-default model where fields are `let` (immutable) unless explicitly declared `var` — making data carriers naturally immutable.
4. **Copy-with-modify as an operation** — If Orthon has a `with` expression or similar mechanism (see `COPY_ON_WRITE.md` for related research), copy-with-modify is a natural extension: `point { x = 3.0 }`.
5. **No positional boilerplate** — Unlike Java records or Python dataclasses, Orthon should support named arguments at construction time by default, reducing the need for positional parameter ordering discipline.

## Open Questions

- Should Orthon provide a built-in `derive` construct, or should derives be a library (macro) feature available in the standard library?
- How does a derive mechanism compose with traits/typeclasses? Can you derive a custom trait?
- What is the relationship between dataclass-style types and the property model (see `PROPERTIES.md`)? Are dataclass fields implicitly properties?
- Should equality derivation be recursive (deep structural equality) or shallow?
- How do derives interact with the evolution model — if a field is added to a derived type, does that break equality/hash contracts?
