# Phase 4: Derived Features & Decision Pipeline — Research

**Researched:** 2026-07-27
**Domain:** Language feature design pipeline — concept processing, classification, validation, and acceptance
**Confidence:** HIGH

## Summary

Phase 4 processes ~58 design concepts (22 essential-tier, 36 important-tier) plus 4 formal rejections and 54 deferrals through a rigorous pipeline: aggregate research files → Decision Pipeline (10 questions) → all 7 DECISION_VALIDATION.md gates → Concept Design Review (5-step) → EDR → move to `what/concepts/`. This is a **documentation-design phase** — no code, no runtime, no external dependencies. The entire phase operates within the Orthon specification repository.

**Key finding:** The processing pipeline is well-defined by prior phases (EDR-013, EDR-016 patterns). The concept research files already exist with DRAFT headers; they just need formal processing. The 10-question Decision Pipeline in `how/process/DECISION_PIPELINE.md` is already finalized as an accepted document — the pipeline application log is the part that stays empty until Phase 4 fills it. The anti-pattern (ANTIPAT-01..10) requirements from REQUIREMENTS.md are already satisfied by integrated concept research, per CONTEXT.md D-01.

**Primary recommendation:** Batch into 4 waves: (1) essential core concepts in dependency order, (2) remaining essential + borderline + policy concepts, (3) important-tier concepts, (4) rejections + deferrals + LIBRARY_BOUNDARY.md + aggregating EDR. Approximately 65-70 EDRs total (58 concept EDRs + 4 rejection EDRs + 1 aggregating EDR + correction EDRs if needed).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**D-01: Processing Order — Essential-First by Tier**
- Essential-tier concepts (22) processed first, then important-tier (36). Within essential: clean language features before Policy-level edge cases (ALLOCATION, REGION_BASED_MEMORY_MANAGEMENT, EXECUTION_PROGRAM).
- Sub-ordering within clean language features: Follow ROADMAP.md priority table — Core → Control → Data → Meta.
- Borderline concepts (CONTEXT_PARAMETERS, REPRESENTATION_MODIFIERS) processed in this phase alongside essential-tier; permission to correct Semantic Model or Primitive Blocks if analysis reveals inconsistency.
- Anti-pattern research (ANTIPAT-01..10): no separate processing block. Already integrated into concept research docs.

**D-02: Full Treatment per Concept**
- Every concept (essential and important tiers) receives full treatment: aggregate related research files → 10-question Decision Pipeline → all 7 DECISION_VALIDATION.md gates → full Concept Design Review (5-step) → concept-specific EDR.
- Pipeline results documented explicitly.
- Single aggregating EDR for Phase 4 (following EDR-013/016 pattern).

**D-03: Classification Criteria — Both Semantic Uniqueness AND Compiler Dependency**
1. Semantic uniqueness (primary) — adds new semantics not expressible via composition of existing primitives? → Language.
2. Compiler dependency (secondary) — must compiler understand this? → Language.
3. If neither → StdLib or External.
- Classification recorded in each concept's EDR.
- LIBRARY_BOUNDARY.md: lightweight summary table derived from EDR classifications.

**D-04: Policy-Level Concepts — Pipeline with Policy Classification**
- ALLOCATION.md, REGION_BASED_MEMORY_MANAGEMENT.md, EXECUTION_PROGRAM.md run through Decision Pipeline but classified as Policy.
- Filed under `how/strategies/` area rather than `what/concepts/`.

**D-05: Rejection EDRs — Full EDR per Concept**
- Full EDRs for PROTOTYPE.md, SIGNIFICANT_WHITESPACE.md, DYNAMIC_TYPING.md, CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md.
- Context, principle violations, alternatives, formal rejection verdict.
- Move rejected files from deferrable/ to reject/.

**D-06: Deferrable-Tier Deferral Documentation**
- Each ~54 deferrable-tier concept gets one-paragraph deferral rationale in organized reference.
- Rationale: why deferred, target version (v0.2/v0.3), dependency if any.

### Claude's Discretion
- Exact ordering within concept families (which Core concept to process first).
- Combined vs. separate EDRs for closely related concepts.
- Pipeline application log format, LIBRARY_BOUNDARY.md format, deferral table format.
- Canonical form examples formatting in Design Reviews.

### Deferred Ideas (OUT OF SCOPE)
- ~54 deferrable-tier concepts explicitly deferred to v0.2/v0.3.
- Literate Programming — if classified External, out of scope for M1.
- LLM Toolchain components — lighter gate track, timing TBD.
- "Orthon for LLM" research beyond LLM Generability Gate → post-Freeze.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| DERIV-01 | Finalize DECISION_PIPELINE.md and apply to every concept | Pipeline document exists and is accepted; application log is the empty section to fill during Phase 4. Each concept's Q&A trace goes here. |
| CONCEPT-ESS-01 | 22 essential-tier concepts pass Decision Pipeline, accepted or rejected | All 22 essential files exist in `how/concepts/research/essential/` with DRAFT headers. 17 core + 3 policy + 2 borderline. All follow template structure. |
| CONCEPT-IMP-01 | 36 important-tier concepts pass Decision Pipeline (second pass) | All 36 important files exist in `how/concepts/research/important/` with DRAFT headers. |
| CONCEPT-DEFER-01 | 54 deferrable-tier concepts deferred with rationale | 54 files in `how/concepts/research/deferrable/`. SEED-001 provides initial tiering rationale. Deferral table format is Claude's discretion. |
| CONCEPT-REJECT-01 | 4 contradicting-principles concepts formally rejected via EDR | Currently in deferrable/: PROTOTYPE.md, SIGNIFICANT_WHITESPACE.md, DYNAMIC_TYPING.md, CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md. Full EDR per concept required (D-05). |
| ANTIPAT-01..10 | Anti-pattern research (already integrated into concept docs) | Per CONTEXT.md D-01: "The imperative-crutch analyses are already integrated into the concept research docs they inform. The REQUIREMENTS.md requirements are effectively satisfied by the integrated findings." No separate processing pass needed. |
| DERIV-02 | LIBRARY_BOUNDARY.md — language vs stdlib vs external classification | Lightweight summary table derived from EDR classification fields. Format is Claude's discretion. |
| DERIV-03 | Zero DRAFT headers remain in how/concepts/research/ | After processing: accepted concepts move to `what/concepts/`; rejected to `reject/`; deferrable stay in `deferrable/` with DRAFT removed or explicitly marked as deferred. |
</phase_requirements>

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Concept processing pipeline | Process layer (documents) | — | All work is document-authoring: reading research files, writing EDRs, updating pipeline logs. No code, no runtime. |
| Classification (Language/StdLib/External/Policy) | Design layer | — | Each concept EDR records its classification; LIBRARY_BOUNDARY.md is a derived reference. Classification uses D-03 criteria. |
| Policy classification | Implementation Strategy layer | — | Policy-level concepts (ALLOCATION, REGION_BASED_MEMORY_MANAGEMENT, EXECUTION_PROGRAM) route to `how/strategies/` area. |
| Rejection EDRs | Decision Records layer | — | Full EDRs with principle violation mapping, filed in `decision_records/architecture/`. |
| Deferral documentation | Decision Pipeline layer | — | Deferral rationale can live in pipeline application log or aggregating EDR. |
| LIBRARY_BOUNDARY.md | Reference layer (what/) | — | Lightweight summary table, not a design artifact. Derived from EDR classifications. |

