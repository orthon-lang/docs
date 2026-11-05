# Design Influences

> **Canonical block.** This document records the external languages and
> systems that shaped Orthon's design. See [`VISION.md`](VISION.md) § Learn
> from What Came Before for how this connects to the project's goals.

Orthon acknowledges its intellectual debt to Python and Java — and does
not repeat their mistakes.

## Python

From Python, Orthon inherits the value of readability and approachable
syntax, but rejects significant whitespace, scope ambiguity, unchecked
runtime type errors, and the concurrency model that surrendered
parallelism for an illusion of simplicity.

## Java

From Java, Orthon inherits the value of explicit semantics, static type
safety, and disciplined architecture, but rejects ceremonial verbosity,
the everything-is-an-object dogma, checked exception fatigue, and a
standard library that resisted modern paradigms for decades.

## Predecessors as Foundation

Orthon studies its predecessors not to imitate them, but to stand on
their shoulders — keeping what worked, fixing what didn't, and designing
for the lessons only hindsight provides.

### Broader Evolutionary Context

The following tables and analysis trace the full generational chain from
Assembler through Zig — showing what problem each language solved, how
priorities shifted across eras, and where Orthon may fit.

---

#### Generational Table: What Each Language Solved

| Language | Built on | Previous generation's limits | What the new language solved |
|----------|----------|------------------------------|------------------------------|
| **Assembler** | Machine code | Virtually impossible to write large programs; full architecture dependence | Symbolic instruction representation; readability |
| **C (1972)** | Assembler | Too low-level; no portability; OS development too hard | **Efficient hardware access** while maintaining cross-platform portability |
| **C++ (1985)** | C | No encapsulation; no code reuse; large projects too complex | **Abstractions without performance loss**; C compatibility |
| **Java (1995)** | C++ | Manual memory management; pointer errors; platform dependence | **Portability ("Write Once, Run Anywhere")**; automatic memory management; safety |
| **C# (2000)** | Java, C++ | Limited Windows integration; lacked modern language features | Modern OO language for .NET platform with high productivity |
| **Python (1991)** | C, Perl, Shell | High barrier to entry; too much boilerplate | **Maximum development speed** and readability |
| **JavaScript (1995)** | HTML + CGI | Pages were static; every action required server round-trip | **Browser interactivity** |
| **Go (2009)** | C++, Java | Slow compilation; language complexity; heavy concurrency | **Simplicity, fast compilation, scalable teams and services** |
| **Rust (2015)** | C, C++ | Memory bugs (use-after-free, data races, buffer overflows) | **Memory safety without a garbage collector** |
| **Zig (2016)** | C, C++ | Complex build system; preprocessor; hidden compiler magic | **Maximum low-level control**; simple language and tooling model |

---

#### Evolution as Priority Shifts

| Generation | Dominant problem | Typical languages |
|------------|-----------------|-------------------|
| 1. Hardware control | How to program a computer at all | Assembler |
| 2. Systems programming | Performance and portability | C |
| 3. Large projects | Managing complexity | C++, Ada |
| 4. Safety and portability | Memory; virtual machine | Java, C# |
| 5. Productivity | Write programs quickly | Python, Ruby |
| 6. Scalable services | Simplicity for large distributed systems | Go |
| 7. Safety without loss | Eliminate an entire class of memory errors | Rust |
| 8. Minimalism and full control | Replace C without historical baggage | Zig |

---

#### The General Trend

Each new popular language does not aim to be "faster" than the previous
one. It aims to remove the dominant pain of its era:

- **C** → How to write OSes and drivers efficiently
- **C++** → How to build large programs without abandoning C's speed
- **Java** → How to write one codebase for multiple platforms without crashing on pointers
- **Python** → How to write programs faster than C/C++ can compile
- **Go** → How a small team can support thousands of servers and microservices
- **Rust** → How to keep C++ speed while eliminating memory errors
- **Zig** → How to regain full low-level control without C/C++'s accumulated complexity

