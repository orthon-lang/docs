# Concept Pipeline — End-to-End Lifecycle

> Complete map of the Orthon concept lifecycle: from raw research to
> registered entry in `CORE_CONCEPTS.md`. Every concept that enters the
> language follows this pipeline.
>
> **See also:** [`how/concepts/README.md`](concepts/README.md) (short form),
> [`what/concepts/README.md`](../what/concepts/README.md) (accepted concepts),
> [`PROCESS_INVENTORY.md`](PROCESS_INVENTORY.md) (tool catalogue)

---

## Pipeline Overview

```
how/concepts/research/{NAME}.md    ◄── RESEARCH INBOX
        │                                Raw analysis from other languages.
        │                                Input material, not Orthon spec.
        ▼
┌─────────────────────────────┐
│ Decision Pipeline (10 Qs)   │  ◄── PRE-FILTER
│ how/process/                │      Is this a language feature at all?
│   DECISION_PIPELINE.md      │      Exit paths:
└──────────┬──────────────────┘       • "Library problem" → REJECT
           │                          • "Exists in primitives" → REJECT
           │                          • Syntactic sugar → mark accordingly
           ▼                          • Optimisation → move to OPTIMIZATION_MODEL
┌─────────────────────────────┐      • ACCEPT / DEFER / REJECT
│ Primitive Decomposition     │  ◄── DECOMPOSITION CHECK
│ Check                       │      Does the concept decompose to
│ what/PRIMITIVE_BLOCKS.md    │      the minimal primitive set?
│ what/SEMANTIC_MODEL.md      │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ Semantic Layer              │  ◄── LAYER CLASSIFICATION
│ Classification              │      Level 0 (Data), Level 1 (Primitive Ops),
│ how/gates/                  │      Level 2 (Patterns), Level 3+ (Lib)
│   _language-design.md §1    │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ Concept Design Review       │  ◄── DETAILED DESIGN
│ (11 steps)                  │      1. Problem         7. Interactions
│ how/concept-design-review.md│      2. Use Cases       8. Rationale
└──────────┬──────────────────┘      3. Necessity       9. Examples
           │                         4. Sufficiency    10. Open Questions
           │                         5. Alternatives   11. EDR
           │                         │
           │                         ├── Step 7 → feeds what/CROSS_CUTTING.md
           │                         └── Step 11 → triggers EDR creation
           ▼
┌─────────────────────────────┐
│ Decision Validation Gates   │  ◄── VALIDATION
│ (6 + 1 gates)               │      Each gate = binary verdict or flag
│ how/gates/                  │
│   DECISION_VALIDATION.md    │      Required gates (new construct):
└──────────┬──────────────────┘      • USER_VALUE_GATE
           │                         • LOGICAL_CONSISTENCY_GATE
           │                         • CONCEPTUAL_SIMPLICITY_GATE
           │                         • ARCHITECTURAL_INTEGRITY_GATE
           │                         • IMPLEMENTATION_INDEPENDENCE_GATE
           │                         • LONG_TERM_MAINTAINABILITY_GATE
           │                         • LLM_GENERABILITY_GATE
           │                         (Fewer gates for syntax-only or
           │                          stdlib-only proposals — see
           │                          DECISION_VALIDATION.md § Gate Selection)
           ▼
┌─────────────────────────────┐
│ Language Design Gate        │  ◄── OPERATIONAL CHECKLIST
│ Checklist                   │      19 criteria — all must pass
│ how/gates/                  │      Records verdict in Decision Journal
│   _language-design.md       │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ EDR (Architecture category) │  ◄── FORMAL RECORD
│ how/decision_records/       │      Context · Decision · Consequences
│   architecture/             │      Alternatives · Related Concepts
│   EDR-NNN-{name}.md        │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ what/concepts/{NAME}.md     │  ◄── ORTHON-SPECIFIC DRAFT
│                             │      Accepted design — Orthon semantics
│                             │      (not raw research)
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ Cross-Cutting Review        │  ◄── PHASE 6 INTEGRATION
│ what/CROSS_CUTTING.md       │      Pair-wise interaction analysis
│ what/CONFLICT_REGISTRY.md   │      Conflict detection & resolution
│ notes/interaction-matrix-   │      (happens after multiple concepts
│   format.md                 │       are accepted — M1, Phase 6)
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ what/CORE_CONCEPTS.md       │  ◄── REGISTRY
│                             │      Final entry: accepted, validated,
│                             │      cross-checked concept
└─────────────────────────────┘
```

---

## Stage Details

### 1. Research Inbox

**Location:** `how/concepts/research/{NAME}.md`
**Documents:** [`how/concepts/research/README.md`](concepts/research/README.md)

Raw analysis of a language concept from other languages (Python, Rust,
Java, etc.). Files here are **input material** — they ask *"how do other
languages solve this?"* rather than *"how does Orthon solve this?"*