## Standard Stack

### Core
| Library/Tool | Version | Purpose | Why Standard |
|-------------|---------|---------|--------------|
| Markdown | — | All document authoring format | This is a documentation-only repository; Markdown is the lingua franca |
| Git | — | Version control and commit tracking | Existing project infrastructure; one intention per commit |

### Supporting
| Tool | Purpose | When to Use |
|------|---------|-------------|
| `mermaid` code blocks | Diagrams in documents (decision flows, dependency graphs) | For pipeline flow diagrams, wave ordering, conceptual dependency visualization |
| `orthon` code blocks | Orthon code examples in concept documents | Per AGENTS.md §5.4: use `orthon` tag for all Orthon example code |
| `text` code blocks | Non-Orthon structural diagrams | Per AGENTS.md §5.4: use `text` for diagrams and pseudo-code |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Per-concept EDRs | Single grouped EDR per wave | User explicitly chose full treatment per concept (D-02). Per-concept EDRs provide traceability. |
| Combined EDRs for related concepts | Separate EDRs per concept | Claude's discretion — combining for trivially related concepts (e.g., CONCURRENCY_MODEL + CONCURRENCY) could reduce EDR count but must maintain per-concept traceability. |

**Version verification:** Phase 4 depends on existing documents only — no external packages to verify. The relevant documents (`DECISION_PIPELINE.md`, `DECISION_VALIDATION.md`, `concept-design-review.md`, `PRIMITIVE_BLOCKS.md`, `SEMANTIC_MODEL.md`) are all verified present and accepted as of 2026-07-27.

## Package Legitimacy Audit

> Not applicable. Phase 4 is a documentation-only phase with no external package dependencies. No npm, PyPI, crates, or other ecosystem packages are required.

## Architecture Patterns

### System Architecture Diagram

```
                                ┌─────────────────────────────────────────────┐
                                │         Phase 4 Processing Pipeline         │
                                └─────────────────────────────────────────────┘
                                             │
                                             ▼
                          ┌─────────────────────────────────────┐
                          │    Step 1: Aggregate Research Files   │
                          │  (Read all related hypothesis files   │
                          │   for a single concept)               │
                          └─────────────────────────────────────┘
                                             │
                                             ▼
                          ┌─────────────────────────────────────┐
                          │   Step 2: Decision Pipeline (10 Qs)  │
                          │  - Q1-Q3: Problem, Layer, Primitives  │
                          │  - Q4-Q7: Principles, Semantics, Sugar│
                          │  - Q8-Q10: Optimization, Compat, Value│
                          │  Record Q&A in pipeline application   │
                          └─────────────────────────────────────┘
                                             │
                                             ▼
                          ┌─────────────────────────────────────┐
                          │   Step 3: Classification (D-03)      │
                          │  Language / StdLib / External /      │
                          │  Policy / Rejection                  │
                          └─────────────────────────────────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
                    ▼                        ▼                        ▼
           ┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
           │ Language/StdLib  │    │ Policy (D-04)    │    │ Rejection (D-05)  │
           │ Continue flow    │    │ Route to         │    │ Full EDR + move   │
           │                  │    │ how/strategies/   │    │ to reject/        │
           └─────────────────┘    └──────────────────┘    └──────────────────┘
                    │
                    ▼
          ┌──────────────────────────────────────────┐
          │ Step 4: 7 Validation Gates               │
          │ USER_VALUE → LOGICAL_CONSISTENCY →        │
          │ CONCEPTUAL_SIMPLICITY → ARCHITECTURAL_    │
          │ INTEGRITY → IMPLEMENTATION_INDEPENDENCE → │
          │ LONG_TERM_MAINTAINABILITY → LLM_          │
          │ GENERABILITY                              │
          └──────────────────────────────────────────┘
                    │
                    ▼
          ┌──────────────────────────────────────────┐
          │ Step 5: Concept Design Review (5-step)   │
          │ 1. Idea/Problem → 2. Minimal Solution →  │
          │ 3. Principle Check → 4. Examples →       │
          │ 5. EDR                                   │
          └──────────────────────────────────────────┘
                    │
                    ▼
          ┌──────────────────────────────────────────┐
          │ Step 6: Create Artifacts                  │
          │ - Concept doc → what/concepts/{NAME}.md   │
          │ - EDR → decision_records/architecture/    │
          │ - Pipeline log entry                      │
          └──────────────────────────────────────────┘
```

### Recommended Project Structure

```
Phase 4 produces or populates:

what/concepts/                  # Destination for accepted concept docs
    {NAME}.md                   # One per accepted Language/StdLib feature

how/strategies/                 # Destination for Policy-level concepts
    {POLICY_NAME}.md            # ALLOCATION, REGION_BASED_MEMORY, EXECUTION_PROGRAM

how/process/DECISION_PIPELINE.md  # Pipeline application log gets populated
                                  # (the document is already finalized)

what/LIBRARY_BOUNDARY.md        # Summary table derived from EDRs

how/decision_records/architecture/
    EDR-NNN-concept-name.md     # Per-concept EDRs (~62-66 total)
    EDR-NNN-aggregating-p4.md   # Single aggregating EDR

how/concepts/research/
    essential/                  # Accepted concepts move OUT to what/concepts/
    important/                  # Accepted concepts move OUT to what/concepts/
    deferrable/                 # Stay with DRAFT replaced by DEFERRED marker
    reject/                     # Rejected concepts move here from deferrable/
```

