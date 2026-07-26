# Expression-Oriented Language

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

Should control flow constructs (`if`, `when`, `try`, `switch`) produce values, or only execute side effects?

Two design traditions:

1. **Statement-oriented** (Java, C, Go) — `if` is a statement. It controls which block executes but produces no value. Assigning a conditional result requires a ternary operator (`?:`) or mutable variable with assignment in each branch:

   ```java
   String status;
   if (age >= 18) {
       status = "adult";
   } else {
       status = "minor";
   }
   ```

   The ternary operator exists as a limited expression form but supports only two branches and no complex logic.

2. **Expression-oriented** (Kotlin, Scala, Rust, most functional languages) — `if`, `match`/`when`, and `try` are expressions. They produce values that can be assigned, passed as arguments, or returned:

   ```kotlin
   val status = if (age >= 18) "adult" else "minor"
   ```

The core problem: **should every control flow construct in Orthon be an expression that returns a value, or should there be a statement/expression distinction?**

Expression orientation has cascading effects: it eliminates mutable temporary variables, enables more concise code, and — critically for Orthon's LLM Readiness goal — produces code with fewer mutable state mutations for an LLM to track.

## Examples

| Construct | Statement-oriented | Expression-oriented |
|---|---|---|
| **Conditional** | `if (c) { a } else { b }` — no value | `result = if c then a else b` |
| **Multi-branch** | `switch (x) { case A -> ... }` — statement | `result = when x { A -> ... B -> ... }` |
| **Error handling** | `try { ... } catch { ... }` — statement | `result = try compute() catch recover()` |
| **Assignment** | `a = b = c` — often disallowed | `a = b = c` — chainable |
| **Block** | `{ stmt; stmt; }` — no value | `{ expr; expr; last_expr }` — last expression is value |

## Implications for Orthon

1. **All control flow is expression-oriented** — `if`, `when`, `try`, and blocks all return values. Every syntactic construct that controls flow also produces a value:

   ```
   status = if age >= 18 then "adult" else "minor"
   ```

2. **Last-expression value** — A block's value is the last expression in the block:

   ```
   greeting = {
       prefix = "Hello"
       name = load_name()
       "$prefix, $name!"     // this is the block's value
   }
   ```

3. **No ternary operator** — Since `if` is already an expression, a ternary operator (`?:`) is redundant. Remove it to keep the core minimal.

4. **No statement/expression split in the grammar** — Every construct is potentially an expression. Side-effecting code (assignments, function calls returning `Unit`) produce the `Unit` value when used in expression position.

5. **Compiler-enforced exhaustiveness** — Because `if`/`when`/`try` are expressions, the compiler must ensure every branch produces a value of the same type. Missing branches become compile-time errors, not runtime `NullPointerException`s.

6. **Explicit discard** — If an expression value is intentionally not used, an explicit discard syntax (e.g., `_ = compute()`) signals the intent to the compiler and to readers.

7. **LLM benefit** — Expression-oriented code has fewer mutable variables, reducing the state an LLM must remember when generating or analysing code. Each expression is a self-contained value computation.

## Open Questions

1. Should `return` from a function be an expression (`return value` produces type `Never` / bottom) or reserved as a statement?
2. How does expression orientation interact with imperative-style loops? Is `for` an expression (producing `Unit` or `List[T]`)?
3. Should assignment (`x = value`) be an expression producing the assigned value (enabling chaining) or a statement only?
4. How does the compiler report type mismatches in expression-oriented branches — errors or warnings?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `PATTERN_MATCHING.md`
- [ ] `ERROR_HANDLING.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/GLOSSARY.md`
- [ ] `DESIGN_PRINCIPLES.md`
