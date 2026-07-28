# EDR-031: Unified Compile-Time Execution (Comptime)

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Phase 4 must decide Orthon's model for generics, reflection, and metaprogramming. The research document [`COMPILE_TIME_EXECUTION.md`](../../how/concepts/research/essential/COMPILE_TIME_EXECUTION.md) evaluates whether a single unified compile-time execution mechanism (Zig-style `comptime`) should replace or coexist with the four separate mechanisms proposed in earlier research (declared generics, structural typing, metaobjects, reflection alternatives).

Several concurrent decisions constrain this choice:

- **GENERICS (Plan 04-02):** [`GENERICS.md`](../../how/concepts/research/essential/GENERICS.md) proposes declared trait-bound generics — a Rust-style model where generic parameters are constrained by explicit trait bounds checked before instantiation.
- **AST_MACROS (EDR-029):** Adopts AST macros as functions, building on comptime.
- **LLM GENERABILITY GATE (EDR-014):** Requires that language constructs be LLM-generable — a fully duck-typed comptime model (Zig-style) would make generic contracts implicit, reducing LLM tooling effectiveness.

The core tension: unified comptime is elegant (one mechanism replaces four), but Zig's duck-typed approach to generic constraints sacrifices discoverability — an LLM (or human) cannot determine what a generic function requires without reading the function body. Orthon needs the elegance of comptime with the discoverability of declared contracts.

### Decision

Adopt a **unified comptime model** with **explicit trait bounds for discoverability**:

1. **`comptime` keyword** — Any function parameter can be declared `comptime`, resolved at compile time. `comptime T: type` replaces separate `<T>` generic syntax.
2. **Explicit trait bounds** — Comptime parameters MAY include trait bounds (`comptime T: type + Comparable`) for discoverability. Public API generics MUST include bounds; private/internal MAY omit them.
3. **Same semantics, earlier phase** — Comptime code uses the same language semantics as runtime code, executed during compilation. No separate sublanguages.
4. **Reflection via comptime** — `@typeInfo(T)`, `@field(value, name)`, `@hasDecl(T, name)` are comptime-evaluated function calls, not a separate reflection API.
5. **Metaprogramming via comptime** — Code that generates code is ordinary `if`/`for`/`while` control flow running at comptime.
6. **Visible marking** — Comptime code is visibly marked (`comptime` keyword at definition site, `@` prefix for reflection operations).
7. **Deterministic, sandboxed** — Comptime evaluation is deterministic. No IO, filesystem access, or network operations.

### Relationship to GENERICS

Comptime IS the generic mechanism in Orthon. `comptime T: type` replaces separate `<T>` generic syntax. Trait bounds on comptime parameters provide the declared-contract discoverability that Rust-style generics offer. See [`GENERICS.md`](../../what/concepts/GENERICS.md) for the complete interaction model.

### Relationship to AST_MACROS

AST macros (EDR-029) execute in the comptime phase. Macro functions are compiled and evaluated via the comptime mechanism. Comptime provides the execution engine; macros provide the structured AST-layer code generation pattern. They are complementary, not alternative.

### Consequences

**Positive:**
- One mechanism (comptime) replaces generics, reflection, and metaprogramming — no separate sublanguages.
- Explicit trait bounds on public comptime parameters preserve discoverability for LLM and IDE tooling.
- Same semantics, earlier phase reduces learning curve — comptime code looks like ordinary Orthon.
- Reflection is just comptime function calls — no separate reflection API to design and maintain.
- Deterministic, sandboxed comptime prevents compilation-time side effects.

**Negative:**
- Comptime interpreter adds compiler implementation complexity.
- Hybrid model (bounds required for public, optional for private) creates two regimes with different rules.
- Duck-typed private comptime parameters may produce less readable error messages than declared bounds.
- LLM generability is still more challenging than fully declared generics — private duck-typed parameters are invisible to tooling.

### Compliance

1. All comptime parameters must use the `comptime` keyword — no implicit comptime.
2. Public API generic functions MUST include trait bounds on all comptime parameters.
3. Private/internal generic functions MAY omit trait bounds (duck-typed).
4. The Schema Provider MUST expose comptime parameter bounds for LLM tooling.
5. Comptime evaluation must be deterministic — identical inputs produce identical outputs.
6. Comptime code must be sandboxed from IO and external side effects.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| **Full duck-typed comptime (Zig)** | Maximum flexibility. Rejected because implicit contracts harm LLM discoverability — an LLM cannot determine generic requirements from the signature alone. |
| **Separate generic syntax (Rust)** | Declared bounds by default. Rejected because it adds a second mechanism (generics) alongside comptime, increasing language surface. Hybrid comptime+bounds gives the best of both. |
| **Runtime reflection only** | No compile-time execution. Rejected because runtime reflection is less safe, slower, and incompatible with Orthon's "no runtime overhead" principle for metaprogramming. |
| **No comptime — separate macro + generics + reflection** | Four mechanisms for four concerns. Rejected as violating Minimal Core and One Concept — One Syntax principles. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmers get generics, reflection, and metaprogramming from one familiar-looking mechanism. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | "Same semantics, earlier phase" is internally consistent. Hybrid bounds regime (public required, private optional) has clear rules. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Flag | The hybrid bounds regime (public MUST have bounds, private MAY omit) adds mild complexity over a pure duck-typed or pure declared model. Acceptable given the LLM discoverability benefit. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Comptime operates within the Core Language layer. The `comptime` keyword is orthogonal to existing constructs. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Comptime semantics (deterministic, sandboxed, same language) are strategy-independent. Implementation Strategy only affects *how* comptime is executed (interpreter, compiled-and-run, JIT). |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Evolution path clear: new comptime operations can be added. Private duck-typed regime can be tightened in future if needed. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | **Critical** — See notes. | Public API bounds requirement ensures LLM can determine contracts from signatures. Private duck-typed parameters are a known risk — Schema Provider MUST expose bounds info. Restricted to visible comptime constructs. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-031 for per-gate reasoning trail.