### Pattern 1: Prior-Phase EDR Pattern (EDR-013, EDR-016)
**What:** Each prior phase produced a single aggregating EDR that: (1) lists source documents and their disposition (Modified/Superseded/Adopted), (2) states the decision as a concise list, (3) lists positive and negative consequences, (4) records compliance with design principles. Phase 4 follows this pattern but adds per-concept EDRs.

**When to use:** For the Phase 4 aggregating EDR. Follow EDR-016's structure: Context (source document table with disposition) → Decision (numbered list of adoptions) → Consequences → Compliance.

### Pattern 2: Concept Design Review (5-step)
**What:** Each concept goes through: (1) Idea/Problem — problem statement from programmer's perspective, (2) Minimal Solution — simplest valid solution, (3) Principle Check — pass/fail per Design Principle, (4) Examples — all canonical forms, (5) EDR — record decision.

**When to use:** Every concept (essential and important). Per D-02, ALL 5 steps are applied to every concept, plus pipeline + gates + EDR.

### Anti-Patterns to Avoid
- **Processing concepts in alphabetical order:** Must follow dependency order (EQUALITY before TYPE_INFERENCE, TRAITS before GENERICS, etc.)
- **Missing the ANTIPAT-01..10 integration:** These are already in concept docs per CONTEXT.md — do NOT create separate anti-pattern documents
- **Writing pipeline context directly as heredoc:** Use the Write tool per execution_flow Step 6 rule

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| EDR template | Custom format from scratch | `how/templates/_edr.md` or `_edr-architecture.md` | Templates already exist with all required fields; using them ensures consistency with prior EDRs |
| Concept doc structure | New format | `how/templates/_concept.md` | 8-section template already specified; all existing research files follow it |
| Validation gates | New criteria | `how/gates/DECISION_VALIDATION.md` | 7 gates with pass/flag/fail criteria already defined per gate |
| LLM Generability Gate | Custom LLM analysis | EDR-014 plus `DECISION_VALIDATION.md` § LLM_GENERABILITY_GATE | Gate criteria already defined with schema-serializable, predictable generation, no hallucination surface checks |

## Common Pitfalls

### Pitfall 1: Processing Order Dependency Violations
**What goes wrong:** A concept is processed before the concepts it depends on are accepted. For example, processing GENERICS.md before TRAITS.md is accepted (generics use trait bounds). Processing TYPE_INFERENCE.md before EQUALITY.md (type inference for custom types depends on equality semantics).
**Why it happens:** Dependencies between concepts are implicit in their research files but not explicitly tracked.
**How to avoid:** Process in documented dependency order within each wave. See Dependency Graph section below.
**Warning signs:** An EDR for concept B references concept A as "settled" when A hasn't been processed yet.

### Pitfall 2: Scope Creep — Processing Too Many Concepts per Wave
**What goes wrong:** A wave tries to process 10+ concepts and each requires full Design Review + gates + EDR, causing the wave to balloon.
**Why it happens:** Estimating Design Review throughput without accounting for the full treatment requirement (Pipeline + 7 gates + 5-step review + EDR writing).
**How to avoid:** Limit waves to 5-8 concepts each. Core concepts with high cross-cutting impact (GENERICS, PATTERN_MATCHING, TRAITS, CONCURRENCY_MODEL) are the most expensive and should be in smaller waves.
**Warning signs:** A plan with 8+ concepts in a single wave that doesn't account for the full treatment overhead.

### Pitfall 3: Overlooking the ANTIPAT Requirement
**What goes wrong:** Planner creates separate ANITPAT-01..10 processing tasks, splitting effort into research that's already done.
**Why it happens:** REQUIREMENTS.md lists ANTIPAT-01..10 as Phase 4 requirements, but CONTEXT.md explicitly states they're already integrated.
**How to avoid:** In the plan, mark ANTIPAT-01..10 as "Satisfied by integrated findings — no separate processing" rather than creating new tasks.
**Warning signs:** Tasks named "ANTIPAT-01 research" or "Write anti-pattern analysis for X."

## Concept Inventory Assessment

### Essential Tier (22 remaining for Phase 4)

| # | File | Exists? | DRAFT? | Template? | Cross-refs | Processing Notes |
|---|------|---------|--------|-----------|------------|------------------|
| 1 | `AST_MACROS.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | COMPILE_TIME_EXECUTION dependency |
| 2 | `COMPILER_AS_STATIC_ANALYZER.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Cross-cutting — affects all concepts |
| 3 | `COMPILE_TIME_EXECUTION.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Core — affects GENERICS, AST_MACROS |
| 4 | `COMPOSABLE_COLLECTION_OPS.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Depends on ITERATOR_PROTOCOL |
| 5 | `CONCURRENCY_MODEL.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | High cross-cutting risk |
| 6 | `EQUALITY.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | **Foundational** — process early |
| 7 | `ERROR_HANDLING.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Core — affects every function |
| 8 | `ERROR_UNION.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Depends on ERROR_HANDLING |
| 9 | `GENERICS.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | **High risk** — depends on TRAITS |
| 10 | `ITERATOR_PROTOCOL.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Depends on LAZY (emit is lazy) |
| 11 | `LAZY_SEQUENCE_GENERATORS.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Uses `emit` (settled in Phase 3) |
| 12 | `NULL_SAFETY.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | **Foundational** — process early |
| 13 | `PATTERN_MATCHING.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Core — high complexity |
| 14 | `PATTERN_MATCHING_DISPATCH.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Depends on PATTERN_MATCHING, TRAITS |
| 15 | `TRAITS.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | **Foundational** — process before GENERICS |
| 16 | `TYPE_INFERENCE.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Depends on EQUALITY, GENERICS |
| 17 | `TYPE_LEVEL_NULL_SAFETY.md` | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Depends on NULL_SAFETY |
| 18 | `ALLOCATION.md` (Policy) | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Policy-level — route to strategies/ |
| 19 | `REGION_BASED_MEMORY_MANAGEMENT.md` (Policy) | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Policy-level — route to strategies/ |
| 20 | `EXECUTION_PROGRAM.md` (Policy) | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | Policy-level — route to strategies/ |
| 21 | `CONTEXT_PARAMETERS.md` (Borderline) | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | May need SEMANTIC_MODEL correction |
| 22 | `REPRESENTATION_MODIFIERS.md` (Borderline) | ✅ in essential/ | ✅ DRAFT | Follows template | TBD | May need PRIMITIVE_BLOCKS correction |

