# AST Macros as Functions

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It proposes AST macros as a metaprogramming mechanism — functions that
> operate on the syntax tree and produce syntax tree — as an alternative
> to the unified `comptime` model documented in `COMPILE_TIME_EXECUTION.md`.
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Hypothesis

Macros in Orthon are **functions that operate on the AST and return AST**.
There is no separate macro-definition language (no `macro_rules!`, no
procedural-macro API with token streams). A macro is an ordinary Orthon
function annotated to run at compile time, receiving a typed AST node and
returning a typed AST node.

```
@macro
fun derive_show(type_def: TypeDef) -> Vec<ImplBlock>
    # inspect type_def, generate ImplBlock nodes
    ...
```

The `@macro` annotation signals that the function:
1. Executes at compile time — no runtime overhead.
2. Receives parsed, type-checked AST nodes (not raw token streams).
3. Returns AST nodes that are type-checked again after macro expansion.
4. Cannot perform IO, access the filesystem, or introduce side effects
   beyond code generation.

Derives (`@derive`) are a restricted form: they are declarative annotations
that the compiler resolves to known macro implementations, without the
programmer writing the macro body.

```
@derive(Show, Eq, Clone)
type Point(x: Int, y: Int)
```

The compiler treats `@derive(Show, Eq, Clone)` as macro invocations that
generate `impl Show for Point`, `impl Eq for Point`, and `impl Clone for Point`
blocks. The user never sees the generated code unless they request it via
a compiler flag (`--expand-macros`).

## Hypothesis

- Macros are first-class Orthon functions, not a separate sublanguage.
- Macro input and output are typed AST nodes — no raw token manipulation.
- The compiler validates the returned AST (type checks, well-formedness)
  just like hand-written code.
- `@derive` is syntactic sugar for the most common macro pattern:
  trait-implementation generation.
- Macro expansion is deterministic and order-independent (no
  phase-ordering bugs).
- Macros are hygienic by default: identifiers defined inside a macro
  do not leak into the calling scope.

## Issue (Why)

Languages solve metaprogramming through four distinct mechanisms:

| Mechanism | Language | Cost |
|-----------|----------|------|
| **Generics** | Rust, Java, C# | Separate syntax, trait bounds, monomorphisation |
| **Reflection** | Java, C#, Python | Runtime overhead, type-unsafe, breaks encapsulation |
| **Annotations/Attributes** | Java, C#, Rust proc macros | Annotation processor pipeline, separate API |
| **Compile-time execution** | Zig `comptime` | Unified but requires careful phase distinction |
| **Code generation** | `@derive`, `@builder` | Declarative — limited to known patterns |

Each mechanism introduces its own sublanguage, its own failure modes, and
its own learning curve. The core problem: **how does Orthon provide powerful
metaprogramming — code that writes code — without introducing multiple
special-purpose sublanguages?**

Existing Orthon research explores two poles:
- **`COMPILE_TIME_EXECUTION.md`** proposes Zig-style `comptime`: one unified
  mechanism where generics, reflection, and metaprogramming are all ordinary
  code running at compile time. No macros, no annotations.
- **`METAOBJECTS.md`**, **`GENERICS.md`**, **`REFLECTION_ALTERNATIVES.md`**
  propose separate mechanisms for each concern.

This document proposes a third pole: **AST macros as functions** — a single
macro mechanism that coexists with declared generics. The macro mechanism
is not a unified `comptime`, but it is not four separate mechanisms either.
It is one mechanism: a function that manipulates the AST at compile time.

## Level

**Core** — Macro semantics are part of the language specification, not a
library or tooling concern. Macro expansion happens during compilation,
before type checking of the expanded code.

## Proposal

### 1. Macro Declaration

A macro is declared with the `@macro` attribute on a function:

```
@macro
fun json_serialize(type_def: TypeDef) -> Vec<ImplBlock>
    # inspect type_def fields
    # generate serialize/deserialize implementations
    ...
```

The function signature declares:
- **Input type:** The kind of AST node the macro accepts (`TypeDef`,
  `FnDecl`, `Pattern`, etc.). The compiler checks that the macro is only
  invoked on matching AST nodes.
- **Return type:** The kind of AST node(s) the macro produces. The compiler
  type-checks the generated code after expansion.

### 2. Derive Sugar

The most common macro pattern — generating trait implementations — receives
dedicated syntax:

```
@derive(Show, Eq, Clone)
type Point(x: Int, y: Int)
```

This is equivalent to:

```
type Point(x: Int, y: Int)

// Compiler-expanded:
impl Show for Point { ... }
impl Eq for Point { ... }
impl Clone for Point { ... }
```

Each trait name in `@derive` refers to a registered macro implementation.
The compiler maintains a registry of `derive`-compatible macros (typically
trait implementations). The `@derive` annotation is checked at compile time:
if `Show` has no registered derive macro, the compiler reports an error.

### 3. Hygiene

Macros are hygienic by default:
- Identifiers introduced by the macro are scoped to the macro expansion
  and cannot collide with identifiers in the calling scope.
