# ROADMAP — Orthon Language Design Roadmap

> Process-oriented milestones for the Orthon language design project.
> Each milestone represents a stage in the design process, from Vision
> to Language Specification Freeze.

---

## Overview

Language design follows a strict sequence of process milestones.
Each milestone produces concrete artifacts and must be substantially
complete before the next begins.

```
M1 — Orthon Language Spec (v0.1)
    │
    ├──► M2 — Standard Library & FFI
    │
    └──► M3 — Build System & Tooling
              │
              ▼
         M4 — Implementation
```

**Key principle: Design before implement.** M1 (Orthon Language Spec) covers
language design only — Phase 1 through Phase 8. M2–M4 (stdlib, tooling,
implementation) begin after the language specification is frozen.

---

## M1 — Orthon Language Spec (v0.1) ⬜

**Goal:** Deliver a frozen, self-consistent v0.1 language specification —
designed through a rigorous architectural pipeline: from vision and principles,
through semantic model and primitive blocks, to derived features, syntax,
cross-cutting verification, execution model, and final freeze.

M1 follows the **11-level engineering design hierarchy**: each level produces
concrete artifacts and gates before the next begins.

```
Phase 1 — Foundation
    │
    ▼
Phase 2 — Semantic Model
    │
    ▼
Phase 3 — Primitive Blocks
    │
    ▼
Phase 4 — Derived Features & Decision Pipeline
    │
    ├──────────────────┐
    ▼                  ▼
Phase 5 — Syntax   Phase 6 — Cross-Cutting
    │                  │
    └────────┬─────────┘
             ▼
    Phase 7 — Execution & Optimization Model
             │
             ▼
    Phase 8 — Evolution Model & Freeze
```

**Dependencies:** Nothing (initial milestone).

---

### Phase 1: Foundation ⬜

**Goal:** Fix architecture stubs, consolidate Vision into one coherent block,
lock Design Principles as the constitutional level. Establish process
infrastructure so subsequent phases have a defined acceptance procedure.

**Steps:**

1. **Remediate concerns** — resolve architecture gaps, tech debt, governance
   issues from current codebase audit:
   - Fill `IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md` stubs
   - Archive stale templates (`_adr.md`, `EXECUTION_IMAGE.md`)
   - Date-stamp/freshness metadata on all concept and architecture documents
   - Complete `FITNESS_FUNCTIONS.md` catalogue
   - Expand documentation principles beyond stub
   - Repository-wide cross-reference link audit
   - Document Glossary maintenance workflow

2. **Consolidate Vision** — produce a compact canonical `VISION.md` structured as:
   **Vision · Mission · Non-goals · Target Audience · Success Criteria**
   Keep existing docs (`GOALS.md`, `MANIFESTO.md`, `ZEN.md`, `WORKING_BACKWARDS.md`)
   as reference expansions.

3. **Lock Design Principles** — `DESIGN_PRINCIPLES.md` declared **closed for
   modification** (changes only via EDR). Every subsequent design decision
   references specific principles.

4. **Process infrastructure:**
   - `DECISION_PROCESS.md` — one-page decision authority (from GAP-03)
   - Collapse 11-step Concept Design Review → 5-step pipeline for solo author
   - Add throughput fitness functions to `FITNESS_FUNCTIONS.md`

**Vision & Philosophy deliverables:**

| Area | Deliverables | Status |
|------|-------------|--------|
| **Vision & Philosophy** | `VISION.md` (consolidated), `MANIFESTO.md`, `ZEN.md`, `GOALS.md` | 🔄 In Progress |
| **Design Method** | `WORKING_BACKWARDS.md` | 🔄 In Progress |
| **Design Principles** | `DESIGN_PRINCIPLES.md` — locked as constitution | 🔄 In Progress |
| **Decision Infrastructure** | `DECISION_VALIDATION.md`, `_language-design.md`, `IMPLEMENTATION_POLICIES.md`, TDR-001–TDR-009 | 🔄 In Progress |
| **Validation Methods** | Socratic, Scientific, Logical Analysis, TRIZ, Einstein | 🔄 In Progress |
| **Templates** | EDR templates (Architecture, Process, base), Design review template, Concept template | 🔄 In Progress |
| **Meta** | `README.md`, `AGENTS.md`, `GLOSSARY.md`, Documentation Principles, Philosophy | 🔄 In Progress |

