# Interactive Development (REPL-Driven)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language support rapid feedback loops during development — allowing the programmer to evaluate code incrementally, inspect intermediate state, and modify the program without restarting from scratch?

Two fundamentally different development paradigms exist:

1. **Edit-compile-run cycle (Java, Rust, Go, C++)** — The programmer writes source files, compiles, and runs the full program. Feedback time is proportional to compile time + startup time. For large programs, this cycle can take seconds to minutes. Incremental compilation helps but does not eliminate the restart — every change starts the program from the beginning, losing all accumulated state.

2. **REPL-driven / interactive development (Clojure, Lisp, Smalltalk, Elixir)** — The programmer connects to a running process (the REPL) and evaluates code incrementally. Functions can be redefined, state can be inspected, and the program evolves without restarting. Feedback is sub-second. The running program is the development environment — not a separate artifact that must be rebuilt.

The core problem: **the edit-compile-run cycle is the dominant model, but it maximises feedback latency**. Every restart loses the program's runtime state, forcing the programmer to re-navigate to the code path they are debugging. REPL-driven development keeps the program running and the programmer in a continuous flow state.

A related question: **does the language design itself support interactive development?** Dynamic dispatch, hot-reloadable namespaces, incremental compilation, and state serialisation are not incidental — they must be designed into the language from the start.

## Principles

1. **Continuous feedback** — The programmer can evaluate any expression and see its result within sub-seconds, without restarting the program.

2. **Live code modification** — Functions, types, and modules can be redefined in the running process. The program adapts without restarting.

3. **State preservation** — When code is modified, existing program state (data, connections, in-memory caches) is preserved. The programmer does not lose context on every change.

4. **Incremental compilation** — Only changed code is recompiled. Compilation is per-form, not per-file or per-project.

5. **Namespace isolation** — Changes in one namespace do not affect other namespaces. A buggy redefinition in a scratch namespace does not corrupt the running system.

6. **Connected debugging** — The REPL provides access to the full state of the running program: stack traces, local variables, function arguments, and module state. No separate debugger attachment is needed.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Compilation Policy | Determines whether incremental compilation (per-form, per-expression) is supported, or only whole-file/whole-project |
| Hot Reload Policy | Governs whether modules can be redefined at runtime, and whether existing state survives redefinition |
| Namespace Policy | Controls whether namespaces are independent runtime units that can be loaded/unloaded independently |
| State Serialization Policy | Determines whether program state can be serialized and restored across REPL sessions |
| Toolchain Policy | Governs whether a REPL is part of the standard language distribution |

## Model (What)

The development workflow is built around a **REPL (Read-Eval-Print Loop)** — a running process that accepts code expressions, evaluates them, prints the result, and waits for the next expression.

### Session-Based Workflow

```
# Start a REPL session
orthon repl

# Evaluate expressions — immediate feedback
orthon> 2 + 2
=> 4

orthon> fn greet(name) -> "Hello, {name}!"
=> #<function greet>

orthon> greet("Alice")
=> "Hello, Alice!"

# Redefine a function — existing state preserved
orthon> fn greet(name) -> "Hi, {name}!"
=> #<function greet>

orthon> greet("Alice")
=> "Hi, Alice!"   # Function was hot-reloaded; no restart needed
```

### File-Driven Development

In practice, REPL-driven development combines a text editor with the REPL:

1. Write code in a `.orthon` file.
2. Send individual forms (function definitions, expressions) to the REPL.
3. The REPL compiles and evaluates each form incrementally.
4. Results appear in the REPL output without leaving the editor.

```orthon
# In editor: user writes this function
fn process(items: List[Int]) -> List[Int]
    items.filter(fn x -> x > 0).map(fn x -> x * 2)

# User sends the definition to the REPL
# REPL responds: #<function process>

# User tests immediately:
# In editor: process([-1, 2, -3, 4])
# Send to REPL => [4, 8]

# User modifies the function:
fn process(items: List[Int]) -> List[Int]
    items.filter(fn x -> x > 0).map(fn x -> x * x)

# Send updated definition to REPL
# REPL redefines the function in-place
# Test again — no restart needed
```

### Comparison of Development Models

| Dimension | Edit-Compile-Run (Java, Go) | REPL-Driven (Clojure, Lisp) | Hybrid (Python, JS) |
|---|---|---|---|
| **Feedback latency** | Seconds to minutes | Milliseconds to sub-second | Sub-second (script), seconds (reload) |
| **State preservation** | None — restart from scratch | Full — running state survives redefinitions | Partial — module reload may reset state |
| **Hot reload** | Limited (JRebel, language server protocol) | First-class (namespace-level redefinition) | Partial (module reload, not always safe) |
| **Incremental compilation** | File-level (incremental compiler) | Expression-level (per-form) | Expression-level (interpreted) |
| **Debugging** | Debugger attach, breakpoints | Inspection via REPL, no breakpoint overhead | Print + debugger |
| **Learning curve** | Low (straightforward model) | Medium (new workflow paradigm) | Low |
| **Production parity** | High (same binary) | Medium (REPL may diverge from compiled) | Medium |

## Default Strategy

Orthon provides a REPL as part of the standard toolchain. The REPL supports:

- **Expression evaluation** — any expression, any context.
- **Hot-reloadable namespaces** — namespaces are independent compilation units that can be reloaded without affecting other namespaces.
- **State inspection** — access to all bindings in the current namespace.
- **Documentation lookup** — inline help for functions, types, and modules.
- **Schema queries** — the Schema Provider is accessible from the REPL, so the programmer can query type definitions, function signatures, and API contracts interactively.

The compiler supports incremental compilation at the expression level during REPL sessions. Functions defined in the REPL are compiled to the same representation as functions in source files — there is no performance difference between REPL-developed and file-developed code.

```
# REPL with Schema Provider integration
orthon> :schema fn process
=> Parameters: [items: List[Int]]
   Returns: List[Int]
   Source: process.orthon:3

orthon> :doc process
=> Filters positive numbers and doubles them.
```

## Alternative Strategies

| Strategy | Description |
|---|---|
| Image-based (Smalltalk, Pharo) | The entire development state is a snapshotable image. Changes persist in the image, not in source files. Highest state preservation; highest coupling to the image. |
| Notebook-based (Jupyter, Pluto) | Cells of code with output below. State is per-kernel. Good for exploration; less suited for application development. |
| Watch-based hot reload (Elixir, JS) | File watchers trigger module reload on save. No explicit REPL session — the development server auto-reloads. Pragmatic compromise for web applications. |
| Compile-and-go (Rust, Go) | No REPL. Fast compiler + language server provides some feedback (type errors, lint). Trade-off: no runtime state inspection without debugger. |

## Open Questions

1. Should the REPL support image snapshots (save-and-restore the full session state)?
2. How does hot reload interact with type safety — can redefining a type break existing values of that type?
3. Should the Execution Program model support a "development" execution strategy that enables REPL features, distinct from "production" strategy?
4. How does the REPL handle multi-file projects — does it auto-detect file dependencies?
5. Can the REPL be embedded in other contexts (notebooks, documentation playgrounds)?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

**See also:**
- [`DEVELOPER_TOOLING.md`](./DEVELOPER_TOOLING.md) — toolchain, package manager, and development environment
- [`EXECUTION_PROGRAM.md`](./EXECUTION_PROGRAM.md) — Execution Program model and execution strategies
- [`SCRIPT_EXECUTION_MODEL.md`](./SCRIPT_EXECUTION_MODEL.md) — one-off script execution
- [`NATIVE_COMPILATION.md`](./NATIVE_COMPILATION.md) — compiled mode vs interpreted mode
