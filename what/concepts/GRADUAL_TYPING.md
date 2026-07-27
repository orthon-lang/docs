# Gradual Typing

## Issue (Why)

How does a language support both rapid prototyping (dynamic typing, no annotation overhead) and production safety (static checking, compile-time guarantees)? Languages typically choose one side:
- **Fully dynamic** (Python, Ruby, JavaScript) — fast iteration, but no compile-time type checking.
- **Fully static** (Rust, Java, C++) — compile-time guarantees, but verbose annotations and slow feedback during early development.

The same programmer needs dynamic speed while sketching and static safety while shipping. The language should not force a permanent choice.

## Principles

1. **Type inference everywhere** — Types are inferred for all expressions. Explicit annotations are never required, but always accepted.
2. **Boundary checking** — Type annotations on function signatures, struct fields, and public APIs act as compiler-checked contracts.
3. **Global inference** — The compiler performs whole-program type inference to catch mismatches even in unannotated code.
4. **No separate declaration files** — Unlike TypeScript's `.d.ts`, type information lives alongside code.
5. **REPL-first** — The REPL and one-off scripts operate fully dynamically. Type checking becomes active as the codebase grows.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Inference Policy | Determines the inference algorithm (local vs global, Hindley-Milner vs bidirectional) |
| Gradual Typing Policy | Governs the interaction between typed and untyped code boundaries |
| Boundary Checking Policy | Specifies what is checked at typed/untyped boundaries |
| Error Reporting Policy | Controls when type errors are reported (always, at boundaries only, or deferred to runtime) |

## Model (What)

Types are always optional at definition sites but can be declared at key boundaries:

```orthon
# Fully dynamic — no annotations, type inferred
name = "Alice"          # type String inferred
age = 30                # type Int inferred
items = []              # dynamic — resolved at use site

# Immutable binding
max_retries := 3        # := denotes immutable binding, type Int inferred

# Annotated boundary — compiler checks at this point
fun greet(person: Person) -> String:
    "Hello, {person.name}!"

# Mixed — some annotations, some inference
fun process(data):
    # data is dynamic, body uses duck typing
    data.transform().collect()
```

### Typed / Untyped Boundary

When a typed function calls an untyped one (or vice versa), the compiler inserts a **boundary check**:

```orthon
fun parse(input: String) -> Int:
    # body is typed — compiler guarantees Int return

fun main():
    data = fetch_data()          # dynamic — no type known
    result = parse(data)         # boundary check: data must be String-like
```

## Default Strategy

Type inference uses a **bidirectional algorithm** (local type information + top-level propagation). Unannotated functions are treated as dynamic at their boundaries but internally inferred. The compiler runs a **global consistency pass** as an optional lint, not a hard error.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Full static inference (Hindley-Milner) | Complete global inference — no annotations needed (OCaml, Haskell). Higher compile-time cost. |
| Optional type annotations (Python `typing`) | Runtime-ignored annotations; checked by external tools. No compiler integration. |
| TypeScript model | Superset language with `.d.ts` declaration files. Requires separate emit step. |
| Dynamic only | No type system. All checking deferred to runtime. |

## Open Questions

1. How does gradual typing interact with algebraic data types and pattern matching exhaustiveness?
2. Performance cost of boundary checks — can they be optimised away when types align?
3. How to handle generic functions in a gradual system — full monomorphisation or erased at boundaries?

## Decision History

- **EDR-059:** Gradual Typing accepted as Language feature — optional type annotations require compiler-level type checking that can be selectively enabled/disabled.
- **Classification per D-03:** Language. The ability to selectively enable/disable type checking at module boundaries requires compiler-level infrastructure. The boundary check mechanism, type inference, and consistency passes are compiler services not expressible via primitives.
- **LLM adoption significance:** Gradual typing is critical for LLM adoption — it allows LLM-generated code to start with minimal annotations while the compiler catches structural errors. The Schema Provider can expose typing level per module for LLM context.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] `how/process/DECISION_PIPELINE.md`