**New artifacts:**
- `VISION.md` — consolidated (Vision · Mission · Non-goals · Audience · Success Criteria)
- `how/process/DECISION_PROCESS.md` — one-page decision authority
- `FITNESS_FUNCTIONS.md` — completed catalogue + throughput functions

**Exit criteria:**
- [ ] All concerns remediated or explicitly deferred
- [ ] Vision consolidated and approved
- [ ] Design Principles declared closed; change requires EDR
- [ ] Decision process documented and referenced from `AGENTS.md`
- [ ] All former `TODO.md` items migrated to tracked GSD requirements

---

### Phase 2: Semantic Model ⬜

**Goal:** Define *what a program means* at the foundational level — identity,
ownership, mutation, evaluation, visibility, lifetime — as a single unified
model. This is the most expensive part of language design; changing it later
is extremely costly.

**Steps:**

1. **Inventory existing semantic research:**
   - `DATA_MODEL.md` — types, representations (Value, Tuple, Reference, etc.)
   - `OWNERSHIP.md` — ownership semantics
   - `MUTABILITY.md` — mutation model
   - `ALLOCATION.md` — memory allocation
   - `EQUALITY.md` — equality semantics
   - `FOUNDATIONAL_ABSTRACTIONS.md` — Data + Data Modifiers hypothesis

2. **Create `SEMANTIC_MODEL.md`:**
   Define each semantic dimension as a coherent whole:
   - **Identity** — What does it mean for two values to be "the same"?
   - **Ownership** — Who owns data? Linear, shared, borrowed?
   - **Mutation** — When and how do values change? Are data immutable by default?
   - **Evaluation** — When are expressions evaluated? Eager, lazy, mixed?
   - **Visibility** — What is visible where? Scoping rules, modules, privacy.
   - **Lifetime** — How long do values live? Stack, heap, arena, GC.

3. **Validate semantic model consistency:**
   - Check cross-dimension conflicts (e.g., ownership × mutation, evaluation × lifetime)
   - Check against all Design Principles
   - Write EDR accepting the Semantic Model

**New artifacts:**
- `what/SEMANTIC_MODEL.md` — unified semantic model
- EDR for Semantic Model acceptance

**Exit criteria:**
- [ ] All 6 semantic dimensions defined and internally consistent
- [ ] No contradictions with Design Principles
- [ ] Semantic Model EDR accepted

**Dependencies:** Phase 1 (Vision, Principles locked).

---

### Phase 3: Primitive Blocks ⬜

**Goal:** Identify the minimal orthogonal set of primitive building blocks.
Every derived feature must decompose into these. If a feature cannot be
decomposed, the primitive set is incomplete.

**Steps:**

1. **Analyse minimum viable set — starting hypothesis:**
   - `identifier` — named reference to a value
   - `literal` — inline value notation
   - `assignment` — bind value to name
   - `function` — parameterized computation
   - `call` — invoke a function
   - `attribute access` — access member of a composite value
   - `scope` — lexical boundary
   - `reference` — indirection to a value
   - `pack` — combine values into a composite
   - `unpack` — decompose a composite
   - `operator definition` — user-definable operator semantics

2. **Check for gaps:**
   - Each primitive must be orthogonal (no overlap, free composition)
   - Each primitive must be semantically justified (answers to Semantic Model)
   - Does any derived feature require a primitive not in the list?

3. **Create `PRIMITIVE_BLOCKS.md`:**
   - Full specification of each primitive
   - Composition rules (how primitives combine)
   - Verification: every concept research doc decomposes to primitives

