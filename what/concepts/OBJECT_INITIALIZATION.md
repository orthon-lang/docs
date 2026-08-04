# Object Initialization — Named Parameters with Defaults and Builder Patterns

> **✅ ACCEPTED — [EDR-054](../../how/decision_records/architecture/EDR-054-object-initialization.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **Classification:** **StdLib** — constructor and builder patterns add no new
> language semantics. Named parameters and default values are already covered by
> the general function call model; copy-and-update is sugar over `new` +
> assignment; builder auto-generation is a macro pattern.
>
> **See also:** [`NAMED_AND_OPTIONAL_PARAMETERS.md`](NAMED_AND_OPTIONAL_PARAMETERS.md),
> [`AST_MACROS.md`](AST_MACROS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Default Value, Named Parameter,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Object initialization in most languages suffers from two classical anti-patterns:
- **Telescoping constructors** — multiple overloaded constructors for different
  subsets of fields. Scales combinatorially as the number of optional parameters
  grows.
- **Builder pattern** — external builder class, fluent setters. Solves
  telescoping but requires boilerplate (separate builder class, builder methods,
  final `build()` call).

The core problem: **constructing objects should be readable, type-safe, and
free of boilerplate**, whether the construction is a simple positional call or
a complex object with many optional fields.

## Principles

1. **Named arguments** — Every parameter can be referenced by name, enabling
   readable call sites and arbitrary argument order.
2. **Default values** — Constructor (and general function) parameters can have
   default values; omitted named arguments use the default.
3. **Copy-and-update** — Immutable modifications use `Config(cfg, port: 9090)`
   style syntax — sugar over `new` + field assignment.
4. **No builder boilerplate** — `@builder` auto-generation is a macro pattern
   (AST macros, EDR-029), not a language construct.
5. **Compile-time completeness check** — All required fields must be provided.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Construction Policy | Governs how objects are initialised (named/default/copy-and-update) |
| Default Value Policy | Determines how omitted parameters resolve to defaults |
| Derivation Policy | Builder auto-generation via the `@derive`/`@builder` macro mechanism |

## Model (What)

### Named Parameters with Defaults

```orthon
let cfg = Config(host: "example.com", port: 443, tls: true)
```

### Copy-and-Update

```orthon
let cfg2 = Config(cfg, port: 9090)   # cfg unchanged; cfg2 is a copy with port changed
```

### Builder via Macro

```orthon
@builder
struct Request:
    url: String
    method: String = "GET"
    headers: Map<String, String> = {}
```

## Default Strategy

Object initialization follows the general function call model: named parameters,
default values, and copy-and-update syntax are already part of the language.
Builder auto-generation is provided via the `@builder` macro (AST macros,
EDR-029). No constructor-specific mechanisms exist.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Built-in builder (compiler generates builder class) | Unnecessary — AST macros (EDR-029) provide the same capability without language changes (rejected in EDR-054). |
| Positional-only constructors (C-style) | Prevents named parameter usage and creates ambiguity with multiple parameters of the same type (rejected). |

## Open Questions

1. Should copy-and-update syntax support nested field updates?
2. What is the exact interaction between defaults and exhaustive construction
   checks for large option sets?

## Decision History

- **EDR-054:** Object Initialization accepted as StdLib — named parameters,
  default values, and copy-and-update are general function call features, not
  constructor-specific. Builder auto-generation is a macro pattern.
- **Classification per D-03:** StdLib. No new language semantics.
