# Declaration by Assignment

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a variable come into existence? Two fundamentally different answers exist:

1. **Explicit declaration** (Java, C, Go) — a variable must be declared with its type before use: `Type name;`. The declaration and the assignment are separate operations. This provides a clear inventory of all variables at the cost of verbosity and repetition.

2. **Declaration by assignment** (Python, JavaScript, Ruby) — a variable is created by its first assignment. There is no separate declaration step. The identifier simply appears on the left side of `=`. This is concise but introduces risks: typos create new variables instead of errors, and there is no compile-time guarantee that a variable exists before use.

The core problem: **how does the language balance conciseness (no redundant declarations) with safety (no accidental variable creation)?**

A related but distinct problem: **how does the type of a variable get determined** — must it be annotated, inferred from the initializer, or a mix of both? The declaration mechanism and the type inference mechanism are coupled: declaration-by-assignment languages tend toward full type inference, while explicit-declaration languages tend toward mandatory type annotations.

## Principles

1. **First assignment is declaration** — A variable is introduced by its first assignment. There is no separate declaration keyword (`let`, `var`, `Type`).
2. **No accidental creation** — Assigning to an undeclared variable is an error, not a new variable creation. The compiler must be able to determine, at compile time, whether a variable exists in the current scope.
3. **Type is inferred from initializer** — The type of a variable is determined by its initializing expression. Explicit type annotations are available for disambiguation but are not required.
4. **Read before write is an error** — Reading a variable before it has been assigned (in any execution path) is a compile-time error (definite assignment analysis).
5. **Shadowing allowed with explicit marker** — Variable shadowing (re-declaring a name in an inner scope) is permitted but must be syntactically visible (e.g., a `let` or `var` keyword that signals *new variable, not reassignment*).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Declaration Policy | Determines whether variables are created by keyword (`let`, `var`), by assignment, or by explicit type declaration |
| Type Inference Policy | Governs how the type of a variable is determined — full inference, partial inference with annotations, or mandatory annotations |
| Definite Assignment Policy | Controls compile-time analysis for "read before write" detection |
| Shadowing Policy | Determines whether shadowing is permitted, disallowed, or permitted only with explicit syntax |

## Model (What)

A variable is introduced by its first assignment. The type is inferred from the initializer. Reassigning to an existing variable is legal only if the variable is declared mutable. Shadowing is permitted but requires an explicit re-declaration keyword.

```
# Declaration by assignment — type inferred from "Hello" → String
greeting = "Hello, Orthon!"

# Explicit type annotation for disambiguation
count: Int = 42

# Read before write is a compile-time error
# print(value)  ← ERROR: value not yet assigned
value = compute()

# Mutable variable — explicit keyword
mut counter = 0
counter += 1      # OK: mut allows reassignment

# Shadowing — explicit re-declaration
mut items = load_all()
# ...
items = filter(items, is_valid)    # reassignment (items is mut)

let items = transform(items)       # shadowing: new variable, different type
# The outer `items` is no longer accessible
```

Key features:
- **No `let` for first declaration** — assignment is declaration. This is the most common case and should be the least ceremony.
- **`let` for shadowing** — the keyword signals "I know I'm creating a new variable with this name, potentially hiding an outer one."
- **`mut` for mutable variables** — variables are immutable by default; mutability must be explicitly requested.
- **Definite assignment analysis** — the compiler tracks whether a variable has been assigned on all paths before any read.
- **No implicit globals** — assignment inside a function always creates a local variable (unlike Python's `global`/`nonlocal` requirement).

### Comparison with Python and Java

| Aspect | Java | Python | Orthon (proposed) |
|---|---|---|---|
| Declaration | `Type name` (must declare) | First assignment (no keyword) | First assignment (no keyword) |
| Reassignment | Always allowed | Always allowed | Only with `mut` |
| Type annotation | Mandatory | Optional (PEP 484) | Optional (inferred) |
| Shadowing | Allowed with type | Allowed (no marker) | Allowed with `let` marker |
| New variable on typo | Compile error | Runtime error (NameError) | Compile error (definite assignment) |
| Implicit globals | N/A (all declared) | Yes (assignment = local, `global` keyword) | No (assignment = local, always) |

## Default Strategy

First assignment creates the variable. Type is inferred from the initializer. Variables are immutable by default (`mut` required for reassignment). Shadowing is allowed but requires `let`. Definite assignment analysis is strict: every path must assign before read.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Keyword declaration | Variables require `let` or `var` keyword (Rust, Swift, Kotlin). More ceremony but makes declaration explicit. |
| Mandatory type annotation | Variables require explicit type; no inference (Java pre-10, C). Verbose but self-documenting. |
| No shadowing | Shadowing is always an error. Prevents confusion but forces awkward naming (`count`, `count2`). |
| Dynamic scoping | Variable lookup follows the call stack, not the lexical scope (early Lisp). Rare and error-prone. |

## Open Questions

1. Should `let` for shadowing be required in all cases, or only when the shadowed variable is still in scope? (Rust makes it always required.)
2. How does declaration by assignment interact with pattern matching — does `case (x, y) =>` declare `x` and `y`?
3. Should there be a separate syntax for destructuring declarations (e.g., `(a, b) = tuple` creating two new variables)?
4. How does definite assignment analysis work across module boundaries (function calls)?
5. Should `const` (compile-time constant) be a separate concept from immutable variables?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] Other: `what/SYNTAX.md`
