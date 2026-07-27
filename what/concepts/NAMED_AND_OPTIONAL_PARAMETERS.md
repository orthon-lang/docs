# Named and Optional Parameters

## Issue (Why)

How should a language make function and constructor calls easier to read and maintain, especially when a callable has many parameters? Positional-only arguments (Java) require overloaded methods for alternative parameter sets, while named arguments and optional parameters reduce overload explosion and clarify call sites.

## Principles

1. **Named arguments** — Call sites can specify parameters by name, improving readability and allowing parameters in any order.
2. **Optional parameters with defaults** — Functions may declare default values, reducing the need for multiple overloads.
3. **Better API evolution** — Adding a new optional parameter does not break existing callers.
4. **Interaction with overload resolution** — The compiler must define how named/optional parameters interact with overloaded call resolution.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Parameter Binding Policy | Governs how arguments are matched to parameters — positional vs named |
| Overload Resolution Policy | Controls how named/optional parameters affect overload candidate selection |

## Model (What)

Named and optional parameters are function call ergonomics:

```orthon
fun connect(host: String, port: Int = 80, use_ssl: Bool = false):
    # ...

# Positional call
connect("example.com", 443, true)

# Named call — any order
connect(host: "example.com", use_ssl: true)

# Mixed — positional then named
connect("example.com", use_ssl: true)
```

Optional parameters with defaults eliminate the need for overloaded variants:

```orthon
# Without optional parameters — three overloads needed
fun connect(host: String): ...
fun connect(host: String, port: Int): ...
fun connect(host: String, port: Int, use_ssl: Bool): ...

# With optional parameters — one function
fun connect(host: String, port: Int = 80, use_ssl: Bool = false): ...
```

## Default Strategy

Named and optional parameters are a **StdLib** convention — they desugar to positional parameters + default values. The compiler recognizes named argument syntax and resolves parameter binding, but no new runtime semantics are introduced. Named arguments are syntactic sugar over positional arguments with reordering.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Positional only | All arguments are positional. Overloads handle optionality. Simple but verbose. |
| Named-only (Smalltalk) | All arguments must be named. Self-documenting but verbose for simple calls. |
| Builder pattern | Builder object accumulates named parameters and produces the target value. Patterns, not language. |

## Open Questions

1. Should named arguments be allowed for all calls or only when explicit parameter names exist?
2. How should default values be evaluated — at definition time or call time?
3. How do named arguments interact with variadic parameters and destructuring patterns?

## Decision History

- **EDR-065:** Named and Optional Parameters accepted as StdLib — function call ergonomics. Desugarable to positional parameters + default values. Named argument resolution is a compiler-level name binding operation, but the semantics are purely syntactic.
- **Classification per D-03:** StdLib. Named/optional parameters are syntactic sugar over positional parameters. The parameter binding and default value generation are desugaring operations, not new runtime semantics.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