- The macro can opt into unhygienic behaviour explicitly using a `#`
  prefix (e.g., `#counter` to access the caller's scope). This mirrors
  Rust's `$crate` but is opt-out rather than opt-in.

### 4. Expansion Order

Macro expansion proceeds in a single pass:
1. Parse the source into an AST.
2. Identify macro invocations (functions with `@macro`, `@derive` annotations).
3. Expand all macros, substituting the returned AST nodes.
4. Type-check the expanded AST.

No macro can depend on the expansion of another macro (no recursive
macro expansion). This eliminates phase-ordering bugs and makes macro
expansion predictable.

### 5. Debugging

The compiler provides `--expand-macros` to show the AST after macro
expansion but before type checking. This lets the programmer inspect
generated code without running a separate tool.

## Trade-offs

| Aspect | AST macros (this proposal) | Unified `comptime` (Zig style) |
|--------|---------------------------|-------------------------------|
| **Model** | Functions over AST — one mechanism for code generation | One execution phase — generics, reflection, and codegen are all `comptime` |
| **Generics** | Separate — declared trait bounds (see `GENERICS.md`) | Integrated — `comptime` parameters subsume generics |
| **Learning curve** | Medium — programmer learns AST types | Medium — programmer learns phase distinction |
| **Code generation power** | High — arbitrary AST manipulation | High — arbitrary comptime code |
| **Compile-time safety** | Higher — macro input/output are typed AST nodes | Medium — compile errors surface at instantiation sites |
| **IDE/LLM support** | Higher — macro contracts are explicit (input type, output type) | Medium — comptime code can do anything, harder to analyse |
| **Risk of abuse** | Lower — macros cannot perform IO, no side effects | Higher — comptime code can do anything the language allows |
| **Implementation complexity** | Medium — AST type system, macro registry | High — comptime interpreter, phase management |
| **Precedent** | Rust proc macros, Scala macros, Elixir macros | Zig, Jai, Roc |

### Why not unified `comptime`?

The unified `comptime` model (Zig, Jai) is elegant: one mechanism for all
compile-time work. However, it requires the compiler to include a full
interpreter (or compile-and-run engine) for the language itself at compile
time. This adds significant compiler complexity and makes compile-time
execution harder to sandbox (comptime code can theoretically do anything
the language allows, including side effects).

AST macros constrain the power: a macro can only inspect and produce AST
nodes. It cannot perform arbitrary computation at compile time, which is
a restriction, not a bug — it makes macros predictable, testable, and
analysable by the IDE and LLM toolchain.

### Why not Rust-style procedural macros?

Rust's proc macros operate on opaque token streams, not typed AST nodes.
The macro author works with `TokenStream` — a flat sequence of tokens —
and must parse it manually or use the `syn` crate. Errors are reported
against the token stream, not the typed AST. This makes proc macros
difficult to write, debug, and maintain.

Orthon's AST macros receive a typed AST node. The compiler has already
parsed and validated the input; the macro works with a structured
representation. Errors are reported against the typed AST, giving precise
diagnostics.

### Why not Java-style annotation processors?

Annotation processors operate in a separate compilation phase, on a
representation (the `Elements` API and `TypeMirror`) that is different
from the parser's AST. They require registering the processor in a
`META-INF/services` file, have no control over processing order, and
cannot generate code that depends on other generated code.

Orthon's macros are part of the same compilation pipeline. No separate
registration, no separate API: a macro is just a function with a `@macro`
attribute.

## Related Concepts and Alternatives

| Document | Relationship |
|----------|-------------|
| [`COMPILE_TIME_EXECUTION.md`](COMPILE_TIME_EXECUTION.md) | Proposes unified `comptime` — the main alternative to this document. Both address metaprogramming through different mechanisms |
| [`GENERICS.md`](GENERICS.md) | Declared trait-bound generics — separate from macros. Macros can generate `impl` blocks for generic types |
| [`METAOBJECTS.md`](../deferrable/METAOBJECTS.md) | Proposes three distinct mechanisms (traits, annotations, macros) — this document collapses annotations and macros into one |
| [`METADATA_ANNOTATIONS.md`](../deferrable/METADATA_ANNOTATIONS.md) | Explores annotation/attribute syntax — `@derive` in this document is a concrete proposal within that space |
| [`REFLECTION_ALTERNATIVES.md`](../deferrable/REFLECTION_ALTERNATIVES.md) | Runtime reflection alternatives — macros are a compile-time alternative to runtime reflection |
| [`HOMOICONICITY.md`](../deferrable/HOMOICONICITY.md) | Code-as-data — macros inherently require the language to represent its own AST as data |
| [`COMPILE_TIME_EXECUTION.md`](COMPILE_TIME_EXECUTION.md) § The Four Documents | Describes how GENERICS.md, STRUCTURAL_TYPING.md, METAOBJECTS.md, and REFLECTION_ALTERNATIVES.md each propose separate mechanisms |
| [`imperative-crutch-metaprogramming.md`](../imperative-crutch-metaprogramming.md) | Analysis of metaprogramming pain points in imperative languages |

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Macro Expansion Policy | Determines when and how macros are expanded (single-pass, recursive, or ordered) |
| Macro Hygiene Policy | Controls identifier scoping rules — hygienic by default with explicit escape hatch |
| Derive Resolution Policy | Governs how `@derive` names are resolved to macro implementations |
| AST Representation Policy | Defines the types used in macro signatures (typed AST node kinds) |
| Compile-time Safety Policy | Restricts what macro bodies can do (no IO, no side effects, deterministic) |

## Open Questions

1. Should macros support recursive expansion (a macro expanding into code
   that contains another macro invocation)?
2. How does the IDE/LLM toolchain discover which derive macros are available
   for a given trait?
3. Should macros be usable in expression position (e.g., `my_macro!(expr)`)
   in addition to declaration position (`@derive`, `@macro`)?
4. What is the caching and incremental compilation story for macros?
5. How do macros interact with the Execution Program model — can a macro
   generate an Execution Program?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