4. **Write EDR accepting the Primitive Blocks set**

**New artifacts:**
- `what/PRIMITIVE_BLOCKS.md`
- EDR for Primitive Blocks acceptance

**Exit criteria:**
- [ ] All primitives are orthogonal (no overlap)
- [ ] Each primitive maps to Semantic Model
- [ ] Every known derived feature decomposes onto primitives
- [ ] Set is minimal — removing any primitive makes some feature inexpressible

**Dependencies:** Phase 2 (Semantic Model defines what primitives mean).

---

### Phase 4: Derived Features & Decision Pipeline ⬜

**Goal:** Design every language feature through a rigorous pipeline — each one
decomposed to primitives, run through the 10-question Decision Pipeline,
classified as language/stdlib/external, and accepted via EDR.

**Steps:**

1. **Define the Decision Pipeline:**
   ```
   1. What problem are we solving?
   2. Is this a language problem or a library problem?
   3. Can it be solved with existing primitives?
   4. Does it violate any Design Principle?
   5. Does it add new semantics (vs. syntactic sugar)?
   6. Can it be expressed through composition?
   7. Can it be syntactic sugar over existing primitives?
   8. Is this an optimisation, not semantics?
   9. Does it affect backward compatibility?
   10. Is it worth adding at all?
   ```

2. **Create `DECISION_PIPELINE.md`** — the canonical decision flow.

3. **Classify and design all concepts:**
   For each concept in `how/concepts/research/`:
   - Run through Decision Pipeline
   - Decompose to Primitive Blocks (from Phase 3)
   - Classify: **Language** | **Standard Library** | **External Library**
   - If "Language": produce full concept design per Concept Design Review
   - Write EDR accepting the concept
   - Move accepted draft to `what/concepts/`

4. **Concepts in scope:**
   | Priority | Concepts |
   |----------|----------|
   | **Core** | `FUNCTIONS.md`, `ERROR_HANDLING.md`, `GENERICS.md`, `PATTERN_MATCHING.md`, `OWNERSHIP.md` |
   | **Control** | `ASYNC_AWAIT.md`, `CONCURRENCY.md`, `GENERATORS.md` |
   | **Data** | `SORTING.md`, `UNPACKING.md`, `OBJECT_INITIALIZATION.md`, `SPAN.md` |
   | **Meta** | `METAOBJECTS.md`, `LITERATE_PROGRAMMING.md`, `LLM_NATIVE_TOOLCHAIN.md` |
   | **Anti-pattern** | 10 `imperative-crutch-*.md` files (inform design) |

5. **Create `LIBRARY_BOUNDARY.md`:**
   - For each feature classified as "Standard Library": document boundary rationale
   - For "External Library": note as explicitly out of scope for M1
   - Canonical reference for language vs. stdlib vs. external

> **LLM Toolchain components** (Schema Provider, Code Completer, etc.)
> go through a parallel track with lighter gates:
> `ARCHITECTURAL_INTEGRITY_GATE`, `USER_VALUE_GATE`, `LLM_GENERABILITY_GATE`.

> **Existing partial concept docs** (`FOUNDATIONAL_ABSTRACTIONS.md`, `DATA_MODEL.md`,
> `EQUALITY.md`, etc.) are considered **drafts / examples**. They will be
> formally reviewed through this Phase's process. A concept is
> registered only after acceptance via EDR (Architecture category).

> **Note on interactions:** This Phase only **identifies which concepts**
> each new concept interacts with. The detailed pair-wise interaction
> analysis is deferred to Phase 6 (Cross-Cutting), when the full set
> of accepted concepts is available.

**New artifacts:**
- `how/process/DECISION_PIPELINE.md` — the 10-question pipeline
- `what/LIBRARY_BOUNDARY.md` — language vs stdlib vs external classification
- Per accepted concept: `what/concepts/{NAME}.md` + EDR