### Important Tier (36 files)
All 36 files exist in `how/concepts/research/important/`. All have DRAFT headers. All follow the `_concept.md` template structure. See `04-CONTEXT.md` § Important-Tier Concepts for the full list.

### Deferrable Tier (54 files)
54 files exist in `how/concepts/research/deferrable/`. Each needs one-paragraph deferral rationale with target version (v0.2/v0.3) and any dependency resolution requirement.

### Rejected Concepts (4, currently in deferrable/)
`PROTOTYPE.md`, `SIGNIFICANT_WHITESPACE.md`, `DYNAMIC_TYPING.md`, `CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md` — currently in `deferrable/`, need formal rejection EDRs and move to `reject/`.

## Dependency Graph

### Essential Tier — Dependency Order (from most foundational)

**Wave 1A — Semantic Preconditions (process first):**
```
EQUALITY.md
    ↓
NULL_SAFETY.md
    ↓
TRAITS.md ──────────────────┐
    ↓                        ↓
ERROR_HANDLING.md         GENERICS.md
    ↓                        ↓
LAZY_SEQUENCE_GENERATORS.md  ↓
    ↓                        ↓
ITERATOR_PROTOCOL.md         ↓
    ↓                        ↓
COMPOSABLE_COLLECTION_OPS.md ↓
    ↓                        ↓
PATTERN_MATCHING.md ────── TYPE_INFERENCE.md
    ↓                        ↓
PATTERN_MATCHING_DISPATCH    TYPE_LEVEL_NULL_SAFETY.md
```

**Wave 1B — Core Language Features:**
```
COMPILE_TIME_EXECUTION.md ──→ AST_MACROS.md
    ↓
CONCURRENCY_MODEL.md
    ↓
COMPILER_AS_STATIC_ANALYZER.md
```

**Wave 1C — Policy and Borderline (processed after core):**
```
ALLOCATION.md (Policy)
REGION_BASED_MEMORY_MANAGEMENT.md (Policy — depends on ALLOCATION)
EXECUTION_PROGRAM.md (Policy — depends on ALLOCATION, CONCURRENCY_MODEL)
CONTEXT_PARAMETERS.md (Borderline — may correct SEMANTIC_MODEL)
REPRESENTATION_MODIFIERS.md (Borderline — may correct PRIMITIVE_BLOCKS)
```

### Important Tier — Approximate Dependency Order

**Wave 2A — Data-related (naturally follow data model):**
```
ALGEBRAIC_DATA_TYPES.md ──→ ENUM_ALTERNATIVES.md
UNION_INTERSECTION_TYPES.md
LITERAL_TYPES.md
STRUCTURAL_TYPING.md
SMART_CAST.md
COPY_ON_WRITE.md
```

**Wave 2B — Control/Execution-related:**
```
ASYNC_AWAIT.md ──→ ASYNC_AS_EXPLICIT_MODIFIER.md
CONCURRENCY.md (may be related to CONCURRENCY_MODEL from essential)
GENERATORS.md (relates to LAZY_SEQUENCE_GENERATORS from essential)
ITERATION_LOOP.md
EMIT_AS_INTERMEDIATE_RESULT.md
PUSH_STREAMS.md
```

**Wave 2C — Ergonomics/Usability:**
```
NAMED_AND_OPTIONAL_PARAMETERS.md
OBJECT_INITIALIZATION.md
UNPACKING.md
PROPERTIES.md
EXTENSION_FUNCTIONS.md
DATACLASSES.md
DECLARATION_BY_ASSIGNMENT.md
DECLARATIVE_CONSTRUCTS.md
DECLARATIVE_MULTI_KEY_SORT.md
SORTING.md (likely StdLib)
SPAN.md
SLOTS.md
```

**Wave 2D — Advanced/Cross-cutting:**
```
CONTRACTS.md
CONTEXT_LIMITED_MODULES.md
DELEGATION.md
COMMAND_PATTERN_VIA_DELEGATE.md
GRADUAL_TYPING.md
TYPE_LEVEL_COMPUTATION.md
PERSISTENT_DATA_STRUCTURES.md
DERIVE_SERIALIZATION.md
COLLECTION_LITERAL_SYNTAX.md
IMMUTABLE_DATE_TIME.md
```

### Cross-Wave Dependencies

- `CONTEXT_PARAMETERS.md` (essential, borderline) may affect `CONTEXT_LIMITED_MODULES.md` (important) — process borderline first
- `PATTERN_MATCHING.md` (essential) affects `SMART_CAST.md` (important) — essential first
- `TRAITS.md` (essential) affects `STRUCTURAL_TYPING.md`, `EXTENSION_FUNCTIONS.md`, `DERIVE_SERIALIZATION.md` (important) — essential first
- `CONCURRENCY_MODEL.md` (essential) affects `CONCURRENCY.md`, `ASYNC_AWAIT.md` (important) — essential first
- `COMPILE_TIME_EXECUTION.md` (essential) affects `TYPE_LEVEL_COMPUTATION.md` (important) — essential first

## Risk Assessment

### High Risk Concepts
| Concept | Risk | Why |
|---------|------|-----|
| **GENERICS.md** | HIGH | Deep cross-cutting effects: affects every type system concept, interacts with TRAITS, TYPE_INFERENCE, COMPILE_TIME_EXECUTION. Design decisions here constrain the entire language. |
| **PATTERN_MATCHING.md** | HIGH | High complexity: exhaustiveness checking, binding semantics, interaction with ownership (consume vs borrow), guard evaluation order, nested patterns. Many open questions in the research doc. |
| **CONCURRENCY_MODEL.md** | HIGH | Cross-cutting: affects memory model, ownership, async, data race guarantees. Incorrect decisions here cascade into Phase 6/7. Must align with SEMANTIC_MODEL's ownership and mutation models. |
| **COMPILE_TIME_EXECUTION.md** | HIGH | Unifies generics, reflection, metaprogramming — a single mechanism serving multiple purposes. If the mechanism can't serve all purposes, the concept splits into separate features. |
| **TRAITS.md** | MEDIUM-HIGH | Underlies GENERICS, PATTERN_MATCHING_DISPATCH, and many important-tier concepts. Coherence rules, orphan rule, dispatch strategy must be settled correctly. |

