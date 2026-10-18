---
spike: 004
idea: semantic-simplicity-metrics
name: metric-comparison-and-schema
type: standard
validates: "Given the hand-computed metrics of spikes 001-003, when they are compared against intuition about core versus sugar, then the metrics that discriminate can be kept and a per-concept schema can be proposed."
verdict: VALIDATED
related: [001, 002, 003]
tags: [simplicity, metrics, schema, graph, grading]
---

# Spike 004: metric comparison and proposed schema

## Metric definitions used (v0)

| Metric | Definition |
|--------|-----------|
| Idea | A semantic operation. The unit of measurement (not the concept file). |
| P | Grammar productions the concept introduces (not counting named library functions or productions it only inherits). |
| Fdup | Number of ideas denoted by more than one production, excluding named library aliases and one production with two roles. Target 0 unless an exception is recorded. |
| K | Distinct primitives, out of the 10, appearing in the canonical desugarings. |
| G | Kernel gaps: constructs the concept needs that no primitive provides (candidate primitives). Counted as distinct gap families. |
| R | Residue: semantics outside the kernel that are not gaps. Split by kind: static check, typing rule, undecided, implementation guarantee (not counted as semantics). |
| D | Derivation weight: kernel applications in one desugaring template per idea. |
| C | depends_on / depended_on_by, curated; instability = out / (in + out). |
| Symbols | Meanings per symbol or keyword introduced or reused. |

## Comparison

| | UNPACKING | ITERATION_LOOP | LAZY_SEQUENCE_GENERATORS |
|---|---|---|---|
| Ideas | 8 | 7 | 3 (+1 delegated) |
| P | 7 | 5 | 3 |
| Fdup | 0 | 2 | 1 |
| K | 7 | 7 | 6 |
| G | 0 | 1 (control transfer) | 1 (suspend/resume) |
| R (static / typing / undecided) | 4 (2 / 0 / 2) | 2 (1 / 1 / 0) | 3 (0 / 1 / 2) |
| D | 19 | 16 | 7 |
| C out / in (curated) | 8 / 3 | 6 / 4 | 1 / 6 |
| Instability | 0.73 | 0.60 | 0.14 |
| Symbols with more than one meaning | `as` (label, generic bound) | none found | `return` (3) |
| Normative words (diagnostic only) | 1,545 | 891 | 655 |

Intuition used as the test: UNPACKING is sugar and deferrable; LAZY_SEQUENCE_GENERATORS is a mechanism others stand on; ITERATION_LOOP is fundamental control with sugar attached.

## Which metrics discriminate

| Metric | Verdict | Why |
|--------|---------|-----|
| G | **Keep** | 0 / 1 / 1: separates pure sugar from concepts that need a kernel mechanism, and names the mechanism |
| Fdup and Symbols | **Keep** | 0 / 2 / 1 and `as` / `return`: find violations of "one concept, one construct" the concepts' own principles miss |
| Instability and fan-in | **Keep** | 0.73 / 0.60 / 0.14 orders the three exactly as intuition does; fan-in is the closest numeric form of "cost of deferral" |
| R by kind | **Keep, split** | the raw count (4 / 2 / 3) mixes static checks, typing rules and undecided items, which have different meaning |
| D | **Drop as a budget item** | 19 / 16 / 7 is the inverse of core-ness: it measures how much sugar a construct expands into, i.e. leverage, not cost |
| K per concept | **Drop; keep per layer** | 7 / 7 / 6 does not discriminate; kernel size is meaningful for the layer as a whole |
| Words | **Diagnostic only** | the largest concept by text has G = 0 |

## Findings that change the plan

