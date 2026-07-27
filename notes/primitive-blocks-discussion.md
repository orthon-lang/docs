# Primitive Blocks — Discussion Record

> **Date:** 2026-07-27
> **Phase:** 03 (Primitive Blocks)
> **Context:** Assumptions-mode discussion between solo author and AI agent,
>   capturing clarifications and corrections that shaped the final CONTEXT.md.

## Topics Covered

### 1. What Are Primitive Blocks?

**Clarification:** Primitives are **atomic operations**, not keywords or data types.
They are the semantically irreducible operations from which all language constructs
are composed. The organizing framework is Data + Data Modifiers (from
`FOUNDATIONAL_ABSTRACTIONS.md`), but the primitives themselves are concrete:
`identifier`, `literal`, `assignment`, `function`, `call`, `attribute access`,
`scope`, `reference`, `pack`/`unpack`.

Types (`Int`, `String`, `List`) and type constructors (`struct`, `class`) are NOT
primitives — they are built from `pack` + `reference` + `scope`.

### 2. `call` vs `function` — Why Separate?

`function` = declaration construct (what is computed). `call` = evaluation
construct (how computation is triggered). They serve different semantic dimensions:
function addresses *definition*, call addresses *evaluation*.

Separation allows:
- Function as first-class value without call (pass as argument, store)
- Different evaluation strategies (eager/lazy) affecting call, not function
- Uniform call syntax regardless of declaration form

### 3. `delegate` — Why Excluded from Primitive Set

`delegate` is an **execution policy** (`DELEGATE.md`), not an atomic operation.
It wraps a state-owning entity in a delegated execution context. Decomposes into:
`reference` (who owns the state) + `function` (what to execute) + ownership semantics.

Distinct from ownership transfer: delegate is about *execution context isolation*,
ownership transfer is about *who is responsible for a value*.

**Open question (Phase 3):** Should ownership/aliasing itself be a primitive rather
than a composition of `reference` + `scope`? The user flagged this as worth revisiting.

### 4. `namespace` — What It Is

Namespace is a logical grouping mechanism (C#-style), independent of file layout.
Not a module system — decomposes to `identifier` + `scope` + visibility.
Module system (if any) is a separate Phase 4 concept.

### 5. `operator definition` — Syntactic Sugar

Per `DESIGN_PRINCIPLES.md` § Named Before Symbolic: every symbolic operator must
have an equivalent named function. Operator definition is syntactic sugar over
function definition, not a primitive.

```
3 + 4     // symbolic form
add(3, 4) // equivalent named form — same semantics
```

### 6. First-Class Values

Functions are first-class. Blocks are expressions (expression-oriented model).
Not every first-class value needs a literal form — functions are, blocks are
(through `{ }`), but the language doesn't require literal syntax for all types.

### 7. Blocks (`{ }`) and Return

**Decision (user correction):** `{ }` blocks require explicit `return` to produce
a value. The "last expression is the value" rule (Phase 2 D-04) applies to
expression-level constructs (`if`, `match`, `when`), NOT to `{ }` block syntax.

```orthon
let x = {
    let tmp = compute()
    return tmp + 1   // explicit return
}
```

Rationale: Distinguishes block-as-scope (side effects) from block-as-expression.

### 8. `emit` — Lazy by Default

**Correction to Phase 2 D-04:** `emit` is lazy by default — it produces values
on demand. For eager production, use `return` with an aggregate collection.

This aligns with Sequence as *what*, not *how*. Lazy emission is the Evaluation
Policy for `emit`; Phase 4 concepts (iterators, generators) build on lazy emit.

### 9. `@` for Metadata Access (Metadata Protocol)

**Decision:** All metadata, protocol methods, and special operations accessed via
`@` prefix notation, NOT via double-underscore conventions (`__len__`, `__getitem__`).

```
list@len()       // instead of list.__len__()
obj@fields       // reflective structural access
type@name        // type metadata
```

Rationale: Syntactically visible, distinct from `.`, LLM-generable (no magic
method names to memorise). Named "Metadata Protocol" in GLOSSARY.md.

Free functions like `len(list)` may exist as syntactic sugar compiling to
`list@len()` — deferred to Phase 4/5.

### 10. Free Functions — Open Design Problem

Functions like `len()`, `sorted()`, `str()` require:
1. A protocol system (`@`-prefixed methods) — locked (D-07)
2. A resolution mechanism mapping free calls to protocol methods
3. Clear boundary between language-provided and user-defined functions

Deferred to Phase 4/5.

### 11. System Functions via Metadata Protocol

System functions like `len()`, `sorted()` map to `@`-prefixed protocol methods.
Not dunder methods, not implicit. The user's notation: `list@len()` rather than
`list.__len__()` or `list.len()`.

### 12. Decorators

Not special syntax (`@annotation`). Decorator = function composition: a function
that takes a callable and returns a callable. First-class functions make this
trivial without dedicated syntax. Compile-time decorator transformation is
`AST_MACROS.md` / `COMPILE_TIME_EXECUTION.md` territory (Phase 4).

### 13. Descriptor vs Metadata Protocol

**Correction:** What the AI initially called "Descriptor" (Python's `__get__`/
`__set__` protocol) should be **Metadata Protocol** — the `@`-based mechanism
for intercepting special-method access.

**Descriptor** in Orthon is something different: a handle combining `id` + `gen`
(identity + generation counter) — analogous to an ECS entity handle or tagged
pointer with versioning. The `gen` field detects use-after-free without full GC.

### 14. Actor/Delegate Model — No Coroutines

Orthon chooses actor/delegate (via `delegate` + `<-` operator) over traditional
coroutines. Rationale:
- `<-` makes async call syntactically visible (avoids forgotten-await bugs)
- Delegate tied to ownership model (state owner = unit of serialization)
- Simpler for LLM generability

`await` may exist as a library-level construct over `delegate` + synchronization,
not as a language primitive.

### 15. First-Class Values in Orthon

All values are first-class (expression-oriented). No statement/expression split.
`if`, `match`, `when`, blocks — all produce values.

### 16. Actor Not a Primitive

Actor (`class` + `act`) is a language-level convenience over `delegate`, which
itself composes `reference` + `function` + ownership. None of these are new
primitives — they are compositions of existing ones.

## Key Corrections to Initial Assumptions

| Assumption | Correction | Source |
|-----------|------------|--------|
| `emit` could be eager or lazy | `emit` is lazy by default; eager = `return` | User |
| Descriptor = `__get__`/`__set__` protocol | That's **Metadata Protocol**. Descriptor = `id + gen` handle | User |
| `{ }` blocks last-expression-is-value | `{ }` blocks require explicit `return` | User |
| Metadata via dunder methods | Metadata via `@` prefix (Metadata Protocol) | User |
| Free functions work automatically | Free functions are open design problem | User |

## Open Questions for Phase 3

1. Ownership/aliasing — should it be a separate primitive beyond `reference`?
2. `mut` vs `&mut` — one keyword or two?
3. Interior mutability — primitive or derived?
4. Closure mutation — how does immutability-by-default apply to captures?
