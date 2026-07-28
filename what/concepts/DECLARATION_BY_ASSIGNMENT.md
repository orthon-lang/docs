# Declaration by Assignment

## Issue (Why)

How does a variable come into existence? Two fundamentally different answers exist:
1. **Explicit declaration** (Java, C) — variable must be declared with its type before use. Verbose but explicit.
2. **Declaration by assignment** (Python, JS) — variable is created by its first assignment. Concise but risks typos creating new variables.

The core questions: how does the language balance conciseness with safety, and how does the type of a variable get determined?

## Principles

1. **First assignment is declaration** — A variable is introduced by its first assignment. No separate declaration keyword.
2. **No accidental creation** — Assigning to an undeclared variable is an error, not variable creation.
3. **Type inferred from initializer** — The type is determined by the initializing expression.
4. **Read before write is an error** — Compile-time definite assignment analysis.
5. **Shadowing allowed with explicit marker** — Shadowing requires an explicit keyword.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Declaration Policy | Determines whether variables are created by keyword, by assignment, or by explicit type |
| Type Inference Policy | Governs how variable type is determined |
| Definite Assignment Policy | Controls compile-time "read before write" detection |
| Shadowing Policy | Determines whether shadowing requires explicit syntax |

## Model (What)

```orthon
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
let items = transform(items)    # shadowing: new variable, different type
```

## Default Strategy

Declaration by assignment with compile-time definite assignment analysis. First assignment creates the variable. Variables are immutable by default (`mut` required for reassignment). Shadowing allowed with `let` keyword.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Keyword declaration (Rust/Swift/Kotlin) | Variables require `let` or `var`. More ceremony but explicit. |
| Mandatory type annotation (Java) | Variables require explicit type. Self-documenting but verbose. |
| No shadowing | Prevents confusion but forces awkward naming. |

## Open Questions

1. Should `let` for shadowing be required in all cases or only when the shadowed variable is still in scope?
2. How does declaration by assignment interact with pattern matching destructuring?
3. Should `const` (compile-time constant) be a separate concept from immutable variables?

## Decision History

- **EDR-074:** Declaration by Assignment accepted — borderline with Phase 5 (Syntax). The declaration model affects how variables are introduced and the concrete syntax for type annotations, shadowing, and mutability. The semantic decisions (first-assignment-is-declaration, definite assignment analysis, type inference from initializer) are specified here. The concrete syntax choices (keyword vs. no-keyword, annotation syntax) are deferred to Phase 5.
- **Classification per D-03:** Borderline with Phase 5 (Syntax). The semantic core (first assignment declares, definite assignment analysis, immutability by default) is language-level. Concrete syntax (how type annotations look, whether `let` is required for shadowing) belongs to Phase 5. The semantic decisions are processed here; syntax is deferred.
- **Phase 5 boundary:** The following are deferred to Phase 5: concrete syntax for type annotations (`: Type` vs `as Type`), the exact keyword for shadowing (`let` vs `var` vs other), and whether `mut` is a keyword or a modifier.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/SYNTAX.md` (Phase 5)
- [ ] `how/process/DECISION_PIPELINE.md`