**Format per file:**
1. Problem — what problem does this concept solve?
2. Examples — how do other languages solve it?
3. Implications for Orthon
4. Open Questions

**No gate required** — research is low-friction. Anyone can add a research file.

---

### 2. Decision Pipeline (10-Question Pre-Filter)

**Documents:** [`how/process/DECISION_PIPELINE.md`](process/DECISION_PIPELINE.md) (10-question pre-filter),
[`how/process/DECISION_PROCESS.md`](process/DECISION_PROCESS.md) (decision authority)

10 sequential questions determine whether a concept should exist
as a language feature. See [`DECISION_PIPELINE.md`](process/DECISION_PIPELINE.md) for the full list.

**Exit paths:**
- **REJECT** — library problem, exists in primitives, violates principle
- **Syntactic sugar** — mark as sugar; no new semantics
- **Optimisation** — move to `what/OPTIMIZATION_MODEL.md`
- **ACCEPT / DEFER** — proceeds to next stage

**Status:** Placeholder — to be populated during M1 Phase 4.

---

### 3. Primitive Decomposition Check

**Documents:** [`what/PRIMITIVE_BLOCKS.md`](../what/PRIMITIVE_BLOCKS.md),
[`what/SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md)

Verify that the concept decomposes into the minimal orthogonal set of
primitives (identifier, literal, assignment, function, call, attribute
access, scope, reference, pack, unpack, operator definition). If the
concept cannot be decomposed, the primitive set is incomplete — this
triggers a revision of either the concept or the primitives.

**Status:** Placeholder — `PRIMITIVE_BLOCKS.md` to be filled during M1 Phase 3.

---

### 4. Semantic Layer Classification

**Document:** [`how/gates/_language-design.md`](gates/_language-design.md) § Semantic layer classification

Every concept must be classified into the Semantic Dependency
Architecture:

| Level | What | Examples |
|-------|------|----------|
| **Level 0** — Data Model | What exists | Data, Data Modifiers, Representations |
| **Level 1** — Primitive Ops | Atomic, non-decomposable operations | (must justify why not decomposable) |
| **Level 2** — Language Patterns | Compositions of L0–L1 — no new semantics | (composition formula required) |
| **Level 3+** — Std Lib, Frameworks | Built on patterns; no special machinery | (library, not language) |

Level 1 claims require justification. Level 2 claims require a
composition formula. See EDR-012 for the full classification rule.

---

### 5. Concept Design Review (11 Steps)

**Document:** [`how/concept-design-review.md`](concept-design-review.md)

The core design workflow. Each concept undergoes 11 steps in order —
see [`concept-design-review.md`](concept-design-review.md) for the full
procedure with step descriptions and gate mappings.

**Step 7 (Interactions)** directly feeds into the interaction matrix
(consolidated in `what/CROSS_CUTTING.md` during M1 Phase 6).
**Step 11 (EDR)** creates the Architecture-category EDR file.

---

### 6. Decision Validation Gates

**Document:** [`how/gates/DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md)

After the 11-step design, the concept is evaluated through 7 independent
validation gates. Each gate produces a binary verdict (pass / fail) or
a conditional flag. Any **fail** or unresolved **flag** means the
concept must be revised.

