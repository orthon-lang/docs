# Tooling: orthon eval — Typed Compile-Time Expression Evaluation

> **Status:** Open · **Priority:** P3 · **Target:** M3 (Tooling)
> **Source:** [BAML gap analysis](../../notes/baml-concepts-orthon-gap-analysis.md) § Tier 2 (#12)

---

## Description

A typed `eval` mechanism that compiles a source expression against
an expected type signature and returns typed compiler errors.
Designed for LLM agents that generate Orthon source dynamically:

```orthon
let callback = package.build().get<() -> string>("hello");
```

If the generated source does not match the type signature, the
compiler returns diagnostics that feed back to the agent.

Analogous to BAML's planned typed `eval` feature.

## Problem

When an LLM agent generates Orthon source dynamically, it needs
a way to validate that the generated code is type-correct before
deployment. A typed eval provides this validation with structured
error feedback that the agent can use to correct itself.

## Spec Impact

### If this tool exists

- Agents can generate Orthon code and validate it against type
  signatures in a single step.
- Compiler errors become machine-readable feedback for agent
  self-correction loops.
- Enables safe dynamic code generation patterns.

### If this tool does NOT exist

- Dynamic code generation requires a separate compile step with
  unstructured error parsing.
- Agent self-correction loops are slower and less reliable.
- Orthon lags behind BAML in LLM-native code generation support.

## Language Feature Dependencies

- **Type System** — The type system must support querying a specific
  type signature (`get<T>()` pattern). See `how/architecture/TYPE_SYSTEM.md`.
- **Compile-time execution** — The evaluation happens at compile time,
  not runtime. See `COMPILE_TIME_EXECUTION.md` concept research.
- **AST_MACROS / metaprogramming** — The mechanism may overlap with
  compile-time metaprogramming facilities.
- **Ecosystem boundary** — Classified as **Tooling** (M3) that depends
  on language core (Type System, Compile-Time Execution).

## Spec Documents Affected

- [`how/architecture/TYPE_SYSTEM.md`](../architecture/TYPE_SYSTEM.md) —
  Must define how type signatures are queryable at compile time.
- [`how/architecture/IR.md`](../architecture/IR.md) — The typed eval
  pipeline: source → parse → type-check → produce diagnostics.
- [`what/OPTIMIZATION_MODEL.md`](../../what/OPTIMIZATION_MODEL.md) —
  Typed eval is a compile-time operation, not a runtime optimisation.
  Its classification in the optimisation model should be documented.

## Notes

- BAML's approach is planned (not yet shipped). Monitor their BEPs
  for design evolution.
- Could be implemented as a library function (`std.eval`) rather than
  a separate CLI tool, if the language supports compile-time execution.
- Important for the LLM Toolchain (M2, item 8.3) — the Code Generator
  and Static Analyser components would use this.