**Exit criteria:**
- [ ] Decision Pipeline defined and applied to every feature
- [ ] Each feature decomposed to Primitive Blocks
- [ ] Each feature classified (Language / StdLib / External)
- [ ] All accepted concepts have EDRs
- [ ] Anti-pattern research completed and informs design

**Dependencies:** Phase 3 (Primitive Blocks must exist to decompose into).

---

### Phase 5: Syntax Design ⬜

**Goal:** Design syntax as the *external interface* of the semantic model —
not as an independent creative exercise. Syntax is derived from semantics,
not the reverse.

**Steps:**

1. **Establish syntax principles:**
   - One concept → one syntax
   - One symbol → one meaning (context-independent)
   - No significant whitespace
   - Named forms preferred over symbolic (when ambiguity risk)
   - Syntax is derived from semantics, not vice versa

2. **Produce syntax for each accepted Language concept:**
   - All canonical forms documented together (per AGENTS.md §4.2)
   - Syntax must be consistent across all concepts

3. **Validate syntax:**
   - No ambiguous parses
   - No context-dependent syntax
   - LLM generability check
   - Readability check — can a human predict the syntax from semantics alone?

4. **Create `SYNTAX.md` and update `PARSER.md`**

**Primary gates applied:** `ARCHITECTURAL_INTEGRITY_GATE`,
`LOGICAL_CONSISTENCY_GATE`.

**New artifacts:**
- `what/SYNTAX.md` — complete syntax reference
- `how/architecture/PARSER.md` — updated with concrete grammar

**Exit criteria:**
- [ ] Every language concept has defined syntax
- [ ] No ambiguous or context-dependent syntax
- [ ] LLM generability gate passed for all syntax
- [ ] Syntax consistent across all concepts

**Dependencies:** Phase 4 (concepts must be accepted before syntax is designed).

---

### Phase 6: Cross-Cutting Review ⬜

**Goal:** Verify that all accepted concepts work together without conflicts.
Produce the Interaction Matrix and resolve all concept-boundary conflicts.

**Steps:**

1. **Build Interaction Matrix — key pairs to investigate:**

   | Pair | Risk |
   |------|------|
   | Generator × Error Handling | Resource lifetime across suspension points |
   | Pattern Matching × Types | Exhaustiveness with nominal vs structural |
   | Modules × Visibility | Cross-module access control |
   | Blocks × Closures | Variable capture and scope boundaries |
   | Async/Await × Error Handling | Exception propagation across async |
   | Ownership × Mutation | Shared mutation semantics |
   | Generics × Pattern Matching | Type parameter destructuring |

2. **Create `CROSS_CUTTING.md`** — full pair-wise interaction matrix

3. **Create `CONFLICT_REGISTRY.md`** — open conflicts and resolutions

4. **Resolve all conflicts before Phase 8**

**Primary gates applied:** `ARCHITECTURAL_INTEGRITY_GATE`,
`LOGICAL_CONSISTENCY_GATE`.

**New artifacts:**
- `what/CROSS_CUTTING.md` — interaction matrix
- `what/CONFLICT_REGISTRY.md` — conflicts and resolutions

**Exit criteria:**
- [ ] All concept pairs analysed
- [ ] All conflicts resolved or documented with EDR deferral rationale
- [ ] No unresolvable contradictions

**Dependencies:** Phase 4 (all concepts designed).

> **Input from Phase 4:** Each concept's Interactions section identifies which
> other concepts it interacts with. This Phase receives that list of affected
> pairs and performs the full pair-wise analysis.

> **Note on format:** The exact format of the interaction matrix requires
> separate investigation (see `docs/notes/interaction-matrix-format.md`).

---

### Phase 7: Execution & Optimization Model ⬜

**Goal:** Define the boundary between language semantics and implementation.
Document what is guaranteed by the language vs. what is an optimisation choice.

**Steps:**

