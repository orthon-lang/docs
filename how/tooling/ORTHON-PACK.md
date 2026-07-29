# Tooling: orthon pack — Function as Standalone Binary

> **Status:** Open · **Priority:** P3 · **Target:** M3–M4 (Tooling → Implementation)
> **Source:** [BAML gap analysis](../../notes/baml-concepts-orthon-gap-analysis.md) § Tier 2 (#11)

---

## Description

A packaging tool that ships a selected function (and its transitive
dependencies) as a standalone binary — small, self-contained, and
deployable. LLM-generated utilities become production artifacts with
one command.

Analogous to `baml pack` in the BAML ecosystem.

## Problem

LLM agents can generate useful utility functions, but turning them
into deployable artifacts requires build configuration, dependency
management, and packaging. A `pack` command automates this: "write
a function → `orthon pack` → deploy binary."

## Spec Impact

### If this tool exists

- LLM-generated functions become deployable in one step.
- Enables the "write → pack → deploy" agent workflow.
- Lowers the barrier from agent-generated code to production artifact.

### If this tool does NOT exist

- Deploying an Orthon function requires manual build and packaging.
- The "one-command to production" workflow is unavailable.
- Orthon lags behind BAML in deployability.

## Language Feature Dependencies

- **Execution Program model** — `pack` is a natural expression of the
  Execution Program concept: it produces a self-contained execution
  image from a function definition.
- **Module system with deterministic dependency resolution** —
  Transitive dependency closure must be computable.
- **Implementation Strategies** — Packaging decisions (static linking,
  target platform, size vs performance) map to Implementation Strategy
  profiles.
- **Ecosystem boundary** — Classified as **Tooling** (M3) with
  Implementation (M4) dependencies.

## Spec Documents Affected

- [`how/concepts/research/EXECUTION_PROGRAM.md`](../concepts/research/EXECUTION_PROGRAM.md) —
  The Execution Program concept should define what "packaging" means
  in semantic terms.
- [`how/strategies/DEFAULT_STRATEGY.md`](../strategies/DEFAULT_STRATEGY.md) —
  Default packaging profile (balanced size/performance).
- [`how/strategies/EMBEDDED_STRATEGY.md`](../strategies/EMBEDDED_STRATEGY.md) —
  Embedded strategy implies minimal binary size packaging.

## Notes

- Binary size is a competitive factor — BAML ships 12 MB binaries.
  Orthon should aim for comparable or better.
- Static linking vs. dynamic runtime trade-off needs investigation
  during M3.
- Could support `--target wasm`, `--target container` as format options.