### Medium Risk Concepts
| Concept | Risk | Why |
|---------|------|-----|
| **EQUALITY.md** | MEDIUM | Seemingly simple but affects Type Inference (type classes for equality), Pattern Matching, hash-based collections. The value semantics vs structural comparison distinction matters. |
| **ERROR_HANDLING.md** | MEDIUM | Monadic Result type interacts with ownership (? consumes or borrows?), evaluation model (short-circuit propagation). Choice affects every function signature. |
| **NULL_SAFETY.md** | MEDIUM | Foundational guarantee but the mechanism (Option type vs nullable references vs flow typing) affects the entire type system. |
| **TYPE_INFERENCE.md** | MEDIUM | Affects generics (Hindley-Milner vs partial inference), pattern matching (type narrowing), smart casting. Scope of inference must be bounded. |
| **COMPILER_AS_STATIC_ANALYZER.md** | MEDIUM | Compiler-level concept — what static analysis guarantees the language provides. Affects how other concepts define their safety contracts. |
| **CONTEXT_PARAMETERS.md** | MEDIUM | Borderline concept that may require SEMANTIC_MODEL correction. Dual parameter model (data vs environment) is a novel design. |

### Low Risk Concepts
| Concept | Risk | Why |
|---------|------|-----|
| **AST_MACROS.md** | LOW | Dependent on COMPILE_TIME_EXECUTION; if that's settled, macro mechanism follows naturally. |
| **COMPOSABLE_COLLECTION_OPS.md** | LOW | Built on ITERATOR_PROTOCOL + traits. Well-understood pattern from Rust/Swift/Kotlin. |
| **ERROR_UNION.md** | LOW | Follows from ERROR_HANDLING; Zig-style error union is a concrete mechanism. |
| **ITERATOR_PROTOCOL.md** | LOW | Built on lazy `emit` (settled in Phase 3 D-06). Well-understood pattern. |
| **LAZY_SEQUENCE_GENERATORS.md** | LOW | `emit` semantics are settled. Generators are syntactic sugar on lazy sequences. |
| **TYPE_LEVEL_NULL_SAFETY.md** | LOW | Depends on NULL_SAFETY; once null safety model is chosen, the type-level expression follows. |
| **PATTERN_MATCHING_DISPATCH.md** | LOW | Depends on PATTERN_MATCHING + TRAITS. Multiple dispatch is a specific extension of matching. |
| **Policy-level concepts** | LOW | Already identified as Policy per D-04. Classification is straightforward; the design work is about how they integrate with Implementation Strategies. |

## Pattern Recognition

### Patterns from Phase 2/3 to Replicate

1. **Aggregating EDR pattern (EDR-013/EDR-016):** Each prior phase created a single aggregating EDR documenting the source documents, their disposition (Modified/Superseded/Adopted), the decision list, consequences, and compliance. Phase 4 follows this pattern but adds per-concept EDRs.

2. **Template-driven consistency:** All research files follow `_concept.md` 8-section template. The EDRs use `_edr-architecture.md` template. This consistency should be maintained.

3. **Source document disposition table:** EDR-013 and EDR-016 both include a table listing each source document and how it was modified, superseded, or adopted. This should be replicated in the Phase 4 aggregating EDR.

4. **One intention per commit (AGENTS.md §10.5):** Each concept processed = one commit. Avoid grouping unrelated concept changes.

### Patterns to Avoid

1. **Skipping Validation Gates:** Per D-02, every concept must pass all 7 gates. Do not shortcut by only applying a subset.

2. **Mixing ANTIPAT work into concept processing:** The anti-patterns are already integrated. Do not create separate "ANTIPAT-01 analysis" tasks.

3. **Creating DRAFT documents for what/concepts/:** The destination documents must be ACCEPTED, not DRAFT. DRAFT is for research files only (which will be removed from research/ after acceptance).

## Wave Strategy

### Recommended Wave Structure

**Wave 1: Essential Core — Precondition Concepts (5 concepts)**
- **Rationale:** These are the most foundational — everything else depends on them being settled.
- **Concepts:** EQUALITY.md, NULL_SAFETY.md, TRAITS.md, ERROR_HANDLING.md, LAZY_SEQUENCE_GENERATORS.md
- **Outputs:** 5 concept docs in what/concepts/, 5 EDRs, 5 pipeline log entries
- **Risk:** LOW-MEDIUM — these are well-understood concepts with strong precedent
- **Depends on:** Phase 3 completed (PRIMITIVE_BLOCKS.md accepted via EDR-016)

**Wave 2: Essential Core — Derived Concepts (7 concepts)**
- **Rationale:** Build on Wave 1's settled preconditions.
- **Concepts:** GENERICS.md, PATTERN_MATCHING.md, ITERATOR_PROTOCOL.md, COMPOSABLE_COLLECTION_OPS.md, ERROR_UNION.md, TYPE_INFERENCE.md, PATTERN_MATCHING_DISPATCH.md
- **Outputs:** 7 concept docs, 7 EDRs, 7 pipeline log entries
- **Risk:** HIGH (GENERICS, PATTERN_MATCHING are highest-risk concepts in the phase)
- **Depends on:** Wave 1 (TRAITS, EQUALITY, ERROR_HANDLING settled)

**Wave 3: Essential Core — System-Level and Advanced (5 concepts)**
- **Rationale:** These are more self-contained or system-level concepts that benefit from having the core settled.
- **Concepts:** COMPILE_TIME_EXECUTION.md, AST_MACROS.md, CONCURRENCY_MODEL.md, COMPILER_AS_STATIC_ANALYZER.md, TYPE_LEVEL_NULL_SAFETY.md
- **Outputs:** 5 concept docs, 5 EDRs, 5 pipeline log entries
- **Risk:** HIGH (CONCURRENCY_MODEL cross-cutting; COMPILE_TIME_EXECUTION unifying mechanism)
- **Depends on:** Wave 2 (GENERICS, TYPE_INFERENCE settled for COMPILE_TIME_EXECUTION)