1. **Create `EXECUTION_MODEL.md`:**
   - Define execution semantics — what the language guarantees about *how*
     a program executes
   - Execution Program as the semantics/execution boundary (see
     [`EXECUTION_PROGRAM.md`](../how/concepts/research/EXECUTION_PROGRAM.md))
   - Supported execution targets: Interpreter, AOT, JIT, LLVM, WASM, Container

2. **Create `OPTIMIZATION_MODEL.md`:**
   - Catalogue optimisations by category:
     | Category | Examples | Classification |
     |----------|----------|----------------|
     | Inlining | Function inlining | Optimisation |
     | Constant folding | Compile-time evaluation | Optimisation |
     | Escape analysis | Stack vs heap allocation | Optimisation |
     | Dead code elimination | Unused code removal | Optimisation |
     | Evaluation order | Expression sequencing | **Semantics** |
     | Memory layout | Struct field ordering | Depends on strategy |

3. **Reconcile with Implementation Strategies:**
   - Ensure `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`,
     `HIGH_PERFORMANCE_STRATEGY.md`, `LLM_STRATEGY.md` are consistent
   - The "Semantics Before Optimization" principle now has a concrete reference

**Primary gates applied:** `IMPLEMENTATION_INDEPENDENCE_GATE`,
`LONG_TERM_MAINTAINABILITY_GATE`.

**New artifacts:**
- `what/EXECUTION_MODEL.md`
- `what/OPTIMIZATION_MODEL.md`

**Exit criteria:**
- [ ] Semantics vs. Optimisation boundary is explicit and unambiguous
- [ ] Every optimisation is classified
- [ ] All Implementation Strategies consistent with the model
- [ ] Execution Program concept formalised via EDR

**Dependencies:** Phase 6 (stable concept set), Phase 2 (Semantic Model).

---

### Phase 8: Evolution Model & Freeze ⬜

**Goal:** Define how the language evolves over time, then freeze v0.1.

**Step 8.1 — Evolution Model:**

- Versioning scheme (SemVer for language? Date-based?)
- Deprecation policy — how features are deprecated, removal timeline
- Experimental mechanism — opt-in, feature gates
- Breaking change policy — what constitutes breaking, migration path requirement
- Create `how/EVOLUTION_MODEL.md`

**Step 8.2 — Consistency Review:**
Review the language as a whole for uniformity, minimality, and absence of
special cases.

- Syntax consistency — similar constructs look similar
- No special cases — context-dependent behaviour is eliminated
- Naming uniformity — consistent naming conventions
- Operator uniformity — consistent operator semantics
- **Global minimality** — no concept duplicates another's responsibility.
  This is distinct from Phase 4's *internal* minimality (is each individual
  concept lean?). This checks *global* minimality — does the language as a
  whole have redundant, overlapping concepts?
  > *This is where approximately half of "interesting" features are removed.*

**Primary gates applied:** `CONCEPTUAL_SIMPLICITY_GATE`,
`LONG_TERM_MAINTAINABILITY_GATE`.

**Step 8.3 — Language Specification:**
Produce the canonical specification. No alternatives or rationale — it fixes
the language. Each section contains:
- Description • Syntax • Semantics • Constraints • Examples • Links to EDRs

**Deliverables:**
- `SPEC.md` — canonical language specification
- `SCHEMA.md` — machine-readable language schema (grammar, types, contracts)
  consumable by the LLM Toolchain

**Step 8.4 — Validation:**
Test the language design in practice before freezing.

- Write reference examples and real programs
- Implement known algorithms to find ergonomic issues
- **LLM generation tests** — validate that LLMs can generate correct Orthon
  code for each concept
- **Schema round-trip tests** — verify schema → generation → validation cycle
- Document ambiguities and friction points
- Produce new EDRs for issues discovered

**Step 8.5 — Freeze:**
Freeze the first version of the language specification.

- All validation issues resolved or deferred with documented EDR rationale
- A documented decision (EDR) confirms the final project name or alternative
- Specification is self-consistent and complete
- Version tagged (Orthon v0.1 Specification)
- Full link audit — zero dangling cross-references across the entire repository

