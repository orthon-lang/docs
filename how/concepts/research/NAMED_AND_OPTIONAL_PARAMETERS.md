# Named and Optional Parameters

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

How should a language make function and constructor calls easier to read and maintain, especially when a callable has many parameters?

Java requires positional arguments for method calls and relies on overloaded methods for alternative parameter sets. C# adds named arguments and optional parameters with defaults, reducing overload explosion and clarifying call sites.

The core problem is: should Orthon support named and optional parameters as part of the call syntax, or should it rely on manual builder patterns and overloads?

## Examples

| Language | Named args | Optional params | Call-site clarity | Overload reduction |
|---|---|---|---|---|
| **Java** | No | No | Poor for many params | No |
| **C#** | Yes | Yes | Good | Yes |
| **Python** | Yes | Yes | Good | Yes |
| **Kotlin** | Yes | Yes | Good | Yes |
| **Swift** | Yes | Yes | Good | Yes |

### C# Example

```csharp
public void Connect(string host, int port = 80, bool useSsl = false) {}

Connect(host: "example.com", useSsl: true);
```

## Implications for Orthon

1. **Named arguments** — Call sites can specify parameters by name, improving readability and allowing parameters to be passed in any order.
2. **Optional parameters with defaults** — Functions and constructors may declare default values, reducing the need for multiple overloads.
3. **Better API evolution** — Adding a new optional parameter does not break existing callers, unlike new positional parameters.
4. **Partial application and named argument expressiveness** — Named parameters can make APIs self-documenting, especially for boolean or configuration-heavy calls.
5. **Interaction with overload resolution** — The compiler must define how named/optional parameters interact with overloaded call resolution and ambiguity.

## Open Questions

1. Should Orthon allow named arguments for all calls or only for calls with explicit parameter names?
2. How should default values be evaluated, especially if they are expressions with side effects?
3. Should optional parameters be allowed on every function, or only on functions with a specific annotation?
4. How should named arguments interact with variadic parameters and destructuring patterns?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [OBJECT_INITIALIZATION.md](OBJECT_INITIALIZATION.md)
- [FUNCTIONS.md](FUNCTIONS.md)
- [TYPE_SYSTEM.md](../architecture/TYPE_SYSTEM.md)