The evolutionary chain:

> **Assembler → C → C++ → Java/C# → Go → Rust → Zig**

Each step answers the dominant engineering challenge of its time: first
efficiency, then complexity management, then portability, then
productivity, then scaling, then memory safety, and finally simplifying
low-level development without historical compromises.

---

#### Where Orthon Sits in the Chain

Orthon's design must answer: *what is the dominant pain of this era that
a new language should solve?* The answer defines Orthon's position in the
evolutionary chain and shapes its core identity. This question — and the
historical pattern above — informs every design decision recorded in
this document.

The generational chain above traces *which* pain each language removed;
the axes below trace *which decision each language was forced to make*.
Where Orthon stands on each axis — and what it pays for each position —
is overlaid in [`what/DIALECTICAL_TRADEOFFS.md`](../what/DIALECTICAL_TRADEOFFS.md).

---

## The Axes of Language Design

An **axis** is a decision no executable language can avoid, where the
available positions are mutually exclusive and each costs something. The
axes are derived from three sources — the generational chain above, the
donor language set, and the classical problems catalogued in
[`what/LANGUAGE_PROBLEMS.md`](../what/LANGUAGE_PROBLEMS.md) — and are
stated here without Orthon's positions. Orthon's position, the accepted
trade-off, and the tension each position creates live in the overlay
ledger, per [`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md).

### 1. Data and Value Model

- **Identity vs value** — reference identity (Java), value semantics (C, Rust `Copy`).
- **Equality** — referential (Java) vs structural (Clojure, Haskell) vs mixed.
- **Mutability** — mutable by default (C, Java, Python) vs immutable by default (Clojure, Haskell) vs opt-in mutation (Rust, Swift).
- **Aliasing and ownership** — shared mutable aliasing (C, Java) vs ownership and borrowing (Rust) vs value copies (C++).

### 2. Memory and Lifetime

- **Allocation** — manual (C), tracing GC (Java, Go, Python), ARC (Swift, Objective-C), arena/region (Zig, Rust).
- **Deallocation** — manual `free` (C), tracing GC (Java), deterministic ARC (Swift), RAII/drop (C++, Rust).
- **Deterministic destruction** — RAII (C++, Rust), `defer` (Go, Zig), `using` (C#), finalizers (Java).
- **Resource management** — scoped (RAII) vs explicit close (C) vs GC.

### 3. Type System

- **Checking regime** — static (Java, Rust), dynamic (Python), gradual (TypeScript).
- **Inference** — full Hindley-Milner (Haskell, OCaml), local (Java, C#), bidirectional local (Rust).
- **Nominal vs structural** — nominal (Java), structural (TypeScript, Go interfaces), both (OCaml).
- **Subtyping and variance** — nominal inheritance (Java), structural (TypeScript), none (Rust).
- **Dependent and refinement types** — dependent (Idris 2, Lean 4, Dafny), refinement (Vera, Liquid Haskell).
- **Totality** — total (Coq, Agda, Dafny) vs partial (most languages).

### 4. Abstraction and Composition

- **Generics** — templates / monomorphization (C++, Rust), erasure (Java), type classes (Haskell).
- **Behavior composition** — class inheritance (Java, C++), traits (Rust), type classes (Haskell), protocols (Swift), none (Go, C).
- **Data composition** — algebraic data types (Haskell, Rust, OCaml), records (C `struct`), classes (Java), composition-only (Flint).
- **The Expression Problem** — two-axis extensibility; see [`what/LANGUAGE_PROBLEMS.md`](../what/LANGUAGE_PROBLEMS.md).

### 5. Functions and Control

- **First-class functions** — closures (Scheme, Rust), function pointers only (C).
- **Invocation uniformity** — method-centric (Java), extension functions (Kotlin), uniform call syntax (D).
- **Control flow** — exceptions (Java, Python), panics (Go, Rust), conditions (Lisp), continuations (Scheme).
- **Generators and coroutines** — `yield` (Python), `emit`/lazy sequences, `async`/`await`.

### 6. Concurrency and Parallelism

- **Task model** — OS threads (Java), goroutines (Go), async/await (Rust, Python, JS), actors (Erlang, Elixir, Swift), virtual threads (Java).
- **Scheduling** — runtime-scheduled (Go), runtime-neutral (Rust), event loop (JS).
- **Memory model** — sequentially consistent (Go, Java) vs relaxed (C, C++, Rust).
- **Cancellation and lifecycle** — structured concurrency (Kotlin, Swift), explicit (Rust), context-passing (Go).
- **Heterogeneous execution** — GPU/SIMD (Mojo, Flint vectors, CUDA), implicit parallelism (Bend).

### 7. Effects and Purity

- **Purity** — pure by default (Haskell) vs effectful by default (most).
- **Effect tracking** — monads (Haskell), algebraic effects (Koka, Vera, Idris 2), capabilities (Unison), none (most).
- **Side-effect visibility** — explicit (Vera `effects`), implicit (most).

### 8. Errors and Absence

- **Error representation** — return codes (C), checked exceptions (Java), `Result` (Rust, Haskell), error unions (Zig), exceptions (Python).
- **Nullability** — `null` (C, Java) vs `Option` (Rust, Swift) vs no null (Haskell).
- **Unions and variants** — tagged (Rust `enum`), untagged (TypeScript), polymorphic variants (OCaml).
- **Contracts** — pre/postconditions (Dafny, SPARK, Vera), Design by Contract (Eiffel), none (most).

### 9. Modules and Compilation

- **Module system** — header files (C), packages (Java), crates (Rust), functors (OCaml).
- **Compilation unit** — whole-program vs separate compilation vs incremental (Swift, Rust).
- **Build and packaging** — external (C `make`, CMake), integrated (Go, Zig, Cargo).

### 10. Metaprogramming

- **Macros** — C preprocessor, Lisp macros, Rust macros, Nim, D.
- **Compile-time execution** — Zig `comptime`, C++ `constexpr`, D.
- **Reflection and introspection** — runtime reflection (Java, C#), compile-time (Zig, Rust), none (C).

### 11. Syntax and Surface

- **Parsing philosophy** — significant whitespace (Python) vs braces (C-family).
- **Expression orientation** — everything-is-an-expression (Rust, OCaml, Elixir) vs statement-oriented (C, Java).
- **Named vs symbolic** — named operators with symbolic sugar.
- **Canonical forms** — one spelling per construct (Vera) vs many (Perl, C++).

### 12. Execution and Deployment Model

- **Compilation target** — native (C, Rust), VM (Java), WASM (Vera), source (Python).
- **AOT vs JIT** — AOT (Rust), JIT (Java, JS), both (Dart, C#).
- **Artifact form** — source (Python), binary (C), complete program (Vera WASM).

### 13. Evolution and Compatibility

- **Versioning** — SemVer (Rust), date-based, none (C).
- **Deprecation** — never-break (Java), epoch breaking (Python 3), stability guarantees (Go).
- **Core vs library evolution** — everything in core (C) vs minimal core plus libraries.

### 14. Tooling and LLM-Facing Surface

- **Toolchain** — separate (C), integrated (Go, Rust, Zig).
- **Checkability vs expressiveness** — constrain to be checkable (Vera, Dafny) vs maximise expressiveness.
- **LLM generability** — designed for LLM writers (Vera) vs human-first legacy.

---

## See Also

- [`VISION.md`](VISION.md) — the core vision this analysis supports
- [`what/GLOSSARY.md`](../what/GLOSSARY.md) — unified terminology reference
- [`what/DIALECTICAL_TRADEOFFS.md`](../what/DIALECTICAL_TRADEOFFS.md) — Orthon's position on each axis, the accepted trade-off, and the new contradiction
- [`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md) — the method that reconstructs and overlays the axes
