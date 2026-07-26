---
date: "2026-07-26 00:00"
promoted: false
---

## Tier (essential/important/deferrable) is not the same axis as Phase (2/3/4)

### The confusion this resolves

`how/concepts/research/README.md` and `SEED-001` both describe the tier
directories (`essential/`, `important/`, `deferrable/`, `reject/`) as a
priority ordering purely *within* Phase 4 ("Phase 4, first pass" / "second
pass" / "deferred"). But `essential/` is described as "semantic bedrock —
the language's skeleton, without which Orthon cannot exist" — which is also
the exact description of Phase 2 (Semantic Model: Identity, Ownership,
Mutation, Evaluation, Visibility, Lifetime) and Phase 3 (Primitive Blocks:
the minimal orthogonal primitive set). Treating all of `essential/` as
"Phase 4 first pass" collapses two different axes into one:

- **Tier** = priority/importance (which concepts matter most)
- **Phase** = decision type (semantic dimension vs. primitive construct vs.
  feature built on top of primitives)

### Decision

`essential/` splits across phases instead of mapping 1:1 to Phase 4:

- Files defining one of the six Semantic Model dimensions → **feed Phase 2**
- Files defining a primitive construct / composition rule → **feed Phase 3**
- Files defining a feature built on top of primitives → **stay in Phase 4**,
  same status as `important/` and `deferrable/`

A fourth pocket also surfaced: a few `essential/` files are **Implementation
Strategy / Policy** material (per `what/GLOSSARY.md`'s Policy definition),
not Core Language semantics at all — see [[move-policy-level-essential-concepts-out-of-pipeline]].

### Classification of the 40 `essential/` files

**→ Phase 2 (Semantic Model)**
DATA_MODEL, OWNERSHIP, OWNERSHIP_METAPROPERTY, OWNERSHIP_TRANSFER_OPERATOR,
MUTABILITY, VALUE_SEMANTICS, IDENTITY_BASED_SAFETY,
VISIBILITY_AND_ENCAPSULATION, SCOPED_RESOURCE_LIFECYCLE,
EXPRESSION_ORIENTED_LANGUAGE

**→ Phase 3 (Primitive Blocks)**
FOUNDATIONAL_ABSTRACTIONS, EXCLUSIVE_DECLARATIONS, STRUCT_AS_VALUE_TYPE,
CLASS_WITH_ACT, ACT_AS_FUNCTION, FUNCTIONS, FINAL_BY_DEFAULT, NAMESPACES,
DELEGATE, COMPOSITION_OVER_INHERITANCE

**→ Phase 4 (Derived Features & Decision Pipeline) — unchanged**
AST_MACROS, COMPILER_AS_STATIC_ANALYZER, COMPILE_TIME_EXECUTION,
COMPOSABLE_COLLECTION_OPS, CONCURRENCY_MODEL, EQUALITY, ERROR_HANDLING,
ERROR_UNION, GENERICS, ITERATOR_PROTOCOL, LAZY_SEQUENCE_GENERATORS,
NULL_SAFETY, PATTERN_MATCHING, PATTERN_MATCHING_DISPATCH, TRAITS,
TYPE_INFERENCE, TYPE_LEVEL_NULL_SAFETY

**→ Policy pocket (neither Phase 2/3 nor Phase 4 — see linked todo)**
ALLOCATION, REGION_BASED_MEMORY_MANAGEMENT, EXECUTION_PROGRAM

> **Caveat (2026-07-26, `/gsd-explore`):** `REGION_BASED_MEMORY_MANAGEMENT`
> is not a clean Policy-pocket member — it embeds semantic ownership/mutation
> claims (linear ownership, no reference types) alongside genuine allocation
> mechanics. It needs to be *split* between Phase 2 and the Policy pocket,
> not moved wholesale. See [[2026-07-26-no-separate-memory-phase]] and the
> updated [[move-policy-level-essential-concepts-out-of-pipeline]] todo.

### Why this matters for the plan

`.planning/REQUIREMENTS.md`'s `SEM-01..03` (Phase 2) and `PRIM-01..03`
(Phase 3) currently don't name a single source file — they read as if the
Semantic Model and Primitive Blocks get authored from scratch. In fact
~20 of the 40 `essential/` files are exactly the raw material for those two
phases. Meanwhile Phase 4's `CONCEPT-01..13` still hardcodes a 13-item list
from when the research directory held ~22 files total; the real Phase 4
input is now ~114 files (38 important + 54 deferrable + 17 essential minus
the Phase 2/3 and Policy carve-outs + reject/ once populated).

This also means `SEED-001` (planted 2026-07-22) is now partially stale: it
treats the whole essential tier as "Phase 4, first pass." It's still the
right source for the *important/deferrable* triage tables, but its essential-tier
table needs the Phase 2/3/4/Policy split applied before Phase 4 execution
starts. See [[rewrite-phase-2-3-4-requirements-for-concept-inventory]].
