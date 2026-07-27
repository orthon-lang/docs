---
phase: 04
plan: 04
name: Wave 3 — Policy Pipeline & Borderline Concepts
subsystem: Decision Pipeline, EDR System, Strategy System
tags: [policy, borderline, decision-pipeline, edr, allocation, execution-program, context-parameters, representation-modifiers]
requires: [04-01, 04-02, 04-03]
provides: [edr-034, edr-035, edr-036, edr-037, edr-038]
affects: [IMPLEMENTATION_POLICIES.md, DEFAULT_STRATEGY.md, DECISION_PIPELINE.md, INDEX.md, SEMANTIC_MODEL.md, PRIMITIVE_BLOCKS.md, GLOSSARY.md, CORE_CONCEPTS.md]
tech-stack:
  added: []
  patterns: [Policy-level pipeline processing, SEMANTIC_MODEL correction pattern, PRIMITIVE_BLOCKS correction pattern]
key-files:
  created:
    - how/decision_records/architecture/EDR-034-allocation.md
    - how/decision_records/architecture/EDR-035-region-based-memory-management.md
    - how/decision_records/architecture/EDR-036-execution-program.md
    - how/decision_records/architecture/EDR-037-context-parameters.md
    - how/decision_records/architecture/EDR-038-representation-modifiers.md
  modified:
    - how/process/DECISION_PIPELINE.md
    - how/IMPLEMENTATION_POLICIES.md
    - how/strategies/DEFAULT_STRATEGY.md
    - how/decision_records/INDEX.md
    - what/SEMANTIC_MODEL.md
    - what/PRIMITIVE_BLOCKS.md
    - what/GLOSSARY.md
    - what/CORE_CONCEPTS.md
decisions:
  - D-04 Policy classification confirmed: ALLOCATION, REGION_BASED_MEMORY, EXECUTION_PROGRAM are Policy, not Language
  - D-03 borderline resolution: CONTEXT_PARAMETERS → SEMANTIC_MODEL correction (EDR-037)
  - D-03 borderline resolution: REPRESENTATION_MODIFIERS → PRIMITIVE_BLOCKS correction (EDR-038)
  - Execution Model Policy added as new Policy type (peer of Allocation, Algorithm, etc.)
  - Region-Based Memory as sub-policy within Allocation Policy
metrics:
  duration: ~15 min
  completed_date: 2026-07-27
status: complete
---

# Phase 4 Plan 4: Wave 3 — Policy Pipeline & Borderline Concepts Summary

Processed 5 concepts through the Decision Pipeline: 3 Policy-classified concepts and 2 borderline concepts. Created 5 EDRs and updated 8 supporting documents.

## What Was Done

### PART A: Policy-classified concepts (ALLOCATION, REGION_BASED_MEMORY, EXECUTION_PROGRAM)

All three confirmed as **Policy** per D-04 — they are implementation choices about HOW primitives are realised, not WHAT the program means. No concept docs created in `what/concepts/`.

**EDR-034: Allocation as Policy** — Allocation Policy with 5 values (Heap, Arena, Linear, GC, Static). Reduced validation gates per D-04 Policy pipeline (USER_VALUE_GATE and LLM_GENERABILITY_GATE skipped).

**EDR-035: Region-Based Memory Management** — Sub-policy within Allocation Policy. 3 values (ScopeRegion, ExplicitRegion, NoRegion). Effective when Allocation Policy = `Arena`.

**EDR-036: Execution Program** — New Execution Model Policy type. 5 values (Interpreted, AOT, JIT, WASM, Container). This is Orthon's core innovation — decoupling semantics from execution strategy.

### PART B: Borderline concepts (CONTEXT_PARAMETERS, REPRESENTATION_MODIFIERS)

Both evaluated through full Decision Pipeline with these classifications:

**CONTEXT_PARAMETERS → SEMANTIC_MODEL correction** (EDR-037): Implicit context flow is a cross-cutting concern of the Evaluation and Visibility dimensions. Not a standalone Language feature. A brief note added to SEMANTIC_MODEL.md § Evaluation acknowledging this design space.

**REPRESENTATION_MODIFIERS → PRIMITIVE_BLOCKS correction** (EDR-038): Representation modifiers (struct, boxed, shared, etc.) are orthogonal annotations on the `pack` and `reference` primitives, not new primitives. Added §7b to PRIMITIVE_BLOCKS.md documenting the decomposition.

### Documents Updated

| Document | Changes |
|----------|---------|
| `DECISION_PIPELINE.md` | Added ### Essential — Policy Level (ALLOCATION, REGION_BASED_MEMORY, EXECUTION_PROGRAM) and ### Essential — Derived Features Wave 3 (CONTEXT_PARAMETERS, REPRESENTATION_MODIFIERS) |
| `IMPLEMENTATION_POLICIES.md` | Updated Allocation Policy with EDR-034 ref; added Region-Based Memory Sub-Policy (EDR-035); added Execution Model Policy (EDR-036) |
| `DEFAULT_STRATEGY.md` | Added Region-Based Memory (ScopeRegion) and Execution Model (AOT) to policy profile |
| `INDEX.md` | Added EDR-034 through EDR-038 |
| `SEMANTIC_MODEL.md` | Added implicit context flow note in Evaluation dimension |
| `PRIMITIVE_BLOCKS.md` | Added §7b Representation Modifiers as orthogonal annotations |
| `GLOSSARY.md` | Added 7 new terms: Allocation Policy, Execution Model Policy, Implicit Context Flow, Policy, Program Enricher, Region-Based Memory, Representation Modifier |
| `CORE_CONCEPTS.md` | Updated status to reflect Wave 3 processing |

## Deviations from Plan

None — plan executed exactly as written.

## Key Decisions

1. **D-04 Policy classification confirmed.** ALLOCATION, REGION_BASED_MEMORY, EXECUTION_PROGRAM all pass the Policy test: no new semantics expressible via primitives, language semantics independent of policy choice, implementation strategy concern.

2. **CONTEXT_PARAMETERS → SEMANTIC_MODEL correction.** Rationale: Context flow is a cross-cutting concern of Evaluation (when is context supplied?) and Visibility (where are `given` instances visible?). It refines existing function/call primitive interaction with scope, not add new primitives.

3. **REPRESENTATION_MODIFIERS → PRIMITIVE_BLOCKS correction.** Rationale: Each modifier (struct, boxed, shared, etc.) decomposes to either `pack` (inline layout) or `reference` (indirection). Adding them as primitives would violate orthogonality.

4. **Execution Model Policy as new Policy type.** Added as peer of Allocation, Algorithm, etc. in the Strategy system. Five values cover the full execution spectrum.

5. **Region-Based Memory as Allocation sub-policy.** Not a standalone Policy — refines the `Arena` value of Allocation Policy.

## Self-Check: PASSED

All 5 EDRs created and verified. All 8 modified files verified. All commits recorded.
