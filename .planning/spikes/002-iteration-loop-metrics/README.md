---
spike: 002
idea: semantic-simplicity-metrics
name: iteration-loop-metrics
type: standard
validates: "Given the accepted ITERATION_LOOP concept, when its constructs are derived over the 10 primitives, then the metrics expose the control-flow gap and the redundant loop forms."
verdict: VALIDATED
related: [001, 003, 004]
tags: [simplicity, metrics, iteration, control-flow, derivation]
---

# Spike 002: ITERATION_LOOP metrics

Source: `what/concepts/ITERATION_LOOP.md` (EDR-053, amended by EDR-094). Metric definitions: see Spike 004.

## What This Validates

Given ITERATION_LOOP, when each construct is derived over the kernel, then the derivation table surfaces what the closure audit (Phase 04.4) is meant to find: constructs the concept itself calls "primitive" but the 10 primitives do not contain, and constructs that are derivable from each other.

## Investigation Trail

1. Tried to derive `for` from the concept's own desugaring (`it = coll@iter(); loop: match it@next(): Some(x) -> ...; None -> break`). Every step has a kernel element **except `loop` and `break`**.
2. Checked `PRIMITIVE_BLOCKS.md`: no branch, loop or jump among the 10; section 10.3 lists the control-flow category with "gaps: none" while the concept's decomposition row says "`loop` -> primitive infinite loop; `break`/`continue` -> primitive control flow". The two statements contradict each other (todo `fix-control-flow-primitive-contradiction`).
3. Applied the "equal kernel expression means one idea" test to the loop forms: `while c: body` is `loop: if not c then break; body`. `for` is `loop` plus the iterator protocol. So all three loop forms reduce to one control primitive.
4. Found a second duplicate: in value position, `break v` and `return v` both yield `Some(v)`.

## Ideas, forms and derivations

| # | Idea | Grammar production(s) | Derivation over the kernel | Kernel elements |
|---|------|----------------------|----------------------------|-----------------|
| 1 | Iterate a sequence | `for x in it` | `it = coll@iter(); loop: match it@next(): Some(x) -> body; None -> break` | assignment x2, call x2, unpack, scope, **loop (gap)**, **break (gap)** |
| 2 | Repeat while a condition holds | `while c` | `loop: if not c then break; body` | scope, call, **loop, break (gap)** |
| 3 | Repeat without a condition | `loop` | the control primitive itself | **loop (gap)** |
| 4 | Early exit | `break`, `break v`, `continue` | control transfer | **break, continue (gap)** |
| 5 | Optional search (`for` as expression) | same `for`, position-dependent | `Some(v)` / `None` construction | pack x2, assignment |
| 6 | Iteration ownership | markers `&mut coll`, `$coll` (inherited, not defined here) | reference (shared, exclusive), move = assignment | reference x2, assignment |
| 7 | Index iteration | `1..n` (delegated to RANGE) | a range value implementing `IntoIterator` | call (delegated) |

## Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Ideas | 7 (6 owned, 1 delegated) | |
| P, grammar productions introduced | 5 | `for`, `while`, `loop`, `break` (with optional value), `continue` |
| Fdup, ideas with more than one production | 2 | repetition with exit: `while` against `loop` + `break` (kernel-equal); exit with value: `break v` against `return v` |
| K, kernel elements used | 7 of 10 | scope, assignment, identifier, call, function, pack/unpack, reference |
| G, kernel gaps | 1 candidate primitive (control transfer: loop, break, continue, break-with-value) | used by 4 constructs; the concept calls them "primitive" |
| R, residue | 2 | iterator-invalidation freeze rule (borrow check); position-dependent typing of `for` (`Optional<T>` in value position, nothing in statement position). One more inherited from OWNERSHIP (read-XOR-write) is not counted here |
| D, derivation weight | 16 | for: 6, while: 2, loop: 1, for-expression: 3, ownership: 3, range: 1 |
| C, curated depends_on / depended_on_by | 6 / 4 | out: ITERATOR_PROTOCOL, LAZY_SEQUENCE_GENERATORS (see also), NULL_SAFETY, RANGE, UNPACKING, PATTERN_MATCHING; in: RANGE, ITERATOR_PROTOCOL, UNPACKING, GENERATORS. Plus one unresolved owner: the markers `&`, `&mut`, `$` are "owned by Reference, OWNERSHIP, MUTABILITY", which are not registry entries |
| C, mined from markdown links | 4 / 1 | |
| Instability, curated | 0.60 | |

## Results

Verdict: **VALIDATED**. The derivation table did what the audit needs it to do:

- G isolates the missing primitive (one candidate, four constructs) mechanically, instead of by a reader noticing the contradiction.
- Fdup = 2 finds duplicates that the concept's own principles ("one iteration construct", "`while` for conditions") overlook: `while` adds nothing to `loop` + `break`.
- For Wirth's test ("what representation does it hide?"): `for` hides the iterator protocol (abstraction, keep); `while` hides nothing (sugar, removal or grade-3 candidate); `loop` is the control primitive itself.

Mined links found only 1 inbound edge where curated reading found 4: the dependencies that matter (UNPACKING patterns in `for`, GENERATORS delegation through `for`) are in prose, not in links. The graph cannot be mined; `depends_on` must be curated.

ITERATION_LOOP is in a five-member cycle with ITERATOR_PROTOCOL, LAZY_SEQUENCE_GENERATORS, RANGE, SLICE (mined links, upper bound).
