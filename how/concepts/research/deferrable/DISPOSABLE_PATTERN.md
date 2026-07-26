# Disposable Pattern

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

How should a language ensure deterministic resource cleanup while still using garbage collection for general memory management?

Java relies on finalizers and try-with-resources for resource cleanup, but finalizers are non-deterministic and expensive. C# introduces `IDisposable` and the `using` statement/declaration to provide deterministic cleanup in a familiar, RAII-like way.

The core problem is: should Orthon adopt an explicit dispose protocol with syntactic support for scoped resource management, or should it rely solely on GC and finalization?

## Examples

| Language | Cleanup protocol | Compiler support | Deterministic release | Common use case |
|---|---|---|---|---|
| **Java** | `try-with-resources` + `AutoCloseable` | Yes | Yes | Streams, files |
| **C#** | `IDisposable` + `using` | Yes | Yes | File handles, sockets |
| **Python** | `with` + context managers | Yes | Yes | Files, locks |
| **Rust** | RAII + `Drop` | Yes | Yes | All owned resources |
| **Go** | `defer` | Yes | Yes | Files, locks |

### C# Example

```csharp
using (var stream = new FileStream(path, FileMode.Open)) {
    // use stream
}
```

### C# using declaration

```csharp
using var stream = new FileStream(path, FileMode.Open);
// stream is disposed at end of scope
```

## Implications for Orthon

1. **Explicit disposal protocol** — Orthon can define a disposable interface or trait that resource types implement to provide cleanup.
2. **Scoped cleanup syntax** — A `using`-style statement or declaration can guarantee resource release at the end of a lexical scope.
3. **Compiler enforcement** — The compiler can warn or error when disposable resources are not used in a scoped cleanup construct.
4. **Separation from GC** — Deterministic cleanup should be separate from garbage collection, allowing memory to remain GC-managed while other resources are released promptly.
5. **Integration with ownership/borrowing** — Disposable resources can blend with ownership models to ensure resources are transferred correctly and not leaked.

## Open Questions

1. Should Orthon use an explicit disposable trait/interface or rely on a more general ownership cleanup mechanism?
2. How should `using` syntax interact with async/await and suspended execution?
3. Can disposable handling be inferred automatically for local variables, or must it always be explicit?
4. Should the language support both scoped `using` declarations and block-based `using` statements?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [OWNERSHIP.md](OWNERSHIP.md)
- [ASYNC_AWAIT.md](ASYNC_AWAIT.md)
- [ERROR_HANDLING.md](ERROR_HANDLING.md)
