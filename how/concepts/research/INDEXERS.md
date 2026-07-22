# Indexers

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

How can an object offer array-like access without exposing its internal storage or forcing users to call access methods?

Java exposes indexed access only through explicit methods such as `get(index)` and `put(index,value)`. C# adds the `this[...]` indexer syntax so objects can support `object[index]` directly. This simplifies collection-like types and custom containers while preserving encapsulation.

The core problem is: should Orthon support custom indexing syntax as a first-class member access form, or should every mapping / sequence type use explicit accessor methods?

## Examples

| Language | Indexer syntax | Custom types supported | Multiple parameters | Setter support |
|---|---|---|---|---|
| **Java** | No | Only arrays via `arr[index]` | No | N/A |
| **C#** | `this[int index]` | Yes | Yes | Yes |
| **Python** | `__getitem__`, `__setitem__` | Yes | Yes | Yes |
| **Swift** | `subscript(index: Int) -> T` | Yes | Yes | Yes |
| **Kotlin** | `operator fun get(index: Int)` | Yes | Yes | Yes |

### C# Example

```csharp
public class Matrix {
    private double[,] data;

    public double this[int row, int col] {
        get => data[row, col];
        set => data[row, col] = value;
    }
}

Matrix m = new Matrix();
double x = m[1, 2];
m[1, 2] = 3.14;
```

## Implications for Orthon

1. **`[]` support for custom objects** — Types can define indexer members to support `obj[key]` syntax, not just arrays and built-in collections.
2. **Multiple index parameters** — Indexers should permit multiple parameters for matrices, dictionaries, and other multi-dimensional access patterns.
3. **Read/write semantics** — Indexers may support both getter and setter behavior, preserving encapsulation of internal storage.
4. **Unified access model** — Indexers should behave like properties from the caller perspective, while still allowing implementation-defined access semantics.
5. **Conflict with existing `[]` forms** — The language must distinguish between array literals, collection indexing, and custom indexer methods unambiguously.

## Open Questions

1. Should indexers be allowed on all types or only on collection-like types?
2. Should indexer invocation be permitted in compile-time constant expressions or only at runtime?
3. How should Orthon represent indexer overload resolution with multiple candidate indexers?
4. Should indexers participate in patterns and deconstruction syntax?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [DATA_MODEL.md](DATA_MODEL.md)
- [PROPERTIES.md](PROPERTIES.md)
- [COLLECTIONS_FUNCTIONAL_API.md](COLLECTIONS_FUNCTIONAL_API.md)
