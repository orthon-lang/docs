# Compile-Time Execution (Unified Comptime)

> **✅ ACCEPTED — [EDR-031](../how/decision_records/architecture/EDR-031-compile-time-execution.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **⚠️ LLM GENERABILITY GATE — Critical.** This concept has significant
> implications for LLM tooling. See § LLM Generability for restrictions.
>
> **See also:** [`GENERICS.md`](GENERICS.md),
> [`AST_MACROS.md`](AST_MACROS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Comptime,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Should generics, duck-typed polymorphism, reflection, and metaprogramming each get their own dedicated language mechanism, or should a single compile-time execution model serve all four needs at once?

Zig answers this question by collapsing all four into one homogeneous mechanism: `comptime`. Any function parameter can be declared `comptime`, including parameters whose declared type is `type` itself. There is no `<T>` bracket syntax, no separate generic-parameter list, and no declared trait/interface bounding what `T` must support. Constraint checking is deferred to instantiation: the compiler substitutes the concrete type and compiles the function body as written.

Orthon adopts a **unified comptime model** inspired by Zig: a single `comptime` keyword serves generics, reflection, and metaprogramming with the same semantics as runtime code, executed in an earlier phase. However, Orthon's comptime model is more constrained than Zig's — it includes **explicit comptime bounds** for discoverability and LLM generability, diverging from Zig's fully duck-typed approach.

### Relationship to Existing Mechanisms

| Mechanism | Relationship |
|---|---|
| **Generics** | Comptime *is* the generic mechanism. `comptime T: type` parameters replace separate `<T>` generic syntax. Trait bounds on comptime parameters provide declared contracts (diverging from Zig's duck-typed approach). See [`GENERICS.md`](GENERICS.md) for the interaction model. |
| **AST Macros** | Comptime provides the execution engine for [`AST_MACROS.md`](AST_MACROS.md). Macro functions are ordinary Orthon functions annotated with `@macro`, executed in the comptime phase. Comptime and macros are complementary: comptime for general compile-time computation, macros for structured AST-level code generation. |
| **Reflection** | Comptime replaces runtime reflection. `@typeInfo(T)`, `@field(value, name)` are comptime-evaluated function calls. |
| **Metaprogramming** | Comptime enables code that generates code via ordinary `if`/`for`/`while` control flow at compile time. |

## Principles

1. **Same semantics, earlier phase** — Comptime code uses the same language semantics as runtime code, just executed during compilation. No separate sublanguages.
2. **Explicit comptime bounds** — Generic comptime parameters can be annotated with trait bounds for discoverability. This diverges from Zig's fully duck-typed approach to preserve IDE/LLM tooling support.
3. **Comptime is not a separate language** — No separate grammar, no separate type system, no separate execution model. The same Orthon code runs at comptime.
4. **Deterministic** — Comptime evaluation is deterministic and free of side effects on the external world. No IO, no filesystem access, no network.
5. **Comptime code is visible** — Functions that execute at comptime must be visibly marked (the `comptime` keyword at the definition site). A reader can tell from local syntax whether code runs at compile time or runtime.
6. **LLM generability is critical** — Comptime's generics and reflection model must be LLM-generable. See § LLM Generability for the constraints that follow from this principle.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Comptime Execution Policy | Defines which functions and expressions execute at compile time vs. runtime |
| Comptime Bound Policy | Governs trait bounds on comptime parameters for discoverability |
| Comptime Sandbox Policy | Restricts comptime code from performing IO, filesystem access, or network operations |
| Monomorphisation Policy | Defines how generic comptime parameters are specialised for concrete types |
| Reflection Policy | Specifies which `@typeInfo`-style operations are available at comptime |

## Model (What)

### Comptime Parameter

A comptime parameter is declared with the `comptime` keyword before the parameter name:

```orthon
fun max(comptime T: type, a: T, b: T) -> T
    # T is resolved at compile time
    return if a > b then a else b
```

The `comptime T: type` syntax means `T` is resolved at compile time. Unlike Zig's fully duck-typed approach, Orthon allows optional trait bounds:

```orthon
fun max(comptime T: type + Comparable, a: T, b: T) -> T
    # T must implement Comparable — checked at comptime
    return if a > b then a else b
```

### Comptime Block

Explicit comptime blocks execute at compile time:

```orthon
comptime:
    # This block executes during compilation
    type_info = @typeInfo(MyType)
    assert(type_info.fields.len > 0)
```

### Comptime Reflection

```orthon
comptime:
    # Type introspection
    fields = @typeInfo(Point).fields
    
    # Field access by name
    field_value = @field(point_instance, "x")
    
    # Declaration presence check
    has_method = @hasDecl(MyType, "serialize")
```

### Generics via Comptime

Comptime replaces separate generic syntax. A generic function is simply a function with a `comptime` parameter:

```orthon
# Generic function — no separate <T> syntax
fun identity(comptime T: type, value: T) -> T
    return value

# Instantiation
result = identity(Int, 42)     # T = Int
result = identity(String, "hello")  # T = String
```

See [`GENERICS.md`](GENERICS.md) for the complete generics model.

## Default Strategy

Comptime code executes in a sandboxed interpreter during compilation. Monomorphisation generates specialised copies for each concrete type. Comptime blocks are evaluated eagerly. Comptime functions with trait bounds have their bounds checked before monomorphisation.

## LLM Generability

The unified comptime model introduces specific challenges for LLM-based code generation:

### Restrictions

1. **Explicit trait bounds required in public APIs** — Generic comptime parameters in public functions MUST be annotated with trait bounds. This ensures an LLM (or human) can determine what a generic function requires without reading the function body.
2. **Private/internal comptime parameters** may omit bounds (duck-typed), following Zig's model for internal code where discoverability is less critical.
3. **Comptime blocks must have syntactically visible scope** — The `comptime:` keyword and `@` prefix for reflection operations make comptime code locally identifiable.
4. **Schema Provider exposure** — The Schema Provider MUST expose comptime parameter bounds and comptime block boundaries for LLM tooling consumption.

### Rationale

The LLM generability constraint is the primary reason Orthon's comptime model diverges from Zig's: a fully duck-typed `comptime` parameter requires an LLM to read the function body to determine what operations are available on a type parameter. With explicit trait bounds, the LLM can determine the contract from the signature alone.

## Alternative Strategies

| Strategy | Trade-offs |
|---|---|
| **Full duck-typed comptime (Zig)** | Maximum flexibility, no annotation burden. Contract discoverability requires reading function bodies — worse for LLM tooling. |
| **Separate generic syntax (Rust)** | Declared trait bounds by default. Better discoverability but adds a second mechanism (generics) alongside comptime. |
| **Runtime reflection only (Java)** | No compile-time execution. Simpler compiler but runtime performance cost and less safety. |

## Open Questions

1. Should comptime code support cross-module evaluation (evaluating comptime code from dependencies)?
2. How should comptime interact with incremental compilation — can comptime results be cached?
3. Should comptime support compile-time allocation with arena semantics?

## Decision History

- **2026-07-27:** Accepted via EDR-031. Unified comptime model adopted (Zig-inspired) with explicit trait bounds for LLM discoverability. Cross-ref with GENERICS established — comptime IS the generic mechanism. Cross-ref with AST_MACROS established — macros execute in comptime. LLM Generability Gate identified as critical with documented restrictions.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/AST_MACROS.md` — cross-reference
- [ ] `what/concepts/GENERICS.md` — cross-reference
- [ ] `how/DESIGN_PRINCIPLES.md`
