# Homoiconicity (Code as Data)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language enable programs to inspect, transform, and generate code as a first-class operation?

Two fundamentally different approaches exist:

1. **Homoiconic languages (Lisp, Clojure, Elixir)** — The primary representation of a program is also a data structure in the language itself. In Lisp, code is lists (s-expressions); in Elixir, code is AST tuples. This means metaprogramming is not a separate facility — it is simply manipulating data structures that happen to represent code. Macros are functions that receive code-as-data and return transformed code-as-data, all at compile time. There is no separate "annotation processor" or "code generator" pipeline — everything is within the language.

2. **Non-homoiconic languages (Java, Python, Rust, Go)** — Code is text. The source code's structural representation is not directly accessible as a language-level value. Metaprogramming requires either (a) a separate macro system with its own syntax and rules (Rust's procedural macros, C preprocessor), (b) annotation processing in a separate compilation phase (Java), or (c) runtime reflection that operates on compiled bytecode rather than source structure (Python, Java). Each mechanism has different rules, different syntax, and different limitations — metaprogramming is fragmented across multiple facilities.

The core problem: **metaprogramming should be as natural as programming**. If code is just data, then generating, transforming, and analysing code is the same skill as working with data structures. This eliminates the gap between "writing code" and "writing code that writes code."

For Orthon's LLM-native design, this question is amplified: **if a language is designed for LLMs, making code accessible as structured data is not optional** — it is the foundation of the Schema Provider, the Static Analyser, and the entire LLM Toolchain.

## Principles

1. **Code is data** — The language's source representation is accessible as a first-class data structure. A program can construct, inspect, and transform code at compile time using the same facilities it uses for any other data.

2. **No separate metaprogramming language** — Macros and code generation use the host language itself, not a separate macro language. The programmer does not switch between "language mode" and "macro mode."

3. **Hygienic by default** — Code transformations do not accidentally capture variables from the macro's definition context. Hygiene is the default; explicit unhygienic escape is available for advanced cases.

4. **Compile-time only** — Homoiconic operations (macro expansion, code transformation) run at compile time. No runtime code generation unless explicitly opted into (JIT compilation).

5. **Phase separation** — Compile-time code and runtime code are distinct phases. A macro runs during compilation and produces runtime code; the runtime code cannot invoke the macro.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Metaprogramming Policy | Determines whether the language is homoiconic (code-as-data macros), has a separate macro system, or uses code generation only |
| Hygiene Policy | Governs whether macro expansion is hygienic (no accidental capture) or unhygienic (explicit capture control) |
| Phase Policy | Controls the distinction between compile-time and runtime phases |
| Schema Policy | Governs whether the Abstract Syntax Tree is exposed as a data structure accessible to both humans and LLMs |

## Model (What)

A homoiconic language represents code as data structures that the language itself can manipulate.

### Full Homoiconicity (Lisp Model)

Programs are data structures — typically lists (s-expressions). A macro is a function that receives code-as-data and returns code-as-data.

```orthon
# Conceptual illustration: code as nested data
# The expression  (+ 1 (* 2 3))  is also the list  [+, 1, [*, 2, 3]]

macro unless(condition, body):
    # `condition` and `body` are received as unevaluated data structures
    # Returns: `if not condition then body`
    return `(if (not ~condition) ~body)

# Usage — transformed at compile time
unless(x > 0, log("x is not positive"))
# Expands to:  if not (x > 0) then log("x is not positive")
```

Key characteristics:
- **No parsing phase distinction** — reading code and reading data use the same parser.
- **Macro expansion is recursive** — macro output is expanded again until no macros remain.
- **Quasiquotation** — `` ` `` quotes code as data; `~` unquotes to splice in computed values.

### Schema-Driven Homoiconicity (Orthon Model)

Orthon is not homoiconic in the Lisp sense — source code is text, parsed into an AST. However, the **Schema Provider** makes the language's grammar, type system, and all APIs available as structured data. This provides a form of code-queryability that serves the same goal: programs (and LLMs) can inspect language structure without parsing source code.

```
# Orthon's approach: query the Schema Provider for code structure
# The Schema Provider exposes every construct as structured data

# A static analyser or LLM tool can query:
schema.query("function", "read_config")
# Returns structured definition: params, return type, doc, constraints

schema.query("type", "Result<T, E>")
# Returns structured type schema: type params, methods, conversions
```

The difference from Lisp-style homoiconicity:
- **Read-only** — LLM tools can query code structure, but runtime code cannot transform it.
- **Schema before instance** — The grammar and type schema are available without parsing any particular program.
- **LLM-first** — The primary consumer of the schema is the LLM Toolchain, not human-written macros.

### Comparison

| Dimension | Lisp/Clojure Homoiconicity | Orthon Schema Provider |
|---|---|---|
| **What is data?** | Source code (s-expressions) | Grammar + type schema + API contracts |
| **Consumer** | Human-written macros | LLM Toolchain + static analysers |
| **Read/write?** | Read + transform (macros generate code) | Read-only (query structure) |
| **Phase** | Compile time | Compile time (schema generation) + LLM query time |
| **Metaprogramming** | Full (macros transform arbitrary code) | Schema-driven (generate code from contracts, not by transforming source) |
| **LLM benefit** | LLMs must learn macro expansion rules | LLMs query canonical schema — no guessing |

## Default Strategy

Orthon does not adopt Lisp-style full homoiconicity. Instead, the Schema Provider exposes the language's grammar, type system, and API contracts as structured, queryable data. This provides the LLM Toolchain with the same "code is queryable data" benefit that homoiconicity provides for human-written macros — but focused on correctness (eliminating hallucination) rather than source transformation.

Code generation and static analysis use the Schema Provider, not source-level macro expansion. If macro-like functionality is needed, it is achieved through the compile-time metaprogramming mechanisms described in [`METAOBJECTS.md`](./METAOBJECTS.md).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Full Lisp-style homoiconicity | Source code is a data structure (s-expressions). Full macro system. Highest flexibility, highest learning curve. |
| Reader macros (Clojure) | Extend the reader to produce custom data structures. Limited to read-time, not compile-time. |
| Procedural macros (Rust) | Separate macro functions that tokenise/parse input and produce output. Not homoiconic — macros operate on token streams, not language data structures. |
| Template-based codegen (C++, D) | Compile-time string or AST manipulation via templates. Not data-first; type-based rather than structure-based. |
| Schema-only (Orthon) | No source-level homoiconicity. Schema Provider exposes all language constructs as structured data for LLM/static analysis consumption. Simplest implementation; most LLM-targeted. |

## Open Questions

1. Should Orthon provide a minimal macro facility (e.g., `@derive`-style code generation from schema definitions) even without full homoiconicity?
2. Can the Schema Provider be extended to make per-file ASTs queryable — effectively providing compile-time read-only access to any program's structure?
3. Does Orthon need quasiquotation for template-style code generation from the Schema Provider?
4. How does the Schema Provider interact with the Execution Program model — should the schema vary per execution strategy?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

**See also:**
- [`METAOBJECTS.md`](./METAOBJECTS.md) — macros, annotations, and compile-time metaprogramming
- [`LLM_NATIVE_TOOLCHAIN.md`](./LLM_NATIVE_TOOLCHAIN.md) — Schema Provider and LLM Toolchain design
- [`FOUNDATIONAL_ABSTRACTIONS.md`](./FOUNDATIONAL_ABSTRACTIONS.md) — Data and Data Modifiers as foundational abstractions
