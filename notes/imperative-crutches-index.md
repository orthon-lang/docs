# Imperative Crutches — Index

> Techniques that were *workhorses* in older versions of Python and Java,
> but are now considered anti-patterns due to verbosity, unsafety, or
> inefficiency. Modern solutions make code **declarative, type-safe, and
> self-documenting**.

## Catalog

| # | Topic | File |
|---|-------|------|
| 1 | Collections and Loops | `imperative-crutch-collections-loops.md` |
| 2 | Null handling | `imperative-crutch-null-handling.md` |
| 3 | Resource management | `imperative-crutch-resource-management.md` |
| 4 | Collection initialization | `imperative-crutch-collection-init.md` |
| 5 | Type checking & casting | `imperative-crutch-type-casting.md` |
| 6 | Date and time | `imperative-crutch-datetime.md` |
| 7 | Lazy sequences / iterators | `imperative-crutch-lazy-sequences.md` |
| 8 | Multi-key sorting | `imperative-crutch-sorting.md` |
| 9 | Metaprogramming and reflection | `imperative-crutch-metaprogramming.md` |
| 10 | Serialization / Deserialization | `imperative-crutch-serialization.md` |

## Common Pattern

All these "imperative crutches" share the following traits:

1. **They force extra code** (loops, checks, wrappers) that is not part of the business logic
2. **They defer errors to runtime** (type casts, NPE, leaked resources)
3. **They mix business logic with implementation details** (*how* to iterate, *how* to close)

Modern approaches (list comprehensions, context managers, Stream API,
Optional, pattern matching, records, coroutines, etc.) move this logic
**to the language and compiler level**, letting the programmer express
*what* to do rather than *how*.

## Application to Orthon

Each file in this directory contains:
- a concrete imperative-crutch example (Python / Java)
- a modern declarative solution
- implications for Orthon's design: how to avoid similar anti-patterns
