# Tooling: orthon describe — AST-Aware Symbol Discovery

> **Status:** Open · **Priority:** P2 · **Target:** M3 (Tooling)
> **Source:** [BAML gap analysis](../../notes/baml-concepts-orthon-gap-analysis.md) § Tier 2 (#8)

---

## Description

An AST-aware CLI tool that lets agents query a symbol — function, type,
variable — and receive its definition, dependencies, and all call sites
in a single structured response, without reading files into context.

Analogous to `baml describe` in the BAML ecosystem.

## Problem

LLM agents currently navigate codebases by reading files into context.
This is slow (token-costly), error-prone (wrong imports, missed
references), and scales poorly with project size. A dedicated
discovery tool is more efficient than LSP for agent workflows because
it returns all relevant information in one call, with no round-trips.

## Spec Impact

### If this tool exists

- Agents can discover symbol structure in ~1 round trip instead of
  N file reads.
- No side-quests into import resolution — the tool resolves everything.
- The tool becomes the primary agent-codebase interface, reducing
  context window pressure.

### If this tool does NOT exist

- Agents must read source files directly and resolve references
  manually — slower, less reliable, more token-expensive.
- Larger projects become impractical for agent navigation.
- Orthon lags behind BAML in LLM agent UX — a stated project goal
  (see `why/VISION.md` § LLM Readiness).

## Language Feature Dependencies

- **IR specification** — The tool needs a queryable IR/AST format.
  The IR must expose symbol definitions, references, and dependency
  edges. This should influence `how/architecture/IR.md` design.
- **Name Resolution** — The tool depends on the name resolution model
  defined in `how/architecture/NAME_RESOLUTION.md`. Cross-module
  symbol resolution must be deterministic and queryable.
- **Module system** — Works best with a deterministic module graph
  (see `CONTEXT_LIMITED_MODULES` / EDR-062).
- **Ecosystem boundary** — Must be classified as **Tooling** (M3),
  not language core.

## Spec Documents Affected

- [`how/architecture/IR.md`](../architecture/IR.md) — IR should expose
  queryable symbol references and dependency edges, not just linear
  instruction sequences.
- [`how/architecture/NAME_RESOLUTION.md`](../architecture/NAME_RESOLUTION.md) —
  Name resolution must produce a queryable resolution graph.
- [`what/EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) — no direct
  impact, but note that `describe` operates on static analysis, not runtime.

## Notes

- Potential precursor to a full LSP implementation.
- If the IR is designed as a graph from the start, `describe` becomes
  a graph query rather than a custom tool.
- Could be extended later to support "what changed between versions"
  queries.
- Contracts (contract unification review, 2026-10-21): the response for a symbol may include its contracts (the named contracts it obeys and its inline conditions), the rule names (predicate names) and, for a contract, the functions and types that obey it. A rule catalog is an output mode of the same query. The description of a rule comes from the predicate name or its comment; whether comments are retained in the AST is open (see Open Question 16 of [`TRAIT_CONTRACT_UNIFICATION.md`](../concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md)). The IR must keep contracts and conformance as named, queryable entities; [`ORTHON-TEST.md`](ORTHON-TEST.md) depends on it too.
