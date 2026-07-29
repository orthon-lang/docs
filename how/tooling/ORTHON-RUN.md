# Tooling: orthon run <function> — Every Function as CLI

> **Status:** Open · **Priority:** P2 · **Target:** M3 (Tooling)
> **Source:** [BAML gap analysis](../../notes/baml-concepts-orthon-gap-analysis.md) § Tier 2 (#9)

---

## Description

A CLI tool that invokes any exported function directly from the
command line, without requiring a `main()` wrapper or test harness.
Every function in an Orthon module is reachable as:

```sh
$ orthon run my_module.my_function arg1 arg2
```

Analogous to `baml run <function>` in the BAML ecosystem.

## Problem

LLM agents operate in write-run-observe loops. For each iteration,
they must either:
- Write a temporary `main()` wrapper around the function they want
  to test, OR
- Navigate a test harness

Both add friction. Direct function invocation eliminates the wrapper
step, shortening the iteration cycle.

## Spec Impact

### If this tool exists

- Agents can invoke any function in 1 command — no boilerplate.
- Write-run-observe loops become: write → `orthon run` → observe → fix.
- The tool is the foundation for `orthon run -e` (inline execution).

### If this tool does NOT exist

- Every agent workflow requires a `main()` harness or test runner.
- Iteration cycles are 2-3x longer for isolated function testing.
- Orthon lags behind BAML in LLM agent UX.

## Language Feature Dependencies

- **Function model** — Functions must be visible at the module level
  with a deterministic entry-point convention (exported functions).
- **Module system** — The module system must support loading and
  invoking a specific function by qualified name at runtime.
- **Execution Program model** — The Execution Program concept
  (`how/concepts/research/EXECUTION_PROGRAM.md`) directly enables
  this: a function call is an Execution Program with arguments.
- **Ecosystem boundary** — Classified as **Tooling** (M3), not language core.

## Spec Documents Affected

- [`what/EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) — The
  execution model should define how a named function can be invoked
  as a standalone entry point.
- [`how/concepts/research/EXECUTION_PROGRAM.md`](../concepts/research/EXECUTION_PROGRAM.md) —
  If the Execution Program concept formalises the function-as-entry-point
  pattern, `orthon run` becomes a direct consumer.

## Notes

- `orthon run -e` (inline expression execution, item #10 in the BAML
  gap analysis) is a natural extension — see `ORTHON-RUN-E.md`.
- The tool implicitly requires the compiler/interpreter to support
  module loading without a project build step.
- Consider `orthon run --watch` for auto-recompile on source change.
