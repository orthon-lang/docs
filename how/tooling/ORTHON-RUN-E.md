# Tooling: orthon run -e — Inline Expression Execution

> **Status:** Open · **Priority:** P3 · **Target:** M3 (Tooling)
> **Source:** [BAML gap analysis](../../notes/baml-concepts-orthon-gap-analysis.md) § Tier 2 (#10)

---

## Description

Inline expression execution — run an Orthon expression directly from
the CLI without creating a file:

```sh
$ orthon run -e '"a,b,c".split(",")'
["a", "b", "c"]
```

Analogous to `baml run -e` in the BAML ecosystem.

## Problem

LLM agents often need to experiment with small code snippets —
checking syntax, testing library behaviour, or validating type
inference. Creating a file for each experiment adds friction.
Inline execution eliminates the file-creation step entirely.

## Spec Impact

### If this tool exists

- Agents can test any expression instantly — no file I/O.
- Reduces the cognitive overhead of "do I need a file for this?"
- Enables rapid prototyping of algorithms and data transformations.

### If this tool does NOT exist

- Every experiment requires a temporary file and cleanup.
- Break-fix cycles for small expressions are slower.
- Orthon lags behind languages with REPL-like CLI tools.

## Language Feature Dependencies

- **Expression-oriented language** — Orthon is expression-oriented
  (see `EXPRESSION_ORIENTED_LANGUAGE` in concept research), so
  any expression is a valid program. This makes `-e` natural.
- **Execution Program model** — An inline expression is trivially
  an Execution Program with no dependencies.
- **Ecosystem boundary** — Classified as **Tooling** (M3), not language core.

## Spec Documents Affected

- [`what/EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) — The
  execution model should confirm that any expression can be evaluated
  as a standalone program.
- [`how/concepts/research/EXPRESSION_ORIENTED_LANGUAGE.md`](../concepts/research/EXPRESSION_ORIENTED_LANGUAGE.md) —
  Expression-orientation is a prerequisite for `-e` to be natural.

## Notes

- Extension of `orthon run` (see `ORTHON-RUN.md`).
- Could evolve into a full REPL mode (`orthon repl`).
- Type information in output (like `baml run -e --json`) would be
  valuable for agent consumption.
