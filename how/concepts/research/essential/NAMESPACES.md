# Namespaces

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-29
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How should a language partition declarations into logical modules without conflating that partition with file system layout, while still providing LLMs and humans with a navigable project map?

Java ties package names directly to directory paths, making the package hierarchy both a namespace and a file organization rule. C# separates namespaces from file structure, which gives developers more flexibility in organizing code while still maintaining a clear logical naming hierarchy.

The core problem is: should Orthon bind namespaces to directory layout, or should namespace declarations be independent of physical file placement? If independent, how does an LLM (or human) discover the module structure without reading every file?

This question has two dimensions:

1. **Language semantics** — Is namespace bound to filesystem (Java/Python/BAML) or independent (C#/Kotlin)?
2. **LLM navigation** — What is the source of truth for dependencies: the program text (module header) or a global namespace resolvable from file paths?

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

### BAML Model: Filesystem as Namespace (No Imports)

BAML takes a polar opposite approach: the directory tree **is** the module system. There are no `import` statements. Directory prefixes (`ns_`) mark namespaces, and cross-namespace access uses fully qualified names (`root.catalog.Product`).

```text
baml_src/
├── ns_catalog/
│   └── product.baml
└── ns_orders/
    └── order.baml
```

This eliminates import management entirely — a major source of LLM context pollution. Agents cannot add wrong imports, miss needed ones, or grep for import paths.

| Aspect | BAML approach | Orthon trajectory |
|--------|---------------|-------------------|
| Namespace source | Filesystem layout | Logical namespace declarations |
| Import syntax | None (FQN only) | Explicit `use` in module header (EDR-072) |
| Encapsulation | None (all visible by FQN) | Module as boundary (VISIBILITY_AND_ENCAPSULATION.md) |
| Effect isolation | None | Capability-based (EDR-072) |
| LLM navigation | Filesystem tree | Module header as semantic index |

**Analysis:** BAML solves a real LLM problem (import errors) but at the cost of language-level encapsulation and capability checking. For Orthon, the lesson is not to adopt filesystem-as-namespace, but to invest in **repository introspection tools** (`orthon describe`, `orthon deps`, `orthon graph`) that export the semantic module map — making the module header the single source of truth that LLMs query instead of guessing imports.

See [`REPOSITORY_INTROSPECTION.md`](../../how/concepts/research/REPOSITORY_INTROSPECTION.md) for the full analysis.

---

## Implications for Orthon

1. **Logical namespace declarations** — Orthon can allow namespace declarations that are independent of file placement, enabling flexible source organization.
2. **Namespace nesting** — Nested namespaces provide a hierarchical naming scheme without requiring corresponding directories.
3. **Simpler file location rules** — Files can be moved without changing namespace declarations, if the namespace is not tied to directories.
4. **Explicit import paths** — The import system can use logical namespace paths rather than physical file paths.
5. **Separation of concerns** — Namespace hierarchy can remain primarily a naming and visibility mechanism, with build tools free to choose file layout independently.
6. **Module header as semantic index** — The module header (`module ... use ... effects ...`) is the source of truth for dependencies. Tooling can export this as an LLM-readable project map, eliminating the need for filesystem-as-namespace while preserving encapsulation.

### Architectural Question

> **What is the source of truth for dependencies — the program text (module header) or a global namespace (filesystem/resolvable FQN)?**

If the source of truth is a **global namespace** (BAML model), then any symbol can be resolved by FQN, and `use` becomes syntactic sugar. This weakens explicit dependency declarations and capability-based access control.

If the source of truth is the **module header** (EDR-072 model), then `use` declarations are not sugar but a contract — they declare capabilities, effects, and the module's public API. The filesystem can serve as a navigation convention (default namespace mapping) but does not replace the module system.

**Orthon's trajectory:** Module header as source of truth. Filesystem-to-namespace mapping is a build convention, not a language requirement. LLM navigation is solved through repository introspection tools, not through elimination of imports.

## Open Questions

1. Should Orthon permit multiple namespace declarations per file, or only one top-level namespace?
2. How should namespace nesting interact with package/module organization?
3. Should the compiler enforce any correspondence between namespace names and directory names?
4. Can namespaces be composed or merged across files, and if so, under what visibility rules?
5. Should the default filesystem-to-namespace mapping be a build convention (enforced by tooling, not by language), leaving namespace declarations as the source of truth?
   - If yes: the convention maps `src/net/http.orthon` → namespace `net::http` by default
   - The module header can still declare `module other.name` to override
   - The compiler never requires filesystem → namespace correspondence

## Decision History

| Date | Decision | Source |
|------|----------|--------|
| 2026-07-29 | Rejected filesystem-as-namespace (BAML model). Module header is source of truth for dependencies. Filesystem mapping is a build convention, not a language requirement. LLM navigation solved via repository introspection tools, not import elimination. | BAML gap analysis + architectural analysis |

---

### Cross-References

- [TOP_LEVEL_DECLARATIONS.md](TOP_LEVEL_DECLARATIONS.md)
- [USING_DIRECTIVES.md](USING_DIRECTIVES.md)
- [VISIBILITY_AND_ENCAPSULATION.md](VISIBILITY_AND_ENCAPSULATION.md)
- [REPOSITORY_INTROSPECTION.md](../../how/concepts/research/REPOSITORY_INTROSPECTION.md) — Tooling approach to LLM project navigation
- [CONTEXT_LIMITED_MODULES.md](../../what/concepts/CONTEXT_LIMITED_MODULES.md) — Module header as source of truth (EDR-072)
- [baml-concepts-orthon-gap-analysis.md](../../notes/baml-concepts-orthon-gap-analysis.md) — BAML comparison