**New artifacts:**
- `how/EVOLUTION_MODEL.md` — versioning, deprecation, experimental, feature gates
- `SPEC.md` — canonical frozen specification
- `SCHEMA.md` — machine-readable schema
- `docs/notes/whitepaper-strategy.md` — whitepaper plan finalised

**Exit criteria:**
- [ ] Evolution model documented and approved via EDR
- [ ] Global consistency review passed
- [ ] Validation programs written and reviewed
- [ ] LLM generation tests passed
- [ ] Zero dangling cross-references
- [ ] Specification frozen and tagged

**Dependencies:** Phase 7 (Execution Model part of frozen spec),
Phase 6 (cross-cutting issues resolved), Phase 5 (syntax resolved),
Phase 4 (concepts designed).

---

## M2 — Standard Library & FFI (post-Freeze) ⬜

**Goal:** Define the standard library architecture and foreign-function
interface.

| # | Item | Description |
|---|------|-------------|
| 8.1 | Standard Library | Collections, I/O, formatting, etc. |
| 8.2 | FFI & Interoperability | C ABI, embedding API |
| 8.3 | LLM Toolchain | Schema Provider, Code Completer, Code Generator, Static Analyser implementation |

**Dependencies:** M1 (stdlib and FFI depend on the complete language
specification).

---

## M3 — Build System & Tooling (post-Freeze) ⬜

**Goal:** Design the developer toolchain.

| # | Item | Description |
|---|------|-------------|
| 9.1 | Build System | Build system & package manager design |
| 9.2 | Tooling | Formatter, linter, LSP (Language Server Protocol) |

**Dependencies:** M1 (tooling depends on the frozen specification).

---

## M4 — Implementation (post-Freeze) ⬜

**Goal:** Build the compiler / runtime (separate repository).

| # | Item | Description |
|---|------|-------------|
| 10.1 | Compiler | Compiler implementation |
| 10.2 | Runtime | Runtime library |
| 10.3 | Whitepapers | Publish launch whitepaper(s) alongside initial release |

**Dependencies:** M1–M3 (implementation should not begin until the language
is sufficiently specified and tooling is designed).

---

## Reference Documents

The following documents are **project reference materials**, available
throughout all milestones. They are not deliverables of any specific
milestone.

| Document | Purpose |
|----------|---------|
| `ARCHITECTURE.md` | Layered architecture definition |
| `FITNESS_FUNCTIONS.md` | Architectural fitness functions — measurable checks guarding against design decay |
| `whitepaper-strategy.md` (notes) | Whitepaper publication plan — timing, topics, and audience per milestone |
| `IMPLEMENTATION_STRATEGIES.md` | Strategy model: Default, Embedded, High-Performance |
| `DEFAULT_STRATEGY.md` | Default implementation strategy profile |
| `EMBEDDED_STRATEGY.md` | Embedded implementation strategy profile |
| `HIGH_PERFORMANCE_STRATEGY.md` | High-performance strategy profile |
| `LLM_STRATEGY.md` | LLM strategy profile — maximum LLM assistance during development |
| `PROCESS_INVENTORY.md` | Process-lens catalogue of all tools, methods, and approaches |

> Concept-specific architecture documents (Parser, Type System, Name
> Resolution, IR) are initial stubs and will be designed through
> the Concept Design Review process (Epic 3).

---

## Guiding Principles

- **One milestone at a time.** Each milestone should be substantially
  complete before the next begins (unless explicitly scoped otherwise).
- **Design before implement.** M1 (language design) precedes M2–M4 (stdlib,
  tooling, implementation).
- **ADRs as you go (EDRs).** Engineering Decision Records are created alongside
  concept design decisions in Phase 4, not deferred.
- **Docs drive, code follows.** This repository contains *only* design
  documentation. The compiler and runtime live in a separate repository.
