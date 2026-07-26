# Partial Classes

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

How should a language allow a single type declaration to be split across multiple files?

Java requires a type to be declared in one file, which is simple but makes code generation and designer-generated support classes harder to integrate cleanly. C# supports **partial classes**, allowing different parts of the same type to live in separate files. This is useful for generated code, large types, and designer tooling.

The core problem is: should Orthon support distributed type definitions for the same nominal type, or should every type be defined in one location?

## Examples

| Language | Partial types | Code generation support | Multiple files | Same compile-time type |
|---|---|---|---|---|
| **Java** | No | No | No | N/A |
| **C#** | `partial` | Yes | Yes | Yes |
| **Swift** | No | No | No |
| **Kotlin** | No | No | No |
| **Rust** | Modules + `impl` blocks | Yes for extensions | Yes | Yes |

### C# Example

```csharp
// File: Person.Properties.cs
public partial class Person {
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

// File: Person.Behavior.cs
public partial class Person {
    public string FullName() {
        return FirstName + " " + LastName;
    }
}
```

## Implications for Orthon

1. **Partial type declarations** — Orthon may allow `partial` type definitions so a single nominal type can be assembled from multiple files.
2. **Tooling and generated code** — Partial definitions help separate generated boilerplate from handwritten behavior.
3. **Single type identity** — All partial fragments compile to the same type and share the same accessibility and inheritance semantics.
4. **Locality and readability** — Partial classes enable file-level organization for large types, but they also increase the risk of splitting related behavior across files.
5. **Dependency management** — The compiler must ensure that all partial fragments are visible together during type resolution and member lookup.

## Open Questions

1. Should partial declarations be permitted for all type kinds, or only for classes/structs?
2. How does Orthon prevent duplicate member declarations across fragments?
3. Should partial types be discoverable by name only, or also by file metadata or explicit grouping?
4. Can partial fragments introduce new base types or interfaces, or must inheritance be declared in one canonical fragment?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [OBJECT_INITIALIZATION.md](OBJECT_INITIALIZATION.md)
- [EXTENSION_FUNCTIONS.md](EXTENSION_FUNCTIONS.md)
- [TOP_LEVEL_DECLARATIONS.md](TOP_LEVEL_DECLARATIONS.md)
