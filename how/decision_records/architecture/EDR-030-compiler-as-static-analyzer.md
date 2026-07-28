# EDR-030: Compiler as Static Analyzer

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Every modern language draws a line between what the compiler checks and what is left to external tools, runtime checks, or programmer discipline. The research document [`COMPILER_AS_STATIC_ANALYZER.md`](../../how/concepts/research/essential/COMPILER_AS_STATIC_ANALYZER.md) explores the trade-offs.

For Orthon, the question is structural: should static analysis be built into the compiler pipeline (where it is always available, always runs, and produces consistent diagnostics) or externalised to linters (where it is optional, pluggable, and may not run in every workflow)?

Orthon's LLM-native design amplifies the case for built-in analysis: an LLM generating code benefits from the fastest possible feedback loop (generate → compile → fix, no runtime needed). External linters would add an extra tooling step that an LLM-driven workflow might skip.

At the same time, the compiler must remain implementable — a fully verified compiler (dependent types, theorem proving) is a multi-decade research project. The model must be layered: cheap checks first (syntax, types), deeper checks second (ownership, effects), optional checks third (contracts).

### Decision

Adopt the **compiler as static analyzer** model with progressive verification layers:

1. **Seven verification layers** built into the compiler pipeline:
   - Layer 1: Syntax & Parsing
   - Layer 2: Name Resolution
   - Layer 3: Type Checking
   - Layer 4: Ownership & Borrowing
   - Layer 5: Effect Verification
   - Layer 6: Exhaustiveness & Completeness
   - Layer 7: Contract Verification (optional)
2. **Guaranteed analyses** (Layers 1–6): Always enabled, always checked. No opt-out for release builds.
3. **Extension analyses** (Layer 7 + linting): Available as opt-in extensions. External tools may add project-specific rules but cannot relax compiler checks.
4. **No undefined behaviour** — The compiler rejects all programs where behaviour is not fully specified.
5. **LLM-friendly diagnostics** — Every diagnostic includes machine-readable error code, location, and repair hint.
6. **`--relaxed` mode** — Skips layers 6–7 for prototyping but produces a non-release artifact.

### Consequences

**Positive:**
- All compilation runs through the same verification pipeline — no separate linter invocation needed.
- LLM feedback loop is maximally fast: compile output = verification output.
- Progressive layers allow fail-fast for cheap checks and deep verification for complex properties.
- Layers 1–6 produce a well-defined "passes compiler" baseline that every Implementation Strategy must support.
- Layer 7 (contract verification) is optional, preventing annotation burden on non-critical code.
- LLM-friendly diagnostics enable automated repair loops.

**Negative:**
- Compiler implementation is more complex than a minimal parser + type checker.
- Some deep verification (Layers 6–7) may slow compilation for large codebases.
- Built-in analysis means the compiler owns the diagnostic format — external tools cannot easily replace it.

### Compliance

1. Every `wvy build` or `wvy check` invocation must run Layers 1–6.
2. Any program that passes Layers 1–6 must be semantically well-defined (no undefined behaviour).
3. Diagnostic output must include structured LLM-readable format by default.
4. `--relaxed` mode must be explicitly documented as producing non-release artifacts.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| **Minimal compiler + external linters (C/Go model)** | Faster compilation but delegates most verification to optional tools. LLM workflow would need to run separate tools. |
| **Full dependent types (ATS/Coq model)** | Maximum verification but extreme annotation burden. Impractical for general-purpose language. |
| **Sound type system only (Haskell/OCaml model)** | Type safety without ownership/effect tracking. Insufficient for systems programming. |
| **Pluggable linter ecosystem (Rust clippy model)** | Rich ecosystem but requires separate toolchain awareness. Orthon's LLM-native design favours unified feedback. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmers get fast, unified feedback. LLM gets single-channel diagnostics. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Layers are ordered by dependency; each layer builds on the previous without contradiction. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Progressive layers are a well-known pattern (Rust, Swift, Haskell). One mechanism (compiler pipeline) replaces compiler + linter + CI checks. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Verification layers are part of the Core Language — every Implementation Strategy must support them. No layering violation. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | The semantic specification of each layer is strategy-independent; only the implementation (check algorithm) differs per strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Layers can be extended independently. New checks can be added at existing layers or as new layers without breaking existing code. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The compiler IS the analyzer — LLM sees all diagnostics from one invocation. Structured error format eliminates parsing ambiguity. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-030 for per-gate reasoning trail.
