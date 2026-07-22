# Query Expressions

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

How should a language make declarative collection queries feel natural and composable?

C# introduced **query expressions** as SQL-like syntax over `IEnumerable<T>` and `IQueryable<T>`, which the compiler rewrites into method chains. This gives developers a concise way to express filtering, projection, joins, and grouping without sacrificing composability.

The core problem is: should Orthon provide dedicated query syntax on top of collection operations, or should query APIs remain method-chain-only?

## Examples

| Language | Query syntax | Compiler rewrite | Lazy evaluation | Collection integration |
|---|---|---|---|---|
| **C#** | SQL-like `from ... select` | Rewritten to LINQ methods | Yes | Native through `IEnumerable`/`IQueryable` |
| **Java** | No query syntax | Streams API methods | Yes | Explicit stream conversion required |
| **Kotlin** | No query syntax | Extension functions on collections | Yes | Direct method chaining |
| **Python** | Comprehensions | Native expression form | No/yes depending on generator | Built into language |
| **Rust** | Iterator methods | No dedicated query syntax | Yes | Direct iterator methods |

### C# Example

```csharp
var result = from person in people
             where person.Age >= 18
             orderby person.LastName
             select new {
                 person.FirstName,
                 person.LastName
             };
```

## Implications for Orthon

1. **Declarative query syntax** — Orthon can offer a `from`/`where`/`select` style syntax as a surface convenience for collection processing.
2. **Translation to collection operations** — Query expressions should be syntactic sugar for existing functional collection APIs, not an independent execution model.
3. **Composability with existing operators** — The query syntax should interoperate with collection method chains and support custom collection types.
4. **Readability for complex queries** — A dedicated syntax can make joins, grouping, and nested queries clearer than deeply nested method chains.
5. **Avoiding syntax bloat** — The query system should stay small and orthogonal, not become a separate mini-language with many exotic clauses.

## Open Questions

1. Should query syntax be confined to collections, or also support optional values, futures, and other monadic APIs?
2. How should Orthon express joins, grouping, and aggregation in a compact way?
3. Should the compiler permit query expressions only on types that implement a specific extension protocol?
4. How should query expressions interact with lazy vs eager collections?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [COLLECTIONS_FUNCTIONAL_API.md](COLLECTIONS_FUNCTIONAL_API.md)
- [EXTENSION_FUNCTIONS.md](EXTENSION_FUNCTIONS.md)
- [PATTERN_MATCHING.md](PATTERN_MATCHING.md)