**Wave 4: Policy-Level + Borderline (5 concepts)**
- **Rationale:** These are separate-tracking items — Policy concepts route to strategies/, borderline may correct earlier documents.
- **Concepts:** ALLOCATION.md, REGION_BASED_MEMORY_MANAGEMENT.md, EXECUTION_PROGRAM.md, CONTEXT_PARAMETERS.md, REPRESENTATION_MODIFIERS.md
- **Outputs:** 3 policy docs in how/strategies/, 2 concept docs in what/concepts/ (or corrections to SEMANTIC_MODEL/PRIMITIVE_BLOCKS), 5 EDRs, 5 pipeline log entries
- **Risk:** MEDIUM (borderline concepts may require cross-document corrections)
- **Depends on:** Wave 3 (CONCURRENCY_MODEL for EXECUTION_PROGRAM)

**Wave 5: Important Tier — Data & Type Concepts (8 concepts)**
- **Rationale:** Natural follow-on from essential data/type concepts.
- **Concepts:** ALGEBRAIC_DATA_TYPES.md, ENUM_ALTERNATIVES.md, UNION_INTERSECTION_TYPES.md, LITERAL_TYPES.md, STRUCTURAL_TYPING.md, SMART_CAST.md, COPY_ON_WRITE.md, SPAN.md
- **Outputs:** 8 concept docs, 8 EDRs, 8 pipeline log entries
- **Risk:** LOW-MEDIUM
- **Depends on:** Wave 2 (GENERICS, TRAITS, PATTERN_MATCHING settled)

**Wave 6: Important Tier — Control & Execution (6 concepts)**
- **Rationale:** Builds on essential control concepts.
- **Concepts:** ASYNC_AWAIT.md, ASYNC_AS_EXPLICIT_MODIFIER.md, CONCURRENCY.md, GENERATORS.md, ITERATION_LOOP.md, EMIT_AS_INTERMEDIATE_RESULT.md, PUSH_STREAMS.md
- **Outputs:** 7 concept docs, 7 EDRs, 7 pipeline log entries
- **Risk:** MEDIUM (ASYNC_AWAIT + CONCURRENCY interaction with CONCURRENCY_MODEL)
- **Depends on:** Wave 3 (CONCURRENCY_MODEL), Wave 2 (ITERATOR_PROTOCOL)

**Wave 7: Important Tier — Ergonomics (10 concepts)**
- **Rationale:** Usability improvements — lowest risk, most straightforward.
- **Concepts:** NAMED_AND_OPTIONAL_PARAMETERS.md, OBJECT_INITIALIZATION.md, UNPACKING.md, PROPERTIES.md, EXTENSION_FUNCTIONS.md, DATACLASSES.md, DECLARATION_BY_ASSIGNMENT.md, DECLARATIVE_CONSTRUCTS.md, DECLARATIVE_MULTI_KEY_SORT.md, SORTING.md, SLOTS.md
- **Outputs:** 11 concept docs, 11 EDRs, 11 pipeline log entries
- **Risk:** LOW (most likely StdLib classification)
- **Depends on:** Wave 2 (TRAITS for EXTENSION_FUNCTIONS, PATTERN_MATCHING for UNPACKING)

**Wave 8: Important Tier — Advanced/Cross-cutting (5 concepts)**
- **Rationale:** Most complex important-tier concepts.
- **Concepts:** CONTRACTS.md, CONTEXT_LIMITED_MODULES.md, DELEGATION.md, COMMAND_PATTERN_VIA_DELEGATE.md, GRADUAL_TYPING.md, TYPE_LEVEL_COMPUTATION.md, PERSISTENT_DATA_STRUCTURES.md, DERIVE_SERIALIZATION.md, COLLECTION_LITERAL_SYNTAX.md, IMMUTABLE_DATE_TIME.md
- **Outputs:** 10 concept docs, 10 EDRs, 10 pipeline log entries
- **Risk:** MEDIUM (GRADUAL_TYPING has adoption strategy implications; TYPE_LEVEL_COMPUTATION is complex)
- **Depends on:** Wave 2 (TRAITS, GENERICS for DERIVE_SERIALIZATION), Wave 4 (CONTEXT_PARAMETERS for CONTEXT_LIMITED_MODULES)

**Wave 9: Rejections, Deferrals, LIBRARY_BOUNDARY.md, Aggregating EDR**
- **Rationale:** Cleanup and consolidation wave. All concepts have been processed.
- **Work:**
  - 4 rejection EDRs (PROTOTYPE, SIGNIFICANT_WHITESPACE, DYNAMIC_TYPING, CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION)
  - Move rejected files to `reject/`
  - 54 deferral rationales in organized table
  - LIBRARY_BOUNDARY.md summary table (derived from EDR classification fields)
  - Aggregating Phase 4 EDR (following EDR-013/016 pattern)
  - DERIV-03: verify zero DRAFT headers remain in `how/concepts/research/`
- **Outputs:** 4 EDRs, 1 reference document, 1 aggregating EDR, deferral table
- **Risk:** LOW
- **Depends on:** All prior waves

### Wave Summary
| Wave | Concepts | EDRs | Output Files | Risk |
|------|----------|------|-------------|------|
| 1 — Preconditions | 5 | 5 | 5 concept + 5 EDR | LOW-MEDIUM |
| 2 — Core derived | 7 | 7 | 7 concept + 7 EDR | HIGH |
| 3 — System/advanced | 5 | 5 | 5 concept + 5 EDR | HIGH |
| 4 — Policy+borderline | 5 | 5 | 3 strategy + 2 concept + 5 EDR | MEDIUM |
| 5 — Important data/types | 8 | 8 | 8 concept + 8 EDR | LOW-MEDIUM |
| 6 — Important control | 7 | 7 | 7 concept + 7 EDR | MEDIUM |
| 7 — Important ergonomics | 11 | 11 | 11 concept + 11 EDR | LOW |
| 8 — Important advanced | 10 | 10 | 10 concept + 10 EDR | MEDIUM |
| 9 — Cleanup | — | 5 | 1 boundary + 1 deferral + 1 aggregating + 4 rejection EDR | LOW |
| **Total** | **58** | **~63** | **~68 files** | |

## Resource Estimate

