# EDR-029: AST Macros as Functions

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Language

---

### Context

Phase 4 (Derived Features) must decide Orthon's metaprogramming mechanism. The research document [`AST_MACROS.md`](../../how/concepts/research/essential/AST_MACROS.md) proposes macros as functions operating on typed AST nodes, as an alternative to both Zig's unified `comptime` (where code generation is done via ordinary code running at compile time) and Rust's procedural macros (where macros use a separate token-stream API).

A concurrent decision ([`COMPILE_TIME_EXECUTION`](EDR-031-compile-time-execution.md)) adopts a unified comptime model for Orthon. AST macros must therefore be defined as building *on* comptime — macro functions are Orthon functions annotated with `@macro` that execute in the comptime phase.

The key design tension: macros that operate on typed AST nodes provide stronger guarantees (input/output types verified) but require the compiler to expose AST types to user code. A pure comptime approach with no macro mechanism is simpler but offers less structure for the common pattern of code generation (trait implementations, derive macros).

### Decision

Adopt **AST macros as functions** — macros are ordinary Orthon functions annotated with `@macro`, executing at compile time via the comptime mechanism:

1. **`@macro` annotation** — Any function can be annotated `@macro` to signal that it executes at compile time, receives typed AST nodes, and returns typed AST nodes.
2. **Typed AST nodes** — Macro input and output are typed AST types (e.g., `TypeDef`, `FnDecl`, `ImplBlock`), not raw token streams.
3. **`@derive` sugar** — The most common macro pattern (trait implementation generation) receives dedicated declarative syntax: `@derive(Show, Eq, Clone)`.
4. **Hygienic by default** — Macro-introduced identifiers are scoped to the expansion. Unhygienic access uses `#` prefix.
5. **Single-pass expansion** — No recursive macro expansion. All macros are expanded before type checking of the expanded code.
6. **Side-effect-free** — Macros cannot perform IO, access filesystem, or introduce side effects beyond code generation.
7. **Building on comptime** — Macro functions are compiled and executed via the comptime mechanism (see EDR-031).

### Consequences

**Positive:**
- No separate macro-definition language — macros are just Orthon functions.
- Typed AST nodes provide stronger safety than raw token streams (Rust proc macros).
- Single-pass expansion eliminates phase-ordering bugs.
- `@derive` provides a declarative surface for the most common pattern.
- Hygiene-by-default prevents accidental identifier collisions.
- Building on comptime means no separate execution engine for macros.

**Negative:**
- Exposing AST types to user code increases the compiler's public API surface.
- Typed AST nodes are less flexible than raw token streams for complex transformations.
- Single-pass expansion prevents macros that generate macro invocations (nested metaprogramming).

### Compliance

1. All `@macro` functions must declare typed AST node input/output signatures.
2. The compiler must verify macro output via type checking after expansion.
3. The `@derive` registry must be checked at compile time — unknown derive targets produce errors.
4. Macro hygiene must be enforced by the compiler; unhygienic access must use `#` prefix.
5. The `--expand-macros` flag must show AST after expansion but before type checking.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| **No macros — pure comptime (Zig-style)** | Code generation via comptime alone is powerful but lacks structured contracts for the common derive pattern. Macros provide explicit input/output types. |
| **Rust-style procedural macros** | Separate token-stream API (`syn`, `quote`) adds complexity. Typed AST nodes provide sufficient power with better IDE/LLM support. |
| **Template-based macros (C preprocessor)** | Unsafe — no type checking, no hygiene, text-level manipulation. Rejected outright. |
| **Lisp-style homoiconic macros** | Requires code-as-data representation. Orthon is not homoiconic. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Macros solve real metaprogramming needs; `@derive` addresses the most common pattern directly. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Hygienic scoping, single-pass expansion, and typed AST contracts are internally consistent. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Single mechanism (`@macro` function) replaces multiple sublanguages. Cannot be expressed via composition of primitives — AST manipulation requires compiler-level AST types. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Macros operate within the compiler pipeline (parsing → macro expansion → type checking). Building on comptime ensures no layering violation. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Macro semantics (hygiene, expansion order, AST types) are strategy-independent. The comptime execution engine is a compiler concern, not an Implementation Strategy concern. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Evolution path clear: new AST types can be added; `@derive` registry is extensible. Single-pass expansion avoids future phase-ordering complexity. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Macro contracts (input type, output type, hygiene rules) are explicit and schema-exposable. `@derive` is purely declarative. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-029 for per-gate reasoning trail.
