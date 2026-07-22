# Function Overloading

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

Can multiple functions in the same scope share the same name as long as they have different parameter types?

Two opposing answers dominate:

1. **Full overloading (Java, C++, C#, TypeScript)** — a single name can refer to multiple function implementations distinguished by their parameter types (number, order, types). The compiler selects the correct overload at the call site based on the argument types. This enables ergonomic APIs (`println(int)`, `println(String)`, `println(double)`) but introduces ambiguity (which overload is called with `null`? What about type promotion? Varargs conflicts?).

2. **No overloading (Go, Rust, C)** — function names in a package/module must be unique. If you need different behaviour for different types, you either use different names (`PrintInt`, `PrintString`) or generics (type parameters). Go is the strictest: even methods on different types must have unique names within the package.

The core problem: **overloading trades naming simplicity (one name to remember) against resolution complexity (which overload actually runs?)** . For LLMs, overloading introduces ambiguity — without seeing runtime types or knowing language-specific resolution rules, the LLM may choose the wrong overload. Go's prohibition eliminates this class of error entirely.

A related question: **if overloading is prohibited, does the language provide alternatives** — generics, default parameters, variadic arguments, or pattern matching — that cover the same use cases without requiring overload resolution?

## Principles

1. **Name uniqueness** — Function names within the same scope are unique. No two functions share a name, regardless of parameter types.

2. **Explicit intent** — The function name tells the caller exactly which function is being called, without needing to understand overload resolution rules.

3. **Alternatives over overloading** — Instead of overloading, use:
   - **Generics / type parameters** — one function works with multiple types
   - **Default parameters** — optional parameters with default values
   - **Variadic arguments** — variable number of arguments of the same type
   - **Pattern matching** — dispatch to different implementations based on argument shape
   - **Distinct names** — `ParseInt`, `ParseFloat`, `ParseBool` instead of `parse(int)`, `parse(float)`, `parse(boolean)`

4. **LLM predictability** — A unique name per function eliminates the most common source of LLM-generated overload resolution errors.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Name Resolution Policy | Determines whether names must be unique within a scope or can be overloaded |
| Dispatch Policy | Governs how the correct function is selected at call sites |
| Generic Policy | Provides the primary alternative to overloading — type-parameterised functions |

## Model (What)

Under the no-overloading model:

- Every function in a module/package has a unique name.
- Method names on different types are also unique within the module (Go's strictest variant) or unique within the type (laxer variant).
- Generics handle the case where the same operation applies to multiple types: `func max[T Ordered](a, b T) T`.
- Default parameters handle the case where some arguments are optional: `func open(path string, mode Mode = ReadOnly)`.
- Distinct names handle the case where the operation is genuinely different per type: `parseInt`, `parseFloat`.

The LLM benefit: unique names mean zero ambiguity at the call site. The LLM never needs to reason about overload resolution priority, type promotion chains, or null-ambiguity.

## Default Strategy

**No function overloading.** All function names in a scope are unique. Generics and default parameters provide the alternatives for operations that vary by type.

## Alternative Strategies

1. **Full overloading (Java-style)** — Allow multiple functions with the same name distinguished by parameter types. Resolve at compile time. Accept the ambiguity trade-offs.

2. **Overloading with restrictions** — Allow overloading only for constructors (if the language has them) or only for methods, not free functions.

3. **Multi-methods (CLOS-style)** — Dispatch is based on runtime types of all arguments, not just the receiver. More expressive but more complex.

## Open Questions

1. Should the prohibition on overloading apply to operators as well, or are built-in operators exempt?
2. How should the language handle the equivalent of `print(int)` and `print(string)` — via generics (`print[T](value T)`) or distinct names?
3. Should methods on different types be allowed to share a name (like Rust traits), or must they be globally unique within the module (like Go)?
4. If default parameters are the alternative, how do they interact with the type system and function identity?

## Cross-References

- See `FUNCTIONS.md` for the overall function model
- See `GENERICS.md` for type-parameterised functions as an alternative to overloading
- See `NAME_RESOLUTION.md` for name resolution rules in the compiler architecture
- See [`../../why/MANIFESTO.md`](../../why/MANIFESTO.md) § Minimal Core for the principle that discourages feature proliferation