| Item | Count | Effort per Unit | Total Effort |
|------|-------|-----------------|--------------|
| Essential concept processing (Wave 1-3) | 17 | HIGH (pipeline + 7 gates + 5-step review + EDR + concept doc) | ~34-51 tasks |
| Policy concept processing (Wave 4) | 3 | MEDIUM (pipeline + classification + strategy placement + EDR) | ~3-6 tasks |
| Borderline concept processing (Wave 4) | 2 | MEDIUM-HIGH (pipeline + gates + review + possible corrections to SEMANTIC_MODEL/PRIMITIVE_BLOCKS) | ~3-6 tasks |
| Important concept processing (Wave 5-8) | 36 | MEDIUM (pipeline + gates + review + EDR) | ~54-72 tasks |
| Rejection EDRs (Wave 9) | 4 | LOW (principle check + alternatives + EDR) | ~4 tasks |
| Deferral documentation (Wave 9) | 54 | LOW (one paragraph each) | ~2-3 tasks |
| LIBRARY_BOUNDARY.md (Wave 9) | 1 | LOW (summary table from EDRs) | ~1 task |
| Aggregating EDR (Wave 9) | 1 | MEDIUM (source document tables + decision summary + consequences) | ~1-2 tasks |
| DERIV-03 verification (Wave 9) | 1 | LOW (check DRAFT headers across research/) | ~1 task |
| **Total** | | | **~103-148 tasks** |

**Suggested number of plans:** 9 (one per wave) or fewer by grouping related waves (e.g., Waves 1+2 as one plan for essential core, Waves 3+4 as one plan for advanced essential, Waves 5+6 as one plan for important first half, etc.).

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | All 22 essential-tier concept files have DRAFT headers and follow the `_concept.md` template structure | Concept Inventory Assessment | If some files have non-standard structure, additional pre-processing needed before pipeline |
| A2 | All 36 important-tier concept files exist and have DRAFT headers | Concept Inventory Assessment | Missing files would need to be created, increasing scope |
| A3 | ANITPAT-01..10 requirements are fully satisfied by integrated concept research findings (per CONTEXT.md) | Common Pitfalls | If user intended separate anti-pattern documents, Phase 4 scope undercounted |
| A4 | PRIMITIVE_BLOCKS.md is fully accepted and stable (EDR-016 confirmed) | Dependency | If corrections are needed (borderline concepts), rework propagates |
| A5 | SEMANTIC_MODEL.md is fully accepted and stable (EDR-013 confirmed) | Dependency | If corrections are needed (CONTEXT_PARAMETERS), rework propagates |
| A6 | The DECISION_PIPELINE.md document is finalized and only the application log needs filling | Standard Stack | If the document itself needs changes, DERIV-01 requires editing the pipeline, not just the log |
| A7 | All deferrable concepts (`how/concepts/research/deferrable/`) are indeed 54 files | Resource Estimate | Count mismatch affects DEFER-01 scope |
| A8 | `EQUALITY.md`, `LAZY_SEQUENCE_GENERATORS.md`, `ITERATOR_PROTOCOL.md` are in essential/ (confirmed by file listing) | Dependency Graph | If any were already consumed by Phase 2/3, 17-core count is wrong |

## Open Questions

1. **ANTIPAT-01..10 requirement status vs CONTEXT.md:**
   - What we know: CONTEXT.md explicitly says anti-pattern research is already integrated into concept docs. But REQUIREMENTS.md still lists them as open Phase 4 requirements.
   - What's unclear: Should the Phase 4 plan explicitly mark ANTIPAT-01..10 as "Satisfied by integrated findings" (and create no tasks), or should there still be verification tasks?
   - Recommendation: Follow CONTEXT.md — mark as satisfied. Add a single verification task: "Confirm anti-pattern findings are traceable in concept docs."

2. **`DECLARATION_BY_ASSIGNMENT.md` phase assignment:**
   - What we know: REQUIREMENTS.md notes this is "arguably Phase 5 (Syntax) material." It's currently in important/.
   - What's unclear: Should it be processed in Phase 4 or deferred to Phase 5?
   - Recommendation: Process in Phase 4 as it's already a research file. If classified as pure syntax sugar, the pipeline will catch it and the EDR will note its lightweight treatment.

3. **`CONTEXT_PARAMETERS.md` and `REPRESENTATION_MODIFIERS.md` correction scope:**
   - What we know: Both moved to essential/ mid-cycle and have permission to correct SEMANTIC_MODEL or PRIMITIVE_BLOCKS.
   - What's unclear: What level of correction is acceptable? A minor clarification, or a full re-EDR?
   - Recommendation: Handle corrections as edits to the existing documents + a "correction note" in the concept's EDR. Only trigger a new EDR for SEMANTIC_MODEL/PRIMITIVE_BLOCKS if the correction changes accepted decisions.

## Environment Availability

> Step 2.6: SKIPPED (no external dependencies — this is a documentation-only phase with no tools, runtimes, or services beyond Git and a text editor).

## Validation Architecture

> nyquist_validation is enabled (`.planning/config.json: workflow.nyquist_validation: true`). The Validation Architecture section applies.

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Manual verification — no automated test framework exists for a documentation-only repo |
| Config file | N/A |
| Quick run command | `gsd_run query commit "..." --files <path>` (for file changes) |
| Full suite command | Manual review checklist per concept |

### Phase Requirements → Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| DERIV-01 | DECISION_PIPELINE.md application log filled per concept | Manual review | — | ✅ (pipeline doc exists, log empty) |
| CONCEPT-ESS-01 | 22 essential concepts processed | Manual review | `grep -r "⚠️ DRAFT" how/concepts/research/essential/` | ❌ Wave 0 |
| CONCEPT-IMP-01 | 36 important concepts processed | Manual review | `grep -r "⚠️ DRAFT" how/concepts/research/important/` | ❌ Wave 0 |
| CONCEPT-DEFER-01 | Deferral rationale documented | Manual review | `ls how/concepts/research/deferrable/` | ❌ Wave 0 |
| CONCEPT-REJECT-01 | Rejection EDRs + moved to reject/ | Manual review | `ls how/concepts/research/reject/` | ❌ Wave 0 |
| DERIV-02 | LIBRARY_BOUNDARY.md created | Manual review | `test -f what/LIBRARY_BOUNDARY.md` | ❌ Wave 0 |
| DERIV-03 | Zero DRAFT headers remain in research/ | Automated check | `grep -rl "⚠️ DRAFT" how/concepts/research/` | ❌ Wave 0 |

