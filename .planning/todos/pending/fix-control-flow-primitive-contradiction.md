---
title: Resolve the control-flow contradiction between PRIMITIVE_BLOCKS and ITERATION_LOOP
date: 2026-10-18
priority: high
status: pending
---

## What

Two accepted documents contradict each other about control flow:

- `what/concepts/ITERATION_LOOP.md` / its `Primitive Decomposition` row in `what/CORE_CONCEPTS.md`: "`loop` → primitive infinite loop; `break`/`continue` → primitive control flow".
- `what/PRIMITIVE_BLOCKS.md` § 10.3 (Control flow row): decomposes onto `function + call + scope`, gaps: "None"; § 9 lists 10 primitives and none is a branch, loop or jump.

Either control transfer (branching, looping, early exit) is a missing primitive of the dynamic core, or it is derivable and `ITERATION_LOOP` is wrong to call it "primitive". The same question covers `?`, `?.` (early exit) and generator suspend/resume.

A second, related contradiction (found by spike 003, `.planning/spikes/003-lazy-sequence-generators-metrics/README.md`): `what/PRIMITIVE_BLOCKS.md` § 10.8 says "the generator protocol is `function` + `call` composition", while the `LAZY_SEQUENCE_GENERATORS` decomposition row says the state-machine transformation is "beyond primitive composition". Third reading: a generator is a coroutine, i.e. `execution_context` (`defer`, EDR-085). Spikes 002 and 003 suggest that loop, break, continue, `?`, `?.` and emit/resume are one control-transfer family; Phase 04.4 should test that as a single hypothesis.

## Why

Found during the 2026-10-18 exploration, see [[2026-10-18-graded-spec-growth]]. It is the clearest concrete evidence that the "primitive set is complete" claim (EDR-016, § 10.5) is unverified. Resolving it is the first item of the Primitive Closure Audit (Phase 04.4); if control transfer is a primitive, the amendment follows the EDR-085 procedure (new EDR, re-run minimality proof).

## Suggested action

1. Decide: derivable or primitive. Test: write `if`, `loop`, `break`, `?` as expressions over the 10 primitives, or show that it cannot be done.
2. If primitive: amend `PRIMITIVE_BLOCKS.md` via a new EDR (precedent EDR-085) and re-run § 10.6 minimality.
3. If derivable: fix the `ITERATION_LOOP` wording and show the derivation in `PRIMITIVE_BLOCKS.md` § 8.
