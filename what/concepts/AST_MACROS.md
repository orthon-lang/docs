# AST Macros

> **✅ ACCEPTED — [EDR-029](../how/decision_records/architecture/EDR-029-ast-macros.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`COMPILE_TIME_EXECUTION.md`](COMPILE_TIME_EXECUTION.md),
> [`GENERICS.md`](GENERICS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Macro, Derive,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Languages solve metaprogramming through four distinct mechanisms — generics, reflection, annotations/attributes, and compile-time code generation. Each introduces its own sublanguage, its own failure modes, and its own learning curve. The core problem: **how does Orthon provide powerful metaprogramming — code that writes code — without introducing multiple special-purpose sublanguages?**

Orthon's answer: **AST macros as functions** — a single macro mechanism where macros are ordinary Orthon functions annotated to run at compile time, receiving typed AST nodes and returning typed AST nodes. There is no separate macro-definition language (no `macro_rules!`, no procedural-macro API with token streams). Combined with a unified `comptime` model ([`COMPILE_TIME_EXECUTION.md`](COMPILE_TIME_EXECUTION.md)) for general compile-time computation, macros handle the specific need for AST-level code generation.

## Principles

1. **Macros are first-class Orthon functions** — not a separate sublanguage. A macro is an ordinary function with a `@macro` annotation.
2. **Typed AST nodes** — Macro input and output are typed AST nodes, not raw token streams. No token-splitting or character-level manipulation.
3. **Hygienic by default** — Identifiers introduced inside a macro do not leak into the calling scope. Opt-in `#` prefix for unhygienic access.
4. **Compiler verifies macro output** — The returned AST is type-checked and well-formedness-checked just like hand-written code.
5. **Deterministic and order-independent** — Macro expansion proceeds in a single pass. No macro depends on the expansion of another macro (no recursive expansion).
6. **No side effects** — Macros cannot perform IO, access the filesystem, or introduce side effects beyond code generation.
7. **`@derive` as syntactic sugar** — The most common macro pattern (trait-implementation generation) receives dedicated declarative syntax.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Macro Hygiene Policy | Defines default-hygienic scoping rules and the `#` opt-in mechanism |
| Macro Expansion Policy | Specifies expansion order, determinism guarantees, and recursion prohibition |
| Derive Registry Policy | Manages the registry of `derive`-compatible macro implementations |
| AST Type Policy | Defines the AST node type system exposed to macro functions |
| Compiler Verification Policy | Governs type-checking and well-formedness verification of macro output |

## Model (What)

### Macro Declaration

A macro is declared with the `@macro` attribute on a function. The function signature declares the kind of AST node it accepts and the kind it produces:

```orthon
@macro
fun json_serialize(type_def: TypeDef) -> Vec<ImplBlock>
    # inspect type_def fields, generate ImplBlock nodes
```

The compiler checks that:
- The macro is only invoked on matching AST node types.
- The returned AST nodes are type-checked after expansion.

### Derive Sugar

The most common macro pattern — trait implementation generation — receives dedicated syntax:

```orthon
@derive(Show, Eq, Clone)
type Point(x: Int, y: Int)
```

This is equivalent to invoking registered derive macros for `Show`, `Eq`, and `Clone`, each generating an `impl Trait for Point` block. The compiler maintains a registry of `derive`-compatible macros. If a trait name has no registered derive macro, the compiler reports an error.

### Hygiene

Macros are hygienic by default: identifiers introduced by a macro expansion are scoped to that expansion and cannot collide with identifiers in the calling scope. Unhygienic access (to the caller's scope) uses an explicit `#` prefix:

```orthon
@macro
fun counter_generator() -> Expr
    # Uses `#` to access caller's scope for an unhygienic binding
    return `#counter += 1`
```

### Expansion Order

Macro expansion proceeds in a single pass:
1. Parse source into AST.
2. Identify macro invocations (`@macro` functions, `@derive` annotations).
3. Expand all macros, substituting returned AST nodes.
4. Type-check the expanded AST.

No macro may depend on the expansion of another macro. This eliminates phase-ordering bugs and makes macro expansion predictable.

### Debugging

```text
wvy build --expand-macros    # Show AST after macro expansion, before type checking
```

This lets the programmer inspect generated code without running a separate tool.

## Default Strategy

Macros are expanded in a single pass during compilation, before type checking of expanded code. All macros are compiled as ordinary Orthon functions and executed in the comptime interpreter. The `@derive` registry is populated by the standard library and can be extended by user code.

## Alternative Strategies

| Strategy | Trade-offs |
|---|---|
| **Procedural macros (Rust-style)** | More powerful (token streams) but more complex — separate compilation unit, separate API. Rejected for Orthon because typed AST nodes provide sufficient power with better safety. |
| **Template-based macros (C preprocessor style)** | Simple but unsafe — no type checking, no hygiene. Rejected. |
| **Unified comptime only (Zig-style)** | No dedicated macro mechanism — all code generation via comptime. Elegant but makes macro contracts harder to discover. Orthon adopts both: comptime for general computation, AST macros for structured code generation. |

## Open Questions

1. Should macro expansion support conditional expansion (expand only if a feature gate is enabled)?
2. How should the `@derive` registry interact with traits defined in external dependencies?
3. Should macros be able to inspect the AST of imported modules, or only the current module?

## Decision History

- **2026-07-27:** Accepted via EDR-029. AST macros defined as functions operating on typed AST nodes. Hygienic by default. Single-pass expansion. `@derive` as declarative sugar. Cross-ref with COMPILE_TIME_EXECUTION established — macros build on comptime but provide a structured AST-layer mechanism.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [ ] `GLOSSARY.md` — added "Macro", "Derive", "Hygienic Macro"
- [ ] `COMPILE_TIME_EXECUTION.md` — cross-reference
- [ ] `PRIMITIVE_BLOCKS.md`