See [`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md) for the gate
catalogue with criteria tables, and § Gate Selection for the matrix of
which gates apply to which decision type.

---

### 7. Language Design Gate Checklist

**Document:** [`how/gates/_language-design.md`](gates/_language-design.md)

19 operational criteria that operationalise the validation gates into
concrete yes/no checks. Every criterion must be satisfied before the
concept enters specification. The Decision Journal at the end records
each verdict with date, concept, verdict, and rationale.

See [`_language-design.md`](gates/_language-design.md) for the full
checklist.

---

### 8. EDR — Engineering Decision Record

**Location:** `how/decision_records/architecture/EDR-NNN-{name}.md`
**Template:** [`how/templates/_edr-architecture.md`](templates/_edr-architecture.md)
**Index:** [`how/decision_records/INDEX.md`](decision_records/INDEX.md)

Formal decision record documenting:
- **Context** — why the decision was needed
- **Decision** — the choice made and why
- **Consequences** — positive and negative impacts
- **Alternatives** — what was considered and why rejected
- **Related Concepts** — links to the concept draft

The EDR number (`NNN`) is the next available sequence from the
unified EDR journal (`decision_records/INDEX.md`).

---

### 9. Accepted Concept Draft

**Location:** `what/concepts/{NAME}.md`
**Documents:** [`what/concepts/README.md`](../what/concepts/README.md)

The Orthon-specific design document — not raw research, but the
language's own specification for this concept. Follows the
[`_concept.md` template](templates/_concept.md):
Issue → Principles → Policy Footprint → Model → Default Strategy
→ Alternative Strategies → Open Questions → Decision History.

This is the first "official" Orthon artifact for the concept.

---

### 10. Cross-Cutting Review (Phase 6)

**Documents:** [`what/CROSS_CUTTING.md`](../what/CROSS_CUTTING.md),
[`what/CONFLICT_REGISTRY.md`](../what/CONFLICT_REGISTRY.md),
[`notes/interaction-matrix-format.md`](../notes/interaction-matrix-format.md)

Happens after multiple concepts are accepted (M1, Phase 6). Each concept's
interaction data (from Concept Design Review Step 7) feeds into the
pair-wise interaction matrix. Conflicts are tracked in the Conflict Registry
with resolution status. Goal: zero open conflicts at end of Phase 6.

**Required gates:** `ARCHITECTURAL_INTEGRITY_GATE`, `LOGICAL_CONSISTENCY_GATE`
reapplied at the interaction level.

---

### 11. Registry Entry

**Location:** `what/CORE_CONCEPTS.md`

The final step. The concept receives a brief registry entry in
`CORE_CONCEPTS.md` with:
- Concept name and one-line description
- Link to the full concept draft in `what/concepts/`
- Link to the acceptance EDR
- Status indicator (Accepted)

This is the public face of the concept — the registry that answers
*"what concepts does Orthon have?"*

---

## Rejection & Bypass Paths

Not every concept follows the full pipeline. The Decision Pipeline
(Stage 2) may redirect a proposal to one of these alternate paths:

```
                          ┌── REJECT (library problem)
                          │
Proposal ──► Decision ────┼── Syntactic Sugar ──► Documented as syntax
              Pipeline    │                        variant; no new semantics
                          ├── Optimisation ──────► what/OPTIMIZATION_MODEL.md
                          │
                          └── ACCEPT ────────────► Full pipeline (stages 3–11)
```

A proposal can also be **DEFER**red — acknowledged as valuable but
postponed to a later milestone (recorded in `when/ROADMAP.md`).

---

## Milestone Context

| M1 Phase | Pipeline Stages Active |
|----------|----------------------|
| **Phase 1** — Foundation | Process infrastructure (Decision Process, Design Review collapsed) |
| **Phase 2** — Semantic Model | Stage 3 (Primitive Decomposition) input — `SEMANTIC_MODEL.md` |
| **Phase 3** — Primitive Blocks | Stage 3 input — `PRIMITIVE_BLOCKS.md` |
| **Phase 4** — Derived Features | **Stages 1–9** active — research → accepted drafts |
| **Phase 5** — Syntax | Syntax design (parallel to Phase 6) |
| **Phase 6** — Cross-Cutting | **Stage 10** — pair-wise interaction matrix, conflict resolution |
| **Phase 7** — Execution & Opt. | Optimisation bypass path active |
| **Phase 8** — Evolution & Freeze | All stages complete; final registry audit |

**See also:** [`when/ROADMAP.md`](../when/ROADMAP.md) for full milestone definitions.

---

## Cross-Reference Summary

Every stage links to its owning document inline above. For a complete
document map of the project, see [`AGENTS.md`](../AGENTS.md) §3.

**Key pipeline anchors:**
| Stage | Document |
|-------|----------|
| Research Inbox | [`how/concepts/research/README.md`](concepts/research/README.md) |
| Decision Pipeline | [`how/process/DECISION_PIPELINE.md`](process/DECISION_PIPELINE.md) |
| Primitive Decomposition | [`what/PRIMITIVE_BLOCKS.md`](../what/PRIMITIVE_BLOCKS.md) |
| Layer Classification | [`how/gates/_language-design.md`](gates/_language-design.md) §1 |
| Concept Design Review | [`how/concept-design-review.md`](concept-design-review.md) |
| Validation Gates | [`how/gates/DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md) |
| Language Design Gate | [`how/gates/_language-design.md`](gates/_language-design.md) |
| EDR (Architecture) | [`how/templates/_edr-architecture.md`](templates/_edr-architecture.md) |
| EDR Index | [`how/decision_records/INDEX.md`](decision_records/INDEX.md) |
| Accepted Concepts | [`what/concepts/README.md`](../what/concepts/README.md) |
| Cross-Cutting | [`what/CROSS_CUTTING.md`](../what/CROSS_CUTTING.md) |
| Registry | [`what/CORE_CONCEPTS.md`](../what/CORE_CONCEPTS.md) |
| — | — |
| Concept Template | [`how/templates/_concept.md`](templates/_concept.md) |
| Core Inclusion Filter | [`how/architecture/CORE_INCLUSION_FILTER.md`](architecture/CORE_INCLUSION_FILTER.md) |
| Process Inventory | [`PROCESS_INVENTORY.md`](PROCESS_INVENTORY.md) |
| Roadmap | [`when/ROADMAP.md`](../when/ROADMAP.md) |
