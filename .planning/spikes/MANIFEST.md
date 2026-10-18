# Spike Manifest

## Ideas

### semantic-simplicity-metrics
Measure the simplicity of the Orthon specification semantically (on a typed graph of kernel elements, ideas, forms, residue and dependencies) instead of by word counts, so that a simplicity budget, the one-idea-one-construct audit and the grade assignment of Phase 04.5 rest on computable data. Decisions: `.planning/notes/2026-10-18-graded-spec-growth.md` (Decisions 10-11).

**Requirements:**

- The unit of measurement is the idea, not the concept file.
- Metrics are a vector, never one composite index.
- `depends_on` is curated with edge strength (`normative` / `see_also`); it is not mined from links.
- Documentation-only repository: spikes produce findings and a schema, no tooling.

## Spikes

| # | Idea | Name | Type | Validates | Verdict | Tags |
|---|------|------|------|-----------|---------|------|
| 001 | semantic-simplicity-metrics | unpacking-metrics | standard | UNPACKING derivation, forms and residue are recordable and read as sugar | PARTIAL | simplicity, metrics, unpacking |
| 002 | semantic-simplicity-metrics | iteration-loop-metrics | standard | the derivation table exposes the control-flow gap and redundant loop forms | VALIDATED | simplicity, metrics, control-flow |
| 003 | semantic-simplicity-metrics | lazy-sequence-generators-metrics | standard | the table separates duplicate spellings from a missing suspension mechanism | VALIDATED | simplicity, metrics, generators |
| 004 | semantic-simplicity-metrics | metric-comparison-and-schema | standard | which metrics discriminate and what schema is sufficient | VALIDATED | simplicity, metrics, schema, graph |