1. **The unit is the idea, not the concept.** UNPACKING bundles eight ideas of different deferral cost (labels are used by call syntax; star has an open scope, DEF-a). Grades must be assignable per idea or per declared slice of a concept.
2. **Dependencies must be curated, with strength.** Mined markdown links undercount badly (UNPACKING out-edges 3 mined against 8 curated; ITERATION_LOOP in-edges 1 against 4). Mixed with "see also" links they also create apparent cycles: over all 57 concept files the mined graph has 79 edges, 14 mutual pairs and 7 strongly connected components, the largest of six (UNPACKING with NAMED_AND_OPTIONAL_PARAMETERS, PATTERN_MATCHING, PATTERN_MATCHING_DISPATCH, STRUCTURAL_TYPING, TRAITS; another of five: ITERATION_LOOP, ITERATOR_PROTOCOL, LAZY_SEQUENCE_GENERATORS, RANGE, SLICE). These are an upper bound. If any remain once `see_also` edges are removed, grades must be assigned per component, not per concept.
3. **One control-transfer gap shows up in two of three concepts**, in contradiction with `PRIMITIVE_BLOCKS.md` in two places: section 10.3 ("control flow: no gaps") against ITERATION_LOOP, and section 10.8 ("generator protocol is `function` + `call`") against the LAZY_SEQUENCE_GENERATORS decomposition row. Hypothesis for Phase 04.4: loop, break, continue, `?`, `?.` and emit/resume are one family (escape, repeat, one-shot resume).
4. **Unresolved dependency owners exist.** ITERATION_LOOP takes the markers `&`, `&mut`, `$` from "Reference, OWNERSHIP, MUTABILITY", which are not registry entries. `depends_on` must accept targets that are kernel documents or research documents, flagged as such.
5. **Redundant forms are real and findable:** `while` against `loop` + `break` (kernel-equal), `break v` against `return v`, `emit` against `return sequence(v)` against `return v ->`.

## Proposed schema (per concept, front matter in the concept file)

```yaml
layer: dynamic            # dynamic | static | staging
grade: 1                  # default for the concept; an idea may override
ideas:
  - id: destructure-positional
    grade: 3              # optional override
    forms:
      - { production: comma-pattern-list, kind: syntax }       # syntax | library-alias | role-of
      - { production: "tuple(...)",        kind: library-alias }
    derives_from: [pack/unpack, assignment, identifier]        # a set; kernel names
    gap: []                                                    # candidate primitive families
    residue:
      - { kind: static-check, note: irrefutability outside match }   # static-check | typing-rule | undecided
symbols:
  - { symbol: "as", meanings: [label, generic-bound] }
depends_on:
  - { target: PATTERN_MATCHING, strength: normative }          # normative | see_also
  - { target: SEMANTIC_MODEL,   strength: normative, kind: kernel-doc }
```

What each metric computes from it: G is the union of `gap`; Fdup counts ideas with more than one `syntax` form; R is the count of `residue` by `kind`; instability and fan-in use `normative` edges only; the monotone-grade and cycle checks run on `normative` edges; the layer-order rule (lower layers never depend on staging) runs on `layer` plus `depends_on`. `D` and per-concept `K` are not stored.

## Limits of this spike

- n = 3, one rater, derivations are judgement-based. Inter-rater agreement and sensitivity of D to the template choice were not measured.
- The mined-graph numbers come from markdown links to sibling concept files, excluding the "Affected Documents" tail; they are an upper bound for cycles and a lower bound for edges.
- The claim that `?` and `?.` belong to the same control family was not derived here; it comes from the registry rows.
- Effort: each concept needed a full read of its `Model (What)` section and judgement calls on ambiguous ideas. Doing all 57 is a multi-session job; starting with the strongly connected components and the grade-1 candidates keeps it bounded.

## Results

Verdict: **VALIDATED**. The metric vector is worth keeping in the reduced form above; the schema needs the `ideas`, `strength`, `symbols` and `kind` fields in addition to what Phase 04.3 planned (`kernel_deps`, `forms`, `residue`, `grade` become the `ideas` list). Recommended changes to the plan: record the schema in the T4 deliverable of Phase 04.3; make the cycle and monotone checks part of Phase 04.5; put the control-transfer family first in Phase 04.4.