### Sampling Rate
- **Per task commit:** Manual review of concept doc + EDR + pipeline log entry
- **Per wave merge:** Check that all concept files in wave are created, all EDRs are filed, all pipeline entries exist
- **Phase gate:** Full verification: zero DRAFT headers in research/, all EDRs indexed in INDEX.md, LIBRARY_BOUNDARY.md complete, aggregating EDR accepted

### Wave 0 Gaps
- [ ] No automated test framework — all verification is manual document review
- [ ] No grep-based DRAFT check script — create one as part of Wave 0 or add it to Wave 9
- [ ] No "verify EDR indexed in INDEX.md" check — add to Wave 9 verification

## Security Domain

> `security_enforcement` is `true` in `.planning/config.json`. Security domain applies.

### Applicable ASVS Categories
| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | No | — (language spec phase; no auth implementation) |
| V3 Session Management | No | — (language spec phase; no session logic) |
| V4 Access Control | No | — (language spec phase; visibility model settled in SEMANTIC_MODEL) |
| V5 Input Validation | Yes | Type safety and compiler-level validation — concepts ensure all input is typed and checked at compile time |
| V6 Cryptography | No | — (deferred to Standard Library Phase) |

### Known Threat Patterns for Documentation-Only Phase
| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| Design ambiguity leading to unsafe implementations | Tampering | Every concept passes all 7 validation gates including LOGICAL_CONSISTENCY_GATE and ARCHITECTURAL_INTEGRITY_GATE |
| Missing safety guarantees in spec | Information Disclosure | NULL_SAFETY, TYPE_LEVEL_NULL_SAFETY, COMPILER_AS_STATIC_ANALYZER ensure safety is explicit in the spec |
| Undefined behavior from concept interactions | Denial of Service | Phase 6 (Cross-Cutting) will detect interaction issues, but Phase 4 must document interactions per concept |

## Sources

### Primary (HIGH confidence)
- [CONTEXT.md] — Phase 4 context document with all locked decisions (D-01 through D-06)
- [DECISION_PIPELINE.md] — 10-question pipeline (finalized, accepted document)
- [DECISION_PROCESS.md] — Decision tiers and recording rules (accepted)
- [DECISION_VALIDATION.md] — 7 validation gates with full criteria tables
- [concept-design-review.md] — 5-step procedure (accepted)
- [PRIMITIVE_BLOCKS.md] — 9-primitive set (EDR-016 accepted) — verified from file read
- [SEMANTIC_MODEL.md] — 6-dimension semantic model (EDR-013 accepted) — verified from file read
- [DESIGN_PRINCIPLES.md] — Locked constitution (verified from file read)
- [IMPLEMENTATION_POLICIES.md] — Policy framework (verified from file read)
- [EDR-013] — Semantic model acceptance EDR
- [EDR-016] — Primitive blocks acceptance EDR
- [SEED-001] — Tier triage and sequencing rationale
- [REQUIREMENTS.md] § Phase 4 — DERIV-01..03, CONCEPT-ESS-01, CONCEPT-IMP-01, CONCEPT-DEFER-01, CONCEPT-REJECT-01, ANTIPAT-01..10
- [ROADMAP.md] § Phase 4 — Priority table (Core/Control/Data/Meta)
- [03-CONTEXT.md] — Prior phase locked decisions (D-01..D-10)
- [02-CONTEXT.md] — Prior phase semantic model decisions (D-01..D-06)
- [AGENTS.md] — Agent instructions, review checklist, cross-reference rules

### Secondary (MEDIUM confidence)
- File listings from `ls how/concepts/research/essential/ important/ deferrable/` — verified concept existence and counts

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — documentation-only project, all tools/templates verified present
- Architecture (processing pipeline): HIGH — well-defined by existing procedures and prior phase patterns
- Pitfalls: HIGH — drawn from observed patterns in prior phases and explicit user decisions
- Concept inventory: HIGH — verified via file listing
- Dependency graph: MEDIUM — based on research file analysis, not cross-referenced against concept content
- Risk assessment: HIGH — based on concept complexity and cross-cutting effects visible from research files
- Wave strategy: MEDIUM — wave sizing is an estimate; actual throughput depends on plan granularity

**Research date:** 2026-07-27
**Valid until:** Phase 4 completion (stable — no external factors that would invalidate this research)

## RESEARCH COMPLETE

**Phase:** 04 — Derived Features & Decision Pipeline
**Confidence:** HIGH

### Key Findings
1. **Processing pipeline is well-defined** — Concept Design Review (5-step) + Decision Pipeline (10 Qs) + 7 Validation Gates + EDR per concept + aggregating EDR. This pattern is established from Phases 2/3.
2. **22 essential + 36 important + 54 deferrable + 4 reject** concepts to process. All research files exist with DRAFT headers. Anti-pattern research is already integrated (no separate ANTIPAT-01..10 processing).
3. **Dependency order is critical** — EQUALITY → TRAITS → GENERICS → PATTERN_MATCHING → TYPE_INFERENCE forms the backbone. Processing out of order creates circular dependencies in EDRs.
4. **4 high-risk concepts** (GENERICS, PATTERN_MATCHING, CONCURRENCY_MODEL, COMPILE_TIME_EXECUTION) need careful treatment — they have the most cross-cutting effects.
5. **9-wave strategy** provides clean separation — essential core (3 waves) → policy+borderline (1 wave) → important tier (3 waves) → cleanup/reference/aggregation (1 wave). Total ~63 EDRs.

### File Created
`.planning/phases/04-derived-features-and-decision-pipeline-run-every-outstanding/04-RESEARCH.md`

### Confidence Assessment
| Area | Level | Reason |
|------|-------|--------|
| Standard Stack | HIGH | Documentation-only; all documents verified present |
| Architecture | HIGH | Pipeline well-defined by existing procedures |
| Pitfalls | HIGH | Drawn from prior phases and explicit decisions |
| Dependency Graph | MEDIUM | Based on research file analysis, not full content review |

### Open Questions
- ANTIPAT-01..10: Mark as satisfied per CONTEXT.md or create verification tasks?
- DECLARATION_BY_ASSIGNMENT: Process in Phase 4 or defer to Phase 5?
- Borderline concept correction scope: Minor edits or full re-EDR for SEMANTIC_MODEL/PRIMITIVE_BLOCKS?

### Ready for Planning
Research complete. Planner can now create PLAN.md files using the 9-wave strategy, processing concepts in documented dependency order per wave.
