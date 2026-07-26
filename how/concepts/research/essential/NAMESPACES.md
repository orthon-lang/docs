# Namespaces

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

How should a language partition declarations into logical modules without conflating that partition with file system layout?

Java ties package names directly to directory paths, making the package hierarchy both a namespace and a file organization rule. C# separates namespaces from file structure, which gives developers more flexibility in organizing code while still maintaining a clear logical naming hierarchy.

The core problem is: should Orthon bind namespaces to directory layout, or should namespace declarations be independent of physical file placement?

## Examples

| Language | Namespace declaration | File-system binding | Import path behavior | Alias support |
|---|---|---|---|---|
| **Java** | `package com.example;` | Must match directory path | Fully qualified package names | No built-in aliasing |
| **C#** | `namespace MyApp;` | Independent of file layout | Logical namespace names | Yes, via `using alias = ...` |
| **Kotlin** | `package` | Typically matches directory but not required | Logical package names | No direct aliasing |
| **Python** | Package directories | Directory structure defines modules | Module names derive from path | Yes, via `import x as y` |
| **Rust** | Modules | File/dir structure maps to module tree | Module paths derive from source tree | Yes, via `use as` |

### C# Example

```csharp
namespace MyCompany.App.Utilities;

public class MathHelpers {
    public static int Square(int x) => x * x;
}
```

## Implications for Orthon

1. **Logical namespace declarations** — Orthon can allow namespace declarations that are independent of file placement, enabling flexible source organization.
2. **Namespace nesting** — Nested namespaces provide a hierarchical naming scheme without requiring corresponding directories.
3. **Simpler file location rules** — Files can be moved without changing namespace declarations, if the namespace is not tied to directories.
4. **Explicit import paths** — The import system can use logical namespace paths rather than physical file paths.
5. **Separation of concerns** — Namespace hierarchy can remain primarily a naming and visibility mechanism, with build tools free to choose file layout independently.

## Open Questions

1. Should Orthon permit multiple namespace declarations per file, or only one top-level namespace?
2. How should namespace nesting interact with package/module organization?
3. Should the compiler enforce any correspondence between namespace names and directory names?
4. Can namespaces be composed or merged across files, and if so, under what visibility rules?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [TOP_LEVEL_DECLARATIONS.md](TOP_LEVEL_DECLARATIONS.md)
- [USING_DIRECTIVES.md](USING_DIRECTIVES.md)
- [VISIBILITY_AND_ENCAPSULATION.md](VISIBILITY_AND_ENCAPSULATION.md)
