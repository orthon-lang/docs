# Top-Level Declarations

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

Must every function, constant, and type be nested inside a class or module, or can declarations exist at the file/package level?

Java's answer is strict: **every declaration must be inside a class**. There is no way to define a function outside a class. This forces:

- **Utility classes** — artificial containers like `Math`, `Collections`, `Files` that exist only because the language requires a class boundary. They have no instance state, no constructor, and no purpose beyond being a namespace.
- **Static imports** — a workaround that pulls static members into scope but obscures origin.
- **Conceptual overhead** — new programmers must understand "why is this a class?" before writing their first helper function.

Kotlin, Go, Rust, and Python (at module level) allow **top-level declarations** — functions, properties, and types defined at the package level, outside any class or object. This eliminates the utility class pattern entirely.

The core problem: **does Orthon require declarations to be nested in a container (class, module, namespace), or can they exist freely at the file scope?**

## Examples

| Language | Top-level functions | Top-level properties | Top-level types | Utility class needed? |
|---|---|---|---|---|
| **Java** | No | No (constants via `static final`) | No | Yes — `Math`, `Collections`, etc. |
| **Kotlin** | Yes | Yes | Yes (classes can be top-level) | No |
| **Go** | Yes | Yes | Yes | No — functions live in packages |
| **Rust** | Yes (free functions) | Yes (const/static) | Yes | No — `impl` blocks on types |
| **Python** | Yes (module level) | Yes (module level) | Yes | No |
| **Swift** | Yes (global functions) | Yes (global variables) | Yes | No |

## Implications for Orthon

1. **Top-level functions** — A function can be declared at the package level without a class wrapper. This eliminates utility classes entirely:

   ```
   // No `class Math { ... }` wrapper needed
   fun sin(x: Float) -> Float ...
   fun cos(x: Float) -> Float ...
   ```

2. **Top-level constants** — Constants can be defined at the file level:

   ```
   PI: Float = 3.1415926535
   MAX_RETRIES: Int = 3
   ```

3. **Top-level types** — Types (structs, enums, traits) are declared at the package level. Nesting is opt-in, not required.

4. **No artificial wrappers** — The compiler never requires a class boundary for functions that operate on a type. Extension functions (see `EXTENSION_FUNCTIONS.md`) provide the "belongs to this type" feel without wrapping.

5. **Package as namespace** — The package/module serves as the namespace boundary. Two files in the same package share top-level declarations by default (subject to visibility rules — see `VISIBILITY_AND_ENCAPSULATION.md`).

6. **Import for access** — Top-level declarations from other packages require explicit import:

   ```
   import math.sin     // imports top-level function sin
   import math.PI      // imports top-level constant PI
   ```

## Open Questions

1. Do top-level declarations belong to a package, a module, or a file? What is the scoping unit?
2. Can top-level declarations be private to a file? Does Orthon need a file-private visibility level?
3. Do top-level functions participate in name resolution the same way as type methods? Can a top-level function be used as a method via uniform call syntax (like Kotlin's extension functions)?
4. Should there be a `main` function convention at the top level for program entry points?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `VISIBILITY_AND_ENCAPSULATION.md`
- [ ] `EXTENSION_FUNCTIONS.md`
- [ ] `FUNCTIONS.md`
- [ ] `OBJECTS_AND_SINGLETONS.md`
- [ ] `what/GLOSSARY.md`
