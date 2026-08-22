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
4. **Comptime granularity — parameter vs. block vs. function.** Is comptime a
   property of a single parameter (`comptime T: type`), a block
   (`comptime:` / `comptime {}`), or a whole function? The three are not
   interchangeable:
   - **Parameter** passes a `type` as a value; it is needed only if `type`
     remains a first-class comptime value. If generics migrate to `<>`
     (EDR-086), parameter-level comptime narrows to reflection helpers.
   - **Block** is the only way to express non-macro compile-time work —
     compile-time constants, compile-time assertions, local metaprogramming —
     and cannot be reduced to a parameter.
   - **Function** already exists in specialised form as `@macro` (EDR-029);
     a general non-macro comptime function marker is sugar over "all
     parameters comptime + body comptime", not a new semantic.
   Resolution order: settle generics first (EDR-031 vs EDR-086), since
   parameter-level necessity depends on that outcome; block-level necessity
   is independent of it.
5. **Comptime as deferred invocation (delegate analogy).** Orthon's unified
   Invocation model (EDR-085) already models deferred execution:
   `delegate(obj)` hands an object to a serialised actor context, `defer(obj)`
   to a coroutine, and `<-` sends messages. Should comptime be modelled by
   the same analogy — comptime code "hands control to the compiler" for
   execution during compilation? Open sub-questions:
   - The target differs: `delegate`/`defer` hand control to a *runtime*
     context (actor/coroutine); comptime would hand control to the
     *compiler*. Is this the same Invocation pattern (EDR-085) or a distinct
     axis?
   - Reusing `<-` would overload the message-send operator; what symbol or
     keyword would mark "hand control to the compiler"?
   - EDR-031 Principle #3 ("comptime is not a separate language — the same
     Orthon code runs at comptime") already rules out a separate execution
     model; a delegate-style comptime must remain the same language, merely
     an earlier phase, not a distinct runtime.
6. **Block syntax and terminology.** The accepted block form `comptime:` reads
   awkwardly; candidates include brace blocks (`comptime {}`), a full-word
   marker (`compiletime {}`), or a mode keyword. Constraint: `#` (unhygienic
   macro access, EDR-029), `@` (metadata/reflection prefix), `with`
   (copy-with-modify, EDR-042), and `const`/`static` (literal preservation
   EDR-043 / Static allocation policy) are all already loaded. This must be
   resolved consistently with Orthon's block style (colon-blocks such as
   `using x = expr:` vs brace-blocks in research examples).
7. **Symbolic comptime markers (`@` / `#` / `<>`).** Could a symbol replace
   the `comptime` keyword? Analysis: `@` is already the Metadata Protocol
   prefix (reflection `@typeInfo`, `@derive`, `@macro`); `#` is already
   unhygienic macro access (EDR-029); `<>` is already type parameters and
   type application (EDR-086). Reusing any of them for a comptime
   phase/parameter marker would overload a loaded symbol and conflate
   orthogonal axes (type vs value vs phase). Proposals to treat `<>` as a
   general "compile-time frame" for expressions/blocks additionally collide
   with the `<`/`>` comparison operators. The simplification is not a new
   symbol: remove `comptime T: type` from the generics path (generics =
   `<>`), leaving `@` (metadata) + one phase/block keyword (OQ 6).
8. **`requires` is not a trait bound.** `requires` is a boolean predicate
   over *values* — CONSTRAINED_TYPES (EDR-080: `type Age = Int requires
   v >= 0 && v <= 150`) and CONTRACTS (EDR-056: `requires x >= 0.0`) —
   enforced at value boundaries. A trait bound is an assertion about a
   *type* (trait satisfaction), resolved at compile time. Proposals such as
   `fun max(a: T, b: T): T requires Comparable as T` conflate value
   constraint with type constraint. The signature tail already separates
   them: `where T as Hash + Eq` for types vs `requires` for values. Call
   sites use type inference (`max(1, 2)`) or turbofish (`max::<Int>(1, 2)`),
   not `using T` — `using` is taken by resource (`using x = expr:`) and
   context (`(using ord: Ord[A])`) clauses.
