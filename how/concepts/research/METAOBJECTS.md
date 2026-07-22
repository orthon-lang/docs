# Metaobjects (Metaclasses, Macros, Traits)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via ECR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How do you add cross-cutting behaviour (logging, serialisation, validation, registration) to many classes without repeating code in each one?

Inheritance and composition solve many reuse problems, but they fail for **cross-cutting concerns**:
- Adding logging to every method requires manual edits in every class.
- Registering every subclass in a registry requires boilerplate in each class definition.
- Serialisation (JSON, XML) must be implemented per-type or via reflection.
- Dynamic behaviour (proxies, lazy loading) requires generated wrappers.

The core problem: **the ability to modify class behaviour at definition time** (metaprogramming) is needed so cross-cutting logic can be expressed once and applied declaratively.

## Principles

1. **Compile-time metaprogramming** — Metaoperations run at compile time, not runtime, to preserve performance.
2. **Declarative application** — Cross-cutting behaviour is applied via annotations/traits/decorators, not by modifying source.
3. **No runtime magic** — Generated code is explicit; no implicit behaviour that cannot be understood by reading the source.
4. **Hygiene** — Macros and metaprogramming do not accidentally capture variables from the surrounding scope.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Metaprogramming Policy | Determines which metaprogramming mechanisms are available (macros, annotations, traits) |
| Hygiene Policy | Governs macro hygiene (hygienic vs unhygienic expansion) |
| Compile-time Execution Policy | Controls the capabilities of compile-time code (I/O, FFI, reflection) |

## Model (What)

The language provides **compile-time metaprogramming** through a combination of mechanisms:

1. **Traits/Mixins** — reusable behavioural components that can be composed into classes.
2. **Annotations/Attributes** — declarative markers that trigger compile-time code generation (serialisation, validation).
3. **Macros** — functions that operate on the AST at compile time, producing code.
4. **Compile-time reflection** — read-only access to type structure at compile time.

```
// Derive — automatic trait implementation generation
@derive(Show, Eq, Clone)
type Point(x: Int, y: Int)
// Compiler generates: Show, Eq, and Clone implementations

// Annotation-driven serialisation
@json
struct Person
    name: String
    age: Int

// Macro for lazy initialisation
lazy var config = loadConfig()

// Trait for logging
trait Logged:
    before method:
        logger.info("Calling {method.name}")
```

Key features:
- **AST macros** — hygienic, pattern-matching on syntax trees.
- **Declarative annotations** — no runtime reflection; code generation at compile time.
- **Traits with method interception** — `before`, `after`, `around` advice (aspect-oriented).
- **Compile-time evaluation** — macro functions execute during compilation.

## Default Strategy

Traits are the default mechanism for behaviour composition. Annotations trigger compile-time code generation. Macros are available for advanced metaprogramming but require explicit `macro` keyword.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Runtime reflection | Full runtime type introspection and method dispatch (Java, Python). |
| Decorators | Function-level wrappers (Python `@decorator`). |
| Procedural macros | Rust-style — macro functions that tokenise/parse input and produce output. |
| Code generation | External code generators (protobuf, OpenAPI) — not a language feature. |

## Open Questions

1. Should macros be in a separate "macro language" or in the host language itself?
2. Should compile-time evaluation have I/O access? (File reading for code generation?)
3. How to prevent macro abuse (non-locality, unexpected behaviour)?
4. Should traits be structural (Go-style) or nominal (Rust-style)?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
