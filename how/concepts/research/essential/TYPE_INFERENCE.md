# Type Inference

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a variable, expression, or function parameter get its type without an explicit annotation — and what are the trade-offs of different inference strategies?

Type inference sits at the intersection of conciseness and clarity:

- **Fully annotated** (Java pre-10, C, Go) — every variable and function signature carries an explicit type. Maximum clarity for the reader; maximum verbosity for the writer. Refactoring is tedious because type changes cascade through annotations.

- **Full inference** (OCaml, Haskell) — types are inferred for all expressions. Function signatures can omit type annotations entirely. Maximum conciseness; but error messages can be opaque, and understanding a complex inferred type requires mental execution of the inference algorithm.

- **Local inference** (Rust, Kotlin, C# `var`, TypeScript) — types are inferred within function bodies but required at function boundaries (parameter types, return types). Balances conciseness with explicit API contracts.

- **Gradual inference** (TypeScript, Python with myPy, Typed Racket) — some code is typed, some is not. Inference crosses the typed/untyped boundary with checked or unchecked conversions.

The core problem: **how does Orthon determine types without sacrificing either readability or compiler rigour?**

This decision is tightly coupled with several other concepts:

- `DECLARATION_BY_ASSIGNMENT.md` — how variables come into existence; declaration mechanism and inference mechanism are coupled.
- `GRADUAL_TYPING.md` — whether inference crosses typed/untyped boundaries.
- `GENERICS.md` — inference with generic parameters and type constraints.
- `TYPED_HOLES.md` — holes expose their inferred type to guide generation.
- `STRUCTURAL_TYPING.md` — structural type satisfaction interacts with inference of interface conformance.

## Principles

1. **Inference within functions, explicit at boundaries** — Function parameters and return types must be annotated. Local variables, intermediate expressions, and generic type arguments within a function body are inferred. This follows Orthon's explicitness principle at API surfaces while avoiding annotation noise inside implementations.

2. **Bidirectional inference** — Inference flows both top-down (contextual, expected type from usage site) and bottom-up (from actual argument types). This enables ergonomic patterns like literal types, generic instantiation from context, and type-narrowing in branches.

3. **Inference is not a loophole** — The compiler must ensure that every inferred type is uniquely determined at compile time. Ambiguous inference results in a compile-time error, not an arbitrary default.

4. **Inferred types are inspectable** — The Schema Provider and Compiler Introspection API expose inferred types. An LLM or IDE can query the type of any expression without running the program.

5. **No inference across module boundaries** — Public API types are always explicit. Inference does not cross modules — a module's public surface is fully specified by annotations. This ensures that module consumers never need to run the inference engine to understand an API.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inference Algorithm Policy | Determines the inference algorithm: local (function-body only), bidirectional (top-down + bottom-up), or global (whole-program). Default: bidirectional within function bodies. |
| Annotation Requirement Policy | Specifies which positions require explicit type annotations (function parameters, return types, public API) and which allow inference. |
| Error Reporting Policy | Controls how inference failures are reported: pinpoint the unresolved expression, show expected type from context, list candidate types. |
| Generics Inference Policy | Determines how generic type arguments are inferred at call sites: full inference, inference with explicit turbofish/annotation fallback, or mandatory annotation. |
| Module Boundary Policy | Controls whether inference crosses module boundaries. Default: no — public APIs are always fully annotated. |

## Model (What)

### Bidirectional Inference

Inference proceeds in two directions simultaneously:

```orthon
// Bottom-up: the type of 42 is inferred as Int
let x = 42

// Top-down: the expected type Float constrains the literal
let y: Float = 42    // 42 is inferred as Float, not Int

// Bidirectional: generic type argument inferred from usage
fn identity[T](value: T) -> T
    return value

let z = identity(42)  // T is inferred as Int from the argument
let w: Float = identity(42)  // T is inferred as Float from expected return type
```

### Inference at Function Boundaries

```orthon
// Parameter and return types are explicit
fn add(a: Int, b: Int) -> Int
    let result = a + b     // inferred: result is Int
    return result

// Generic parameters may be inferred from usage at call sites
fn first[T](list: List[T]) -> Option<T>
    if list.is_empty()
        return None
    return list[0]

let item = first([1, 2, 3])  // T inferred as Int
```

### Inference with Literal Types

```orthon
// Literal types are inferred when used in const-like positions
fn create_array() -> Array<Int, 3>
    return [0, 0, 0]   // size 3 inferred from literal count

// Type widening: literal Int widens to Float when context demands it
let threshold: Float = 100   // 100 widened to Float(100)
```

### No Cross-Module Inference

```orthon
// Module A — public API
pub fn parse(input: String) -> Result<Json>
    // full inference inside body
    let trimmed = input.trim()
    let parsed = json_decode(trimmed)
    return parsed

// Module B — consumer sees only the signature
// No need to infer anything about parse's internals
let data = parse(raw_text)
// data is known to be Result<Json> from the signature alone
```

## Default Strategy

Bidirectional inference within function bodies, explicit annotations at function boundaries and all public API surfaces. Generic type arguments inferred at call sites, with optional turbofish-style disambiguation (`::<T>`). No cross-module inference. Inferred types exposed via the Compiler Introspection API and Schema Provider for LLM consumption.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Global inference** | OCaml, Haskell | Most concise but complex error messages; refactoring can silently change inferred types across large codebases. |
| **Local inference only** | Rust, Kotlin | Simple, predictable error messages. Function boundaries create natural documentation points. Cannot infer generic type arguments from expected return type. |
| **Full annotation required** | Java (pre-10), Go | Maximum explicitness but high verbosity; type changes propagate through annotations. |
| **Gradual inference with dynamic fallback** | TypeScript, mypy | Flexible but loses compile-time guarantees in untyped regions; boundary checks add complexity. |
| **Declaration-site inference only** | C++ `auto` | Simple rule but some patterns (generic lambdas, return type deduction) require multiple passes. |

## Open Questions

1. Should there be an explicit annotation to request global inference (e.g., `auto`-like keyword at module level)?
2. How should inference interact with ownership and borrowing — can the borrow checker infer lifetimes from inferred types?
3. Should inference be allowed in public API signatures for backwards-compatible evolution (like C++ `auto` return type deduction)?
4. How does inference interact with error unions — should the inferred error set be part of the public API?
5. Should there be an IDE/LLM command to "materialize" inferred types as explicit annotations for documentation purposes?
6. Can inference span function boundaries within the same module without creating fragility?

## References

- [`DECLARATION_BY_ASSIGNMENT.md`](../important/DECLARATION_BY_ASSIGNMENT.md) — declaration mechanism and inference coupling
- [`GRADUAL_TYPING.md`](../important/GRADUAL_TYPING.md) — gradual typing and boundary inference
- [`GENERICS.md`](./GENERICS.md) — generic type argument inference
- [`TYPED_HOLES.md`](../deferrable/TYPED_HOLES.md) — hole type inference
- [`STRUCTURAL_TYPING.md`](../important/STRUCTURAL_TYPING.md) — structural type satisfaction and inference
- [`LITERAL_TYPES.md`](../important/LITERAL_TYPES.md) — literal type inference and widening
- [`ERROR_UNION.md`](./ERROR_UNION.md) — inferred error sets
- [`FOUNDATIONAL_ABSTRACTIONS.md`](./FOUNDATIONAL_ABSTRACTIONS.md) — compiler-inferred representation choices
- [`COMPILER_AS_STATIC_ANALYZER.md`](./COMPILER_AS_STATIC_ANALYZER.md) — broader compiler verification beyond type inference