9. **Comptime as invocation-in-context.** EDR-085 unifies deferred execution
   under context constructors (`delegate`, `defer`, `spawn`, `fork`) with
   submission operators (`<-`, `|>`) and materialisation (`take`, `await`,
   `next`). Could comptime be a fifth execution policy — an invocation
   evaluated during compilation? Analysis: conceptually yes — comptime is an
   execution-policy axis ("when code runs"), and the colourless-function
   model extends naturally. Syntactically no: the delegate/actor machinery
   (persistent context object, mailbox, message queue, ownership transfer,
   runtime lifetime) has no compile-time analogue. `@typeInfo(Point)` is
   already a "comptime invocation" — the `call` primitive evaluated in the
   comptime phase, marked by `@`. There is no `comptime_ctx <- fn(args)`;
   comptime has no runtime lifecycle. Invocation-with-context is a runtime
   mechanism; comptime is a phase mechanism — orthogonal axes.

## Synthesis (Draft Thesis — 2026-08-22)

> The distilled thesis is captured in
> [`THESES.md`](../THESES.md) § Compile-Time Execution — three
> orthogonal axes. This section records the full analysis behind it.

Compile-time execution (Zig-style comptime) is needed: it moves work from
the runtime to the compiler. But "comptime" is not one homogeneous
mechanism — it names three different things by nature, each answering a
different question:

1. **Type inference / generics** — *which types does this code work with?*
   Answered at compile time by type inference and specialisation. Surface
   form: the angle-bracket diamond `<>` (EDR-086) or an equivalent
   `where`-clause form. Already settled — and it never needs the word
   "comptime": `<>` marks the compile-time nature implicitly.
2. **Metadata / reflection during compilation** — *what is the structure
   of this type or value?* Answered at compile time via the Metadata
   Protocol. Surface form: `@` — `@typeInfo`, `@field`, `@hasDecl`,
   `@derive`, `@macro`. Already unified; `@` marks the compile-time
   evaluation implicitly.
3. **Comptime parameters, blocks, or whole functions** — *when does this
   code run?* The phase axis, and the only axis that needs an explicit
   phase marker. Granularity is open (OQ 4), but the candidates are not
   equal:
   - **Block** — the irreducible form: the only way to express non-macro
     compile-time work (compile-time constants, compile-time assertions,
     local metaprogramming). Cannot be reduced to a parameter or a
     function marker.
   - **Function** — sugar, not a primitive: "this function runs at
     compile time" = "all parameters comptime + body executes in a
     comptime context". Its specialised case already exists as `@macro`
     (EDR-029) — a function executed in the comptime phase to generate
     AST.
   - **Parameter** (`comptime T: type`) — needed only while `type`
     remains a first-class comptime value; once generics live in `<>`
     (EDR-086), parameter-level comptime narrows to reflection helpers.

Key observation: axes 1 and 2 are already settled and never use the word
"comptime" — their markers (`<>`, `@`) denote compile-time-ness implicitly.
Only axis 3 needs an explicit phase marker, and only for work the other two
do not cover. The `comptime T: type` parameter form (EDR-031) is exactly
where axis 1 (types) was expressed through axis 3 (phase) — the conflation
that causes the syntax tension. Keeping the three axes separate (types via
`<>`, metadata via `@`, phase via one keyword) dissolves the tension: no
`comptime T: type`, no `@`/`#`/`<>` reuse, no `requires`-as-bound. Status:
draft thesis for Phase 5, not a decision.

## Decision History

- **2026-07-27:** Accepted via EDR-031. Unified comptime model adopted (Zig-inspired) with explicit trait bounds for LLM discoverability. Cross-ref with GENERICS established — comptime IS the generic mechanism. Cross-ref with AST_MACROS established — macros execute in comptime. LLM Generability Gate identified as critical with documented restrictions.
- **2026-08-22:** Design review recorded the comptime syntax tension. Open
  Questions 4–9 added (granularity, delegate analogy, block syntax,
  symbolic markers, `requires` vs bounds, comptime invocation). Draft
  thesis: comptime splits into three orthogonal axes — generics (`<>`),
  metadata (`@`), and phase (one keyword). No decision; feeds Phase 5.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/AST_MACROS.md` — cross-reference
- [ ] `what/concepts/GENERICS.md` — cross-reference
- [ ] `how/DESIGN_PRINCIPLES.md`
