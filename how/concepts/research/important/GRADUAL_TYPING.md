# Gradual Typing

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

How does a language support both rapid prototyping (dynamic typing, no annotation overhead) and production safety (static checking, compile-time guarantees)?

Languages typically choose one side:
- **Fully dynamic** (Python, Ruby, JavaScript) — fast iteration, REPL-friendly, but no compile-time type checking. Large codebases accumulate type-related bugs that could have been caught statically.
- **Fully static** (Rust, Java, C++) — compile-time guarantees, but verbose type annotations and slow feedback loops during early development. REPL and scripting workflows are painful.

The tension is real: the same programmer needs dynamic speed while sketching and static safety while shipping. The language should not force a permanent choice between the two.

## Principles

1. **Type inference everywhere** — Types are inferred for all expressions. Explicit annotations are never required, but always accepted.
2. **Boundary checking** — Type annotations on function signatures, struct fields, and public APIs act as compiler-checked contracts. Unannotated code is dynamically typed but still benefits from inference.
3. **Global inference** — The compiler performs whole-program type inference (like OCaml/ReasonML) to catch mismatches even in unannotated code.
4. **No separate declaration files** — Unlike TypeScript's `.d.ts`, type information lives alongside code. The language is self-describing.
5. **REPL-first** — The REPL and one-off scripts operate fully dynamically. Type checking becomes active as the codebase grows and annotations appear.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Inference Policy | Determines the inference algorithm (local vs global, Hindley-Milner vs bidirectional) |
| Gradual Typing Policy | Governs the interaction between typed and untyped code boundaries |
| Boundary Checking Policy | Specifies what is checked at typed/untyped boundaries |
| Error Reporting Policy | Controls when type errors are reported (always, at boundaries only, or deferred to runtime) |

## Model (What)

The language uses **gradual typing** with global inference. Types are always optional at definition sites but can be declared at key boundaries.

```orthon
# Fully dynamic — no annotations, type inferred
name = "Alice"          # type String inferred
age = 30                # type Int inferred
items = []              # type List<??> — fully dynamic, resolved at use site

# Immutable binding with :=
max_retries := 3        # := denotes immutable binding, type Int inferred

# Annotated boundary — compiler checks at this point
fun greet(person: Person) -> String
    "Hello, {person.name}!"

# Mixed — some annotations, some inference
fun process(data)
    # data is dynamic, body uses duck typing
    data.transform().collect()
```

Key features:
- **`name = value`** — type-inferred, mutable binding (default).
- **`name := value`** — type-inferred, immutable binding.
- **`fun name(args) -> ReturnType`** — annotated function boundary. Return type is optional.
- **No `.d.ts` equivalent** — type information is embedded in source files.
- **`dynamic` escape hatch** — explicit opt-out of static checking for specific values.

### Typed / Untyped Boundary

When a typed function calls an untyped one (or vice versa), the compiler inserts a **boundary check**:

```orthon
fun parse(input: String) -> Int
    # body is typed — compiler guarantees Int return

fun main()
    data = fetch_data()            # dynamic — no type known
    result = parse(data)           # boundary check: data must be String-like
```

The boundary check behaves like a type assertion at compile time: if the dynamic value does not match the expected type, the compiler flags it (or falls back to a runtime check in fully dynamic mode).

## Default Strategy

Type inference uses a **bidirectional algorithm** (local type information + top-level propagation). Unannotated functions are treated as dynamic at their boundaries but internally inferred. The compiler runs a **global consistency pass** as an optional lint, not a hard error — enabling iterative development.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Full static inference (Hindley-Milner) | Complete global inference — no annotations needed, all types known at compile time (OCaml, Haskell). Higher compile-time cost. |
| Optional type annotations (Python `typing`) | Runtime-ignored annotations; checked by external tools (mypy, pyright). No compiler integration. |
| TypeScript model | Superset language with `.d.ts` declaration files and full static checking. Requires a separate emit step. |
| Dynamic only (Python, Ruby) | No type system. All checking deferred to runtime. |

## Open Questions

1. How does gradual typing interact with algebraic data types and pattern matching exhaustiveness?
2. Should `dynamic` be an explicit type keyword or an implicit default?
3. Performance cost of boundary checks — can they be optimised away when types align?
4. How to handle generic functions in a gradual system — full monomorphisation or erased at boundaries?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] Other: `how/concepts/research/DATA_MODEL.md`
