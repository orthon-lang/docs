# Typed Holes

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

How does a language support incremental, incomplete program construction — where the programmer (or an LLM) fills in parts of a program one piece at a time?

Traditional approaches:

- **Stub functions** (C, Java, Python) — the programmer writes `fn foo() { todo!() }` or `throw UnimplementedError`. The type signature is written, but the body is a runtime crash. No compiler guidance on what the body should produce.
- **Placeholder variables** (Python `_`, JavaScript `_`) — conventional but meaningless. The compiler knows nothing about what `_` is supposed to represent.
- **`undefined` / `null` as placeholder** — introduces null into the type system. Every `undefined` is a potential runtime error.
- **Typed holes in dependently typed languages** (Idris, Agda, Haskell with `-XTypedHoles`) — the programmer writes `?hole_name` or `_`, and the compiler reports the expected type, available bindings in scope, and valid hole fillings. This is the closest existing precedent, but remains niche in mainstream languages.

The core problem: **incremental construction requires a way to mark "I'll fill this in later" without breaking type checking or introducing runtime failures**. For LLMs generating code incrementally, typed holes serve as:

1. **Generation targets** — The LLM sees a hole with its expected type and the surrounding context. It generates exactly the expression that fills the hole, with no need to regenerate the entire function.

2. **Self-verification points** — The compiler type-checks everything *around* the hole. When the LLM fills the hole, only the hole's expression and its integration need re-checking.

3. **Incremental scaffolding** — A complex function can be scaffolded with holes for each sub-expression, then filled one at a time. The program remains compilable (with hole warnings) at every step.

## Principles

1. **Holes are syntactically distinct** — A hole is a visible, unambiguous marker in the syntax. It cannot be confused with a valid identifier or a missing expression.

2. **Holes are typed** — Every hole has an expected type, inferred from context. The compiler reports the hole's type, available bindings, and any additional constraints.

3. **Holes produce compile-time warnings, not errors** — A program with unfilled holes compiles with warnings. Holes never cause runtime failures; they are semantically equivalent to unreachable code at runtime.

4. **Holes are fillable by composition** — A hole of type `T` can be filled with any expression of type `T`. Holes can contain sub-holes, enabling compositional filling.

5. **Tooling-visible** — Holes are exposed in the language schema and usable by the LLM Toolchain. The Schema Provider reports holes, their types, and valid filling candidates.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Hole Policy | Controls whether typed holes are allowed, required for incomplete code, or forbidden |
| Hole Checking Policy | Determines when holes are reported (compile-time only, or also in IDE) |
| Hole Filling Policy | Specifies whether the compiler/toolchain can suggest or auto-fill holes |
| Error Policy | Governs whether holes produce warnings or errors in different build modes |

## Model (What)

### Hole Syntax

A hole is written as `??` optionally followed by a name for readability:

```orthon
fn sort(list: List<Int>) -> List<Int>
    ??

fn find_user(id: Uuid) -> Result<User, Error>
    ??find_user
```

The hole name is for tooling and documentation only — it does not affect semantics.

### Typed Holes in Expressions

Holes can appear in any expression position. The compiler infers the expected type and reports it:

```orthon
let doubled = list.map(|x| x * ??)    # Hole expected type: Int
```

Compiler output:

```
-- Hole ------------------------------------------ <stdin>:3:33
?? : Int
Context: x : Int
Available bindings:
  x : Int
  list : List<Int>
```

### Holes as Function Bodies

A hole as a function body marks the function as incomplete:

```orthon
fn process(config: Config) -> Result<Status, Error>
    ??process

fn validate(data: String) -> Bool
    if data.is_empty() then
        ??validate_empty    # sub-hole: expected Bool
    else
        true
```

### Holes in Type Signatures

Holes can appear in type positions, leaving types to be inferred:

```orthon
fn cache<T>(key: String, value: T) -> ??
    # Hole expected type: Result<Cached<T>, CacheError>
    # (inferred from usage context)
```

### Composition

Holes compose naturally — a hole at expression level can contain sub-holes:

```orthon
fn render(item: Item) -> String
    let header = ??  # expected: String
    let body = ??    # expected: String
    header ++ body   # type checks even with holes
```

## Core Inclusion Filter

Before entering formal validation, this concept is run through the Core Inclusion Filter procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 1–2 (Primitive Operation / Language Pattern)

Library test:   Can Typed Holes be implemented as a Standard Library
                function using existing Core constructs?

                Typed holes require syntactic support (the `??` marker)
                and type system integration (hole type inference).
                Neither can be provided by a library function.
                A library could provide a `todo()` function that panics
                at runtime, but that loses the type inference and
                compile-time verification benefits.

                → FAIL (cannot be a library).

Pattern test:   Can Typed Holes be expressed as a composition of existing
                Primitive Operations and Data Model constructs?

                A pattern could use a special `unimplemented<T>()` function
                that returns a value of any type (like Haskell's `undefined`).
                However:
                - No existing primitive provides type inference for
                  a missing expression.
                - No existing primitive provides compiler diagnostics
                  for unfilled code positions.
                - Composition of `function` + `call` + type inference
                  cannot express "this expression is intentionally missing
                  and here is what type it should have."

                → FAIL (cannot be composed from existing primitives).

Primitive test: Is a Typed Hole an atomic operation on data that cannot
                be decomposed?

                A typed hole introduces a new semantic concept: a syntactic
                position whose type is inferred from context but whose value
                is deferred. The combination of syntactic marker + deferred
                value + type inference is atomic — it cannot be decomposed
                into simpler operations. However, the mechanism is close to
                a Language Pattern in that the hole itself has no runtime
                semantics (it simply defers to the filler).

                → PASS — belongs in Primitive Operations (Level 1) with
                  strong Language Pattern characteristics (Level 2).

Filter result: Level 1 (Primitive Operation), with Level 2 (Language
               Pattern) as a secondary classification — the hole's syntactic
               and type-inference support is primitive; the filling pattern
               is a pattern.
```

## Default Strategy

By default, holes produce compile-time warnings but do not block compilation. The compiler reports the type and available bindings for each hole. In release builds, holes produce errors — no incomplete code may be released.

The IDE and LLM Toolchain expose holes as actionable diagnostics: the Schema Provider lists hole types, and the Code Completer can suggest fillings.

## Alternative Strategies

| Strategy | Description |
|---|---|
| **Holes as errors** | Unfilled holes produce compile-time errors in all build modes. Ensures no incomplete code is ever executed, but prevents incremental compilation workflows. |
| **Holes as runtime panics** | Holes compile but panic if reached at runtime. Suitable for test-driven development where tests exercise filled holes before unfilled ones. |
| **No holes** | Incomplete code is simply a compile error. The programmer must use `todo()` or similar library stubs. Zero language support. |
| **Haskell-style typed holes** | Holes produce errors, but GHC suggests valid fillings using available bindings and types. Full IDE-like feedback from the compiler itself. |
| **Idris-style metavariables** | Holes can be inside type signatures too, with the compiler attempting to unify them during type checking. Highest expressiveness, highest complexity. |

## Open Questions

1. Should holes be allowed in type signatures (e.g., `fn foo() -> ??`), or only in expression positions?
2. Should the compiler suggest fillings for holes, or should that be deferred entirely to the LLM Toolchain?
3. How do holes interact with the module system — should modules with holes be publishable as "incomplete"?
4. Should holes support named goals (like Idris `?hole_name`) for documentation and tooling?
5. How does the `??` syntax interact with the null-safe operator or error propagation operator?
6. Should holes be allowed in patterns, or only in expressions and function bodies?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [x] `how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/LLM_STRATEGY.md`
