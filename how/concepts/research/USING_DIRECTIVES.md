# Using Directives

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

How should a language import names from other modules or namespaces while keeping the calling code readable and unambiguous?

Java uses `import` for type names and has no direct aliasing or static import in the same way as C#. C# uses `using` directives with powerful features: aliasing, static imports, and global imports applicable across a project. This gives shorter code and flexible scoping, but also adds rules for name resolution.

The core problem is: what import syntax should Orthon use, and how much convenience should it provide without hurting clarity?

## Examples

| Language | Import syntax | Aliasing | Static import | Global/project-level import |
|---|---|---|---|---|
| **Java** | `import com.example.Foo;` | No | `import static ...` | No |
| **C#** | `using MyAlias = Some.Long.Namespace;` | Yes | `using static ...` | `global using ...` |
| **Kotlin** | `import foo.bar` | Yes | `import foo.bar.BAZ` | No |
| **Python** | `import foo as bar` | Yes | N/A | No |
| **Rust** | `use foo::bar as baz;` | Yes | `use foo::bar;` | No |

### C# Example

```csharp
using IO = System.IO;
using static System.Math;

global using System.Collections.Generic;

namespace MyApp;

public class Example {
    public void Do() {
        var files = IO.Directory.GetFiles(".");
        var x = Sqrt(2.0);
    }
}
```

## Implications for Orthon

1. **Alias imports** — Orthon can support import aliases to shorten long namespace paths and resolve naming conflicts.
2. **Static imports** — A `using static`-style feature can bring selected members into scope without importing the entire type.
3. **Global imports** — Project- or package-scoped imports can reduce boilerplate for ubiquitous namespaces.
4. **Scoped imports** — Import directives should be allowed at file scope or narrower scopes for local clarity.
5. **Explicitness vs convenience** — The language should balance imported convenience with the risk of obscuring symbol origins.

## Open Questions

1. Should global imports be module-level only, or can they be scoped to a package/project?
2. How should aliasing interact with type inference and symbol resolution?
3. Can `using` directives import only specific members, and how does that affect name collision rules?
4. Should Orthon support wildcard imports, qualified imports, or only explicit imports?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [NAMESPACES.md](NAMESPACES.md)
- [TOP_LEVEL_DECLARATIONS.md](TOP_LEVEL_DECLARATIONS.md)
- [VISIBILITY_AND_ENCAPSULATION.md](VISIBILITY_AND_ENCAPSULATION.md)
