# Object Initialization

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How do you create complex objects with many optional fields without writing verbose or error-prone code?

Object initialization suffers from two classical anti-patterns:
1. **Telescoping constructors** — multiple overloaded constructors for different subsets of fields. Scales combinatorially.
2. **Builder pattern** — external builder class, fluent setters. Solves telescoping but requires boilerplate (separate builder class, builder methods, final `build()` call).

The core problem: **the language lacks syntax for naming which field receives which value**, forcing either ordering (positional parameters) or external abstractions (builder).

## Principles

1. **Named arguments** — Every parameter in a constructor call can be referenced by name, not just position.
2. **Default values** — Constructor parameters can have default values; omitted named arguments use the default.
3. **No builder boilerplate** — The language should generate whatever wiring the builder pattern provides manually.
4. **Compile-time completeness check** — All required (non-defaulted) fields must be provided.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Mutability Policy | Determines whether constructed objects are immutable by default |
| Default Policy | Governs how default values are resolved (literal, function call, lazy) |
| Builder Policy | Controls whether the compiler auto-generates a builder API |

## Model (What)

Object construction uses **named parameters with defaults**. The compiler generates the necessary glue.

```
struct Config
    host: String = "localhost"
    port: Int = 8080
    timeout: Duration = Duration.seconds(30)
    tls: Bool = false

// Usage:
let cfg1 = Config()
let cfg2 = Config(host: "example.com", port: 443, tls: true)
let cfg3 = Config(host: "example.com")  // port, timeout, tls use defaults
```

Key features:
- Named parameters in struct/class constructors.
- Default values in the struct definition.
- Copy-and-update syntax for immutable modifications: `Config(cfg, port: 9090)`.
- No separate builder class needed; compiler may offer `build()` as an extension.

## Default Strategy

All parameters have explicit names. Defaults are evaluated eagerly at construction time. Builder generation is opt-in via annotation.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Builder auto-generation | Compiler generates a builder class from struct definition on demand. |
| Positional-only | No named parameters; only order-based (C-style). |
| Lazy defaults | Default values are lazily evaluated on first access. |
| Partial construction | Object can be constructed in two phases — required fields first, optionals later. |

## Open Questions

1. Should copy-and-update syntax be part of the language, or should it use `with` blocks?
2. Should there be a way to expose positional constructors alongside named ones for ergonomics?
3. How do named parameters interact with inheritance?
4. Should default values be allowed to reference other parameters?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
