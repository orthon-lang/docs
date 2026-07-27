# EDR-054: Object Initialization — Named Parameters with Defaults and Builder Patterns

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

Object initialization in most languages suffers from two classical anti-patterns:
- **Telescoping constructors** — multiple overloaded constructors for different subsets of fields. Scales combinatorially as the number of optional parameters grows.
- **Builder pattern** — external builder class, fluent setters. Solves telescoping but requires boilerplate (separate builder class, builder methods, final `build()` call).

The research document at `how/concepts/research/important/OBJECT_INITIALIZATION.md` proposes solving this with:
- **Named arguments** — every parameter can be referenced by name
- **Default values** — constructor parameters can have default values
- **Copy-and-update syntax** — immutable modifications: `Config(cfg, port: 9090)`
- **No builder boilerplate** — compiler generates wiring
- **Compile-time completeness check** — all required fields must be provided

The question: **do these patterns add new language semantics, or are they StdLib patterns?**

Analysis:
- **Named arguments** are a language feature — the parser must support `name: value` syntax in function calls, and the type system must match named parameters to declaration parameters. Required for function calls generally, not just constructors.
- **Default values** are a language feature — the compiler must evaluate defaults for omitted parameters.
- **Copy-and-update syntax** is a language feature — the compiler must generate copy-with-modification code.
- **Builder auto-generation** is a StdLib/annotation concern — `@builder` can be implemented via AST macros (EDR-029).

However, Orthon's declaration model (`type`, `struct`, `new`) already has a constructor mechanism. Named parameters and default values are general function call features, not constructor-specific. The copy-and-update syntax (`Config(cfg, port: 9090)`) is syntactic sugar over `new` + field assignment.

The Decision Pipeline classified OBJECT_INITIALIZATION as **StdLib**: Constructor patterns and builder patterns add no new language semantics. Named parameters and default values are already covered by the general function call model (SYNTAX.md). Copy-and-update syntax is sugar over `new` + assignment. Builder auto-generation is a macro pattern.

---

### Decision

Object initialization in Orthon follows existing mechanisms:

1. **Named parameters in function calls** — Already part of the general function call model. Parameters can be referenced by name in any function call, not just constructors.

    ```orthon
    let cfg = Config(host: "example.com", port: 443, tls: true)
    ```

2. **Default values in type/function declarations** — Parameters may have default values in their declaration. Omitted named arguments use the default. Already a general feature (SEMANTIC_MODEL).

    ```orthon
    type Config(
        host: String = "localhost",
        port: Int = 8080,
        tls: Bool = false
    )
    ```

3. **Copy-and-update syntax** — A syntactic sugar for creating a new value from an existing one with specific field modifications. Desugars to `new` + field assignments.

    ```orthon
    let cfg2 = Config(cfg1, port: 443)
    # Desugars to: let cfg2 = Config(host: cfg1.host, port: 443, tls: cfg1.tls)
    ```

4. **Builder pattern via macro** — `@builder` annotation on a type auto-generates a builder API. Implemented via AST macros (EDR-029), not language-level.

    ```orthon
    @builder
    type Config(host: String, port: Int = 8080)
    # Generates: Config.builder().host("...").port(443).build()
    ```

5. **Required fields** — Parameters without default values are required. The compiler enforces that all required fields are provided (compile-time error if missing).

**Key principle:** Object initialization uses Orthon's existing mechanisms — no new language constructs. Named parameters, default values, and copy-and-update are all general features, not constructor-specific.

---

### Consequences

**Positive:**
- Zero new language semantics — named parameters are already part of the function call model.
- Copy-and-update syntax is a natural extension of named parameters (partial application of fields).
- Builder patterns are available via `@builder` macro — no boilerplate, no language changes.
- Compile-time completeness checking for required fields is already part of the parameter model.
- Default values are already part of the type declaration model.

**Negative:**
- Copy-and-update syntax must be formally specified in SYNTAX.md — currently implicit.
- `@builder` macro requires AST macros (EDR-029) to be available — builders deferred until the macro mechanism is stable.
- Positional-only constructors (for ergonomics with few parameters) must be documented separately.

---

### Compliance

1. Object initialization uses the existing function call model — `Type(field1: value1, field2: value2)`.
2. Default values are evaluated eagerly at construction time (per SEMANTIC_MODEL).
3. Copy-and-update syntax `Type(existing, field: new_value)` desugars to field-by-field copy with replacements.
4. `@builder` macro is an AST macro (EDR-029) — not a language construct.
5. Compile-time completeness checking for required fields is part of the general parameter model.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Built-in builder (compiler generates builder class) | Unnecessary — AST macros (EDR-029) provide the same capability without language changes. |
| Positional-only constructors (C-style) | Would prevent named parameter usage and create ambiguity with multiple parameters of the same type. |
| Separate `init`/`constructor` syntax | Orthon's `new` declaration kind already serves this role. A separate constructor syntax would duplicate functionality. |
| Lazy default values | Default values evaluated eagerly are simpler and more predictable. Lazy evaluation is an optimisation concern. |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (Standard Library addition — `@builder` macro pattern): `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE`.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to create objects with many optional fields without writing a separate builder class." Named parameters with defaults solve this directly. `@builder` provides fluent builder style when desired. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | All features use existing language mechanisms. Named parameters are already part of the function call model. Copy-and-update is sugar over existing primitives. Builder is a macro. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "Create objects with named parameters; omit optional ones." The model matches Kotlin, Swift, and TypeScript — proven patterns. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | Named parameters have consistent semantics with positional ones. Default values follow the same rules as any parameter default. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | All patterns use existing Core Language mechanisms. No layer violations. |

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — StdLib addition, implementation concerns are covered by existing mechanisms. `LLM_GENERABILITY_GATE` — Named parameters are standard; LLMs generate them reliably.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
