---
phase: "04"
plan: "06"
subsystem: "derived-features"
tags: [decision-pipeline, important-tier, async-await, generators, concurrency, push-streams, iteration-loop, unpacking, object-initialization, emit]
requires: [04-01, 04-02, 04-03, 04-04, 04-05]
provides: [EDR-047, EDR-048, EDR-049, EDR-050, EDR-051, EDR-052, EDR-053, EDR-054, EDR-055]
affects: [what/CORE_CONCEPTS.md, what/GLOSSARY.md, how/decision_records/INDEX.md, how/process/DECISION_PIPELINE.md]
tech-stack:
  added: []
  patterns: []
key-files:
  created:
    - how/decision_records/architecture/EDR-047-async-await.md
    - how/decision_records/architecture/EDR-049-concurrency.md
    - how/decision_records/architecture/EDR-050-generators.md
    - how/decision_records/architecture/EDR-051-push-streams.md
    - how/decision_records/architecture/EDR-052-emit-as-intermediate-result.md
    - how/decision_records/architecture/EDR-053-iteration-loop.md
    - how/decision_records/architecture/EDR-054-object-initialization.md
    - how/decision_records/architecture/EDR-055-unpacking.md
    - what/concepts/ASYNC_AWAIT.md
    - what/concepts/GENERATORS.md
    - what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md
    - what/concepts/ITERATION_LOOP.md
    - what/concepts/UNPACKING.md
  modified:
    - how/decision_records/INDEX.md
    - how/process/DECISION_PIPELINE.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
decisions:
  - "EDR-047: Async/await as orthogonal modifier on proc/fun/new (combined with async-as-explicit-modifier)"
  - "EDR-048: Skipped — combined into EDR-047"
  - "EDR-049: Concurrency StdLib utilities on delegate model (channels, select, supervision)"
  - "EDR-050: Generators with bidirectional yield and generator expressions"
  - "EDR-051: Push streams as StdLib observable-style reactive streams"
  - "EDR-052: emit as intermediate result — semantic refinement of EDR-021"
  - "EDR-053: Iteration loop — for/while/loop with protocol desugaring"
  - "EDR-054: Object initialization — StdLib patterns using existing mechanisms"
  - "EDR-055: Unpacking/destructuring assignment with pack/unpack symmetry"
metrics:
  duration: "~2 hours"
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 6: Important Tier — Wave 4 Decision Pipeline Summary

## One-Liner

Processed 9 important-tier concepts (ASYNC_AWAIT, ASYNC_AS_EXPLICIT_MODIFIER, CONCURRENCY, GENERATORS, PUSH_STREAMS, EMIT_AS_INTERMEDIATE_RESULT, ITERATION_LOOP, OBJECT_INITIALIZATION, UNPACKING) through the full Decision Pipeline — producing 8 EDRs (EDR-047 through EDR-055, with EDR-048 skipped as combined), 5 concept docs, and updated registries.

## Classification Decisions

| Concept | EDR | Classification | Rationale |
|---------|-----|---------------|-----------|
| ASYNC_AWAIT + ASYNC_AS_EXPLICIT_MODIFIER | EDR-047 (combined) | **Language** | Async as orthogonal modifier on proc/fun/new. Compiler-level state machine transformation. |
| CONCURRENCY | EDR-049 | **StdLib** | Channels, select, supervision, timers — all StdLib utilities on delegate model. |
| GENERATORS | EDR-050 | **Language** | Bidirectional yield adds consumer-to-producer communication beyond EDR-021. |
| PUSH_STREAMS | EDR-051 | **StdLib** | Observable-style streams built on delegate + channel. No new semantics. |
| EMIT_AS_INTERMEDIATE_RESULT | EDR-052 | **Language** (semantic refinement) | Refines EDR-021: emit serves both lazy sequences and intermediate results. |
| ITERATION_LOOP | EDR-053 | **Language** | for/while/loop constructs require compiler-level syntax and desugaring. |
| OBJECT_INITIALIZATION | EDR-054 | **StdLib** | Named params, defaults, copy-and-update are existing mechanisms. |
| UNPACKING | EDR-055 | **Language** | Destructuring syntax matching pack/unpack symmetry. Compiler resolves patterns. |

## Key Decisions Made

1. **Async colourless model** — `Future<T>` is a first-class value; `await` required only when result needed. Eliminates function colouring problem.
2. **Async modifier, not separate kind** — `async` is a modifier on `proc`/`fun`/`new`, preserving Orthon's three-kind declaration system.
3. **Concurrency is StdLib** — All concurrency utilities (channels, select, supervision) are implementable using delegate model primitives. No new language constructs.
4. **`yield` extends `emit`** — `yield` without consumer interaction ≡ `emit`. Bidirectional `yield expr` adds consumer-to-producer communication.
5. **Push streams are StdLib** — Observable-style streams are fully expressible via delegates, channels, and closures. No built-in push model needed.
6. **One iteration construct** — `for ... in` is the only iteration loop. No C-style `for (;;)`. `while` for conditions, `loop` for infinite.
7. **Object initialization uses existing mechanisms** — Named parameters, defaults, copy-and-update syntax are general function call features, not constructor-specific.
8. **Destructuring desugars to pack/unpack** — All destructuring forms desugar to `pack`/`unpack` primitives. No new runtime semantics.

## Deviations from Plan

None — plan executed exactly as specified. All 9 concepts processed with full pipeline (Pipeline Q&A → Classification → 7 Validation Gates → 5-step Design Review via EDR → concept doc → registry updates).

## Commits

- `ea33e7c`: feat(04-06): add Wave 4 important-tier EDRs (EDR-047 through EDR-055)
- `e1156b1`: docs(04-06): add Wave 4 concept docs for Language-classified features
- `b2a2626`: docs(04-06): update registries with Wave 4 concept entries

## Self-Check: PASSED

All created files verified:
- `how/decision_records/architecture/EDR-047-async-await.md` ✓
- `how/decision_records/architecture/EDR-049-concurrency.md` ✓
- `how/decision_records/architecture/EDR-050-generators.md` ✓
- `how/decision_records/architecture/EDR-051-push-streams.md` ✓
- `how/decision_records/architecture/EDR-052-emit-as-intermediate-result.md` ✓
- `how/decision_records/architecture/EDR-053-iteration-loop.md` ✓
- `how/decision_records/architecture/EDR-054-object-initialization.md` ✓
- `how/decision_records/architecture/EDR-055-unpacking.md` ✓
- `what/concepts/ASYNC_AWAIT.md` ✓
- `what/concepts/GENERATORS.md` ✓
- `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` ✓
- `what/concepts/ITERATION_LOOP.md` ✓
- `what/concepts/UNPACKING.md` ✓
