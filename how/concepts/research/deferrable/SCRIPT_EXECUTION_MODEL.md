# Script Execution Model

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

When a source file is executed, where does execution begin?

Two contrasting models exist:

1. **Mandatory entry point** (Java, C, C++, Go, Rust) — every executable program must define a function with a specific name and signature (`public static void main(String[] args)` in Java, `int main()` in C). The runtime calls this function; nothing executes outside it. This provides a clear, predictable starting point but adds ceremony: even the simplest program requires a class/function wrapper.

2. **Top-down script execution** (Python, JavaScript, Ruby) — statements at the top level of a file execute in order when the file runs. Functions and classes are defined when their `def`/`class` statements are reached. The "main" pattern is a convention (`if __name__ == "__main__":` in Python, checking `require.main === module` in Node.js), not a language requirement. This reduces ceremony at the cost of implicit execution order: top-level statements with side effects can execute unexpectedly when a file is imported rather than run directly.

The core problem: **how does the language distinguish between a file being executed as a program versus being imported as a module, and where does execution actually start?**

A related problem: **how does this interact with Orthon's Execution Program model** — where a program's semantic meaning is decoupled from its execution strategy? If the same source can be interpreted, compiled, or containerized, the entry point model must be independent of the execution strategy.

## Principles

1. **Top-down execution** — Statements at module level execute in order when the file is run directly. This includes variable declarations, function definitions (which bind the function name), and any top-level expressions.
2. **Explicit module boundary** — When a file is imported, only its definitions are loaded; top-level expressions with side effects do not execute unless explicitly guarded.
3. **Guarded entry point** — A convention (not a language requirement) marks the program entry point. The compiler/runtime recognizes this marker and executes the block only when the file is the main program.
4. **Zero ceremony for simple programs** — A single-expression program should require no enclosing function, class, or boilerplate.
5. **Compatible with Execution Program model** — The entry point mechanism must work identically across all Execution Program strategies (interpret, compile, containerize, WASM).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Entry Point Policy | Determines whether entry is mandatory (`main` function), conventional (`if __name__`), or implicit (top-level only) |
| Module Loading Policy | Governs what executes when a module is imported vs. run directly |
| Top-Level Expression Policy | Controls which expressions are allowed at module level (declarations only, or any expression) |
| Execution Program Policy | Ensures the entry point model is compatible with the Execution Program abstraction |

## Model (What)

Execution starts at the top of the file and proceeds downward. Function and type definitions bind names. A special guard expression marks the program entry point — code inside this guard executes only when the file is the main program, not when it is imported as a module.

```
# file: hello.or
# Top-down: this runs when the file is executed directly

say_hello("Orthon")          # This runs — top-level expression

fun say_hello(name: String):  # Function definition — binds the name
    print("Hello, {name}!")

# Guarded entry point — only runs when this file is the main program
entry:
    print("Starting program...")
    say_hello("World")
    print("Done.")

# When imported as a module:
# - `say_hello` is available to the importer
# - `say_hello("Orthon")` does NOT execute (see Open Question 2)
# - `entry:` block does NOT execute
```

Key features:
- **`entry:` block** — a special block that executes only when the file is the main program. Multiple `entry:` blocks in the same file are allowed (executed in definition order).
- **Function/type definitions** — bind names when executed (top-down), but their bodies are not evaluated until called.
- **No `main` function** — no required name or signature. `entry:` can contain any valid code.
- **Import semantics** — when a file is imported, only definitions are processed. `entry:` blocks and other top-level expressions (outside `entry:`) do not execute on import (see Open Question 2).
- **Compatible with Execution Program** — `entry:` works identically whether the program is interpreted, compiled to native, or packaged for WASM.

### Comparison with Java and Python

| Aspect | Java | Python | Orthon (proposed) |
|---|---|---|---|
| Entry point | `public static void main(String[] args)` | `if __name__ == "__main__":` | `entry:` block |
| Ceremony | High (class + method + signature) | Low (one conditional) | Low (one keyword) |
| Top-level expressions | Not allowed (must be in class) | Allowed (execute on import) | Allowed (guarded by `entry:`) |
| Side effects on import | N/A (no top-level) | Yes (unintended) | No (guarded) |
| Multiple entry points | One per class | One per file (convention) | Multiple `entry:` blocks |
| Program arguments | `String[] args` | `sys.argv` | `args` (implicit in `entry:`) |

## Default Strategy

Files execute top-down. `entry:` blocks define the program entry point. When imported as a module, only top-level definitions (functions, types, constants) are processed — `entry:` blocks and top-level expressions outside definitions are skipped.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Mandatory `main` function | A single `main` function is required (Java, C, Rust). Clear but ceremonial for small programs. |
| No entry point (always top-down) | No special marker. All top-level code executes on both direct run and import (early Python behavior). Simple but causes accidental side effects. |
| Module-level declarations only | No expressions at module level — everything must be inside a function (Go). Clean but rigid. |
| Named entry with CLI integration | Entry point is a function with structured CLI argument parsing (Rust's `clap`, Go's `flag`). Best for complex tools but over-engineered for simple scripts. |

## Open Questions

1. Should `entry:` accept implicit program arguments (like `entry(args):` where `args` is a typed argument list)?
2. Should top-level expressions outside `entry:` execute on import, or only on direct execution? Python's approach (all top-level code runs on import) is simple but causes side-effect problems. Orthon could choose:
   - Only definitions execute on import (expressions skipped).
   - All top-level code executes on both import and direct run (Python model).
   - Only `export`-marked declarations execute on import.
3. How does `entry:` interact with testing — should test modules also have an `entry:` or a separate `test:` block?
4. Should `entry:` support async by default (i.e., `entry:` implies `async main`)?
5. How does `entry:` compose with Execution Programs — can an Execution Program override which `entry:` block is used?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/concepts/research/EXECUTION_PROGRAM.md`
- [ ] `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- [ ] Other: `how/architecture/ARCHITECTURE.md`
