# Dynamic Typing

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

How should a statically typed language provide an escape hatch for dynamic runtime behaviour?

Java is statically typed and has limited support through reflection and `Object`. C# introduces the `dynamic` keyword as an optional type that bypasses compile-time type checking and resolves member access at runtime.

The core problem is: should Orthon permit optional dynamic typing within a primarily static language, and if so, how should that interact with tooling, performance, and safety?

## Examples

| Language | Dynamic escape hatch | Compile-time checks | Runtime dispatch | Tooling impact |
|---|---|---|---|---|
| **Java** | Reflection / `Object` | Static with casts | Runtime lookup via reflection | Weak |
| **C#** | `dynamic` keyword | None for dynamic expressions | Runtime binder resolves calls | Stronger than raw reflection |
| **Python** | Dynamic by default | None | Native | N/A |
| **Kotlin** | `Any` + reflection | Static on `Any` | Runtime reflection | Manual |
| **TypeScript** | `any` / `unknown` | None / limited | Runtime type-dependent | IDE-assisted |

### C# Example

```csharp
dynamic x = GetValue();
int result = x + 5;
```

## Implications for Orthon

1. **Optional dynamic type** — Orthon can permit a `dynamic`-like type that disables compile-time member checks and defers them to runtime.
2. **Runtime dispatch semantics** — Dynamic expressions should resolve calls using runtime type information, with well-defined failure modes.
3. **Toolchain support** — IDEs and LLM tools may still provide best-effort completion based on runtime types, but should clearly mark dynamic code as less safe.
4. **Safety boundary** — Dynamic typing should be an explicit escape hatch, not the default, so the static type system remains the norm.
5. **Interoperability use cases** — Dynamic typing is useful for interop, reflection-heavy APIs, and rapid prototyping.

## Open Questions

1. Should dynamic types be implicitly converted to/from static types, or require explicit casts?
2. How should dynamic semantics interact with null safety and optional values?
3. Should Orthon provide a separate runtime binder API, or keep dynamic behavior limited to the `dynamic` type?
4. How does dynamic typing impact code generation and target runtimes that lack rich reflection?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [TYPE_SYSTEM.md](../architecture/TYPE_SYSTEM.md)
- [REFLECTION_ALTERNATIVES.md](REFLECTION_ALTERNATIVES.md)
- [NULL_SAFETY.md](NULL_SAFETY.md)
