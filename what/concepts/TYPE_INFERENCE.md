# Type Inference

> **✅ ACCEPTED — [EDR-027](../how/decision_records/architecture/EDR-027-type-inference.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`EQUALITY.md`](EQUALITY.md),
> [`GENERICS.md`](GENERICS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Type Inference, Bidirectional Inference,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How does a variable, expression, or function parameter get its type without an explicit annotation — and what are the trade-offs of different inference strategies?

Type inference sits at the intersection of conciseness and clarity:

- **Too much annotation** (Go, Java pre-10) — every variable carries an explicit type. Maximum clarity; maximum verbosity. Refactoring is tedious.
- **Too little annotation** (OCaml, Haskell) — types inferred everywhere. Maximum conciseness; but error messages can be opaque and understanding complex inferred types requires mental execution of the inference algorithm.

The core problem: **find the right balance** where inference eliminates noise inside function bodies while annotations document contracts at API boundaries.

## Principles

1. **Inference within functions, explicit at boundaries** — Local variables and intermediate expressions are inferred. Function parameters and return types are annotated.
2. **Bidirectional inference** — Bottom-up (from expressions) and top-down (from expected type context) inference flows meet at each expression node.
3. **No cross-module inference** — Public API surfaces are fully annotated. Module consumers never need to run the inference engine to understand a module's API.
4. **Inferred types are inspectable** — The Schema Provider and Compiler Introspection API expose inferred types for any expression.
5. **No ambiguous inference** — If inference is ambiguous, the compiler reports an error. No silent defaults.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inference Algorithm Policy | Determines inference algorithm — bidirectional within function bodies |
| Annotation Requirement Policy | Specifies which positions require explicit type annotations |
| Generics Inference Policy | Determines how generic type arguments are inferred at call sites |
| Module Boundary Policy | Controls whether inference crosses module boundaries — default: no |

## Model (What)

### Bidirectional Inference

Inference flows in two directions simultaneously:

```orthon
// Bottom-up: 42 is inferred as Int
let x = 42

// Top-down: expected type Float constrains the literal
let y: Float = 42    # 42 is inferred as Float, not Int

// Bidirectional: generic type argument inferred from both sides
fn identity[T](value: T) -> T
    return value

let z = identity(42)          # T inferred as Int from argument
let w: Float = identity(42)   # T inferred as Float from expected return type
```

### Function Body Inference

Within a function body, every local expression, variable, and intermediate type is inferred:

```orthon
fn compute(a: Int, b: Int) -> Int
    let sum = a + b             # sum is Int
    let doubled = sum * 2       # doubled is Int
    let strings = ["a", "b"]    # strings is List[String]
    return doubled
```

### Generic Type Argument Inference

At call sites, generic type arguments are inferred from argument types and expected return type:

```orthon
fn first[T: Iterator](items: T) -> Option<T::Item>
    for item in items
        return Some(item)
    return None

let result = first([1, 2, 3])       # T inferred as List[Int]
let result: Option[Float] = first(data)  # T inferred from expected return type
```

### Turbofish Disambiguation

When inference is ambiguous, turbofish `::<T>` disambiguates:

```orthon
fn parse[T](input: String) -> Result<T, ParseError>

let num = parse::<Int>("42")        # explicit type argument
let text = parse::<String>("hello") # explicit type argument
```

### Public API Boundaries

Function parameters and return types at public API surfaces require explicit annotations:

```orthon
// Public API — fully annotated
pub fn find_user(id: Int) -> Result<User, DbError>
    let query = "SELECT * FROM users WHERE id = ?"   # inferred inside body
    return db.execute(query, id)

// Private function — inference recommended but not required
fn internal_helper(data: String) -> Int    # explicit, acts as documentation
    let parsed = parse_config(data)        # inferred
    return parsed.count
```

### Deferred Syntax

The concrete `: Type` annotation syntax is determined in Phase 5 (Syntax). The semantic model established here is independent of the concrete syntax used for annotations.

## Default Strategy

Bidirectional inference within function bodies. Explicit annotations at function boundaries and all public API surfaces. Generic type arguments inferred at call sites. Turbofish `::<T>` for disambiguation. No cross-module inference. Inferred types exposed via Schema Provider.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Global inference** | OCaml, Haskell | Most concise; complex error messages; refactoring silently changes inferred types across large codebases |
| **Full annotation** | Go, Java (pre-10) | Maximum explicitness; high verbosity; refactoring propagates through annotations |
| **Gradual inference** | TypeScript, mypy | Flexible; loses compile-time guarantees in untyped regions; boundary checks add complexity |
| **Unidirectional bottom-up** | Simple type checkers | Cannot handle contextual inference — expected return type cannot constrain generics |

## Open Questions

1. Should there be an explicit annotation to request global inference within a module (e.g., `auto`-like keyword)?
2. How should inference interact with ownership and borrowing — can the borrow checker infer lifetimes from inferred types?
3. Should inference be allowed in public API signatures for backwards-compatible evolution (like C++ `auto` return type deduction)?
4. How does inference interact with error unions — should the inferred error set be part of the public API signature?
5. Should there be an IDE/LLM command to "materialize" inferred types as explicit annotations for documentation purposes?

## Decision History

- **Local bidirectional inference** adopted over global inference and full annotation. Rationale: Best balance of ergonomics and explicitness, proven by Rust and Kotlin.
- **Explicit at public API boundaries** adopted over full inference. Rationale: Module consumers must not need to run the inference engine to understand APIs.
- **No cross-module inference** adopted. Rationale: Preserves module boundary contract — public surface is fully specified by annotations.
- **Defer `: Type` syntax to Phase 5** adopted. Rationale: The semantic model of inference is independent of the concrete annotation syntax.
- **Accepted via EDR-027** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/EQUALITY.md`
- [ ] `what/concepts/GENERICS.md`
- [ ] `what/concepts/TRAITS.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
