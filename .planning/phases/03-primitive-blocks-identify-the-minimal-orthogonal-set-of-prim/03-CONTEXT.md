# Phase 3: Primitive Blocks — Context

**Gathered:** 2026-07-27 (assumptions mode)
**Status:** Ready for planning

<domain>
## Phase Boundary

Identify the minimal orthogonal set of primitive building blocks in `what/PRIMITIVE_BLOCKS.md`, replacing its current DRAFT placeholder. Every derived feature designed in Phase 4 must decompose into these; if a feature cannot be decomposed, the primitive set is incomplete and must be revisited before Phase 4 proceeds.

Primitives are the irreducible atomic operations from which all language constructs are composed. They are defined in terms of the Semantic Model (Phase 2) — each primitive serves one or more semantic dimensions (Identity, Ownership, Mutation, Evaluation, Visibility, Lifetime) and must be orthogonal (no primitive overlaps another's responsibility).

Covers requirements: PRIM-01, PRIM-02, PRIM-03.
</domain>

<decisions>
## Implementation Decisions

### D-01: Primitive Set Scope and Granularity
- **Decision:** The final primitive set is approximately 8-11 primitives — close to the existing 11-item starting hypothesis in `what/PRIMITIVE_BLOCKS.md`, but with key reorganizations:
  - `pack` and `unpack` are a symmetric pair under a single `composition` / `decomposition` concept (construction and destruction follow the same syntax — `UNPACKING.md` symmetry principle)
  - `function` (declaration construct) and `call` (evaluation construct) are separate primitives — they serve different semantic dimensions
  - `operator definition` is NOT a primitive — it is syntactic sugar for function definition with a symbolic name (per `DESIGN_PRINCIPLES.md` § Named Before Symbolic)
  - `delegate` and `namespace` are NOT primitives — they decompose into `reference` + `function` + ownership, and `identifier` + `scope` + visibility respectively
- **Rationale:** The set must be granular enough that Phase 4 concepts can be meaningfully decomposed (too coarse → cannot verify completeness) but not so granular that primitives overlap (too many → non-orthogonality revealed during verification)
- **Starting hypothesis confirmed from:** `what/PRIMITIVE_BLOCKS.md` lines 20-42; `FOUNDATIONAL_ABSTRACTIONS.md` two-abstraction model as organizing framework
- **Source:** PRIMITIVE_BLOCKS.md (starting hypothesis), FOUNDATIONAL_ABSTRACTIONS.md, FUNCTIONS.md, DESIGN_PRINCIPLES.md § Named Before Symbolic

### D-02: Data and Data Modifiers as the Organizing Framework
- **Decision:** The Data/Data Modifiers hypothesis from `FOUNDATIONAL_ABSTRACTIONS.md` serves as the *conceptual organizing framework* for the primitive set, but NOT as the primitive set itself. Primitives are categorized into "Data primitives" and "Data Modifier primitives" rather than being replaced by just those two meta-abstractions.
- **Rationale:** Two abstractions alone are too coarse-grained for Phase 4 decomposition verification (a `for` loop and a function call would both be "Data Modifiers," losing the granularity needed). The taxonomy helps verify completeness (is there a third category missing?) and guides Phase 4 classification of whether new concepts are primitives or derivatives.
- **Source:** FOUNDATIONAL_ABSTRACTIONS.md lines 39-48 (hypothesis requiring validation), SEMANTIC_MODEL.md lines 471-476 (confirmed influence on accepted design), GLOSSARY.md lines 449-456 (Primitive Operation definition)

### D-03: Type Constructors Are Not Primitives
- **Decision:** `struct` and `class` are NOT primitives. They are type-level constructs built from composition of simpler primitives:
  - `pack` (composite data construction) underlies struct field composition
  - `reference` (indirection) underlies class identity semantics
  - `scope` underlies class/struct lifetime
- **Rationale:** Including type constructor keywords alongside atomic operations mixes abstraction levels. `STRUCT_AS_VALUE_TYPE.md` establishes "structs are data, not behaviour." `CLASS_WITH_ACT.md` describes class as reference type built on indirection. The Semantic Model's Identity section confirms reference (indirection) is the underlying mechanism — the class keyword is a convenience, not a primitive.
- **Source:** STRUCT_AS_VALUE_TYPE.md, CLASS_WITH_ACT.md, SEMANTIC_MODEL.md § Identity

### D-04: Function Model Primitives
- **Decision:** Two primitives cover the function domain:
  1. `function` — the declaration/definition construct (first-class, named or anonymous, with explicit closure capture)
  2. `call` — the invocation/evaluation mechanism (unified syntax regardless of declaration form)
- **The three declaration kinds** (`fun`/`proc`/`new`) are *tags on the function primitive*, not separate primitives. They modify how the function interacts with its context (pure, mutating, transforming).
- **Rationale:** Separating declaration from invocation is conceptually clean: declaration addresses *what* (construct definition), invocation addresses *how* (evaluation trigger). The three-kind system from `EXCLUSIVE_DECLARATIONS.md` is a function-level annotation, not an orthogonal primitive dimension.
- **Source:** FUNCTIONS.md (first-class, uniform call syntax), EXCLUSIVE_DECLARATIONS.md (fun/proc/new), SEMANTIC_MODEL.md D-03/D-04

### D-05: Key Exclusions from Primitive Set
- **`operator definition`** — syntactic sugar for function definition with a symbolic name (DESIGN_PRINCIPLES.md § Named Before Symbolic). The actual primitive is `function`.
- **`delegate`** — execution policy composing `reference` + `function` + ownership semantics (DELEGATE.md: "Execution is orthogonal to declaration"). Not an atomic operation.
- **`namespace`** — organizational convenience decomposing to `identifier` + `scope` + visibility (NAMESPACES.md). Not irreducible.
- **`struct` / `class`** — type constructors built from `pack` + `reference` + `scope` (see D-03).
- **Rationale:** Each excluded item is either (a) syntactic sugar over a real primitive, (b) a composition of multiple primitives, or (c) a meta-language feature (syntax extension). Including any would violate orthogonality or mix abstraction levels.

### D-06: `emit` is Lazy by Default (Correction to Phase 2 D-04)
- **Correction to prior assumption:** `emit` is **lazy by default** — it produces values on demand, not eagerly. For eager sequence production, use `return` with an aggregate collection.
- **Rationale (user clarification):** Lazy `emit` aligns with Sequence as a description of *what*, not *how*. Eager production is better served by constructing a collection and returning it — the distinction is explicit in the choice of mechanism.
- **Consequence:** The Evaluation Policy for `emit` is settled as lazy. Phase 4 concepts (iterators, generators) are built on lazy `emit`.

### D-07: `@` for Metadata Access (Metadata Protocol)
- **Decision:** All metadata, protocol methods, and special operations are accessed via the `@` prefix notation, NOT via double-underscore conventions (`__len__`, `__getitem__`, etc.) or hidden method names. This is called the **Metadata Protocol**.
  - `list@len()` instead of `list.__len__()` or Python-style dunder methods
  - `obj@fields`, `type@name`, etc. for reflective/structural access
- **Rationale:** `@` makes metadata access syntactically visible and distinct from regular attribute access (`.`). This is more LLM-generable (no memorisation of which methods are "special") and more explicit for human readers. The `@` prefix is a single, consistent marker for "this is a language-level operation on the type/object, not a user-defined method."
- **Consequence:** System functions like `len()`, `sorted()`, `str()` are mapped to `@`-prefixed protocol methods (Metadata Protocol). Free functions (`len(obj)`) may exist as syntactic sugar that compiles to `obj@len()` — this is a Phase 4/5 decision.

### D-08: Free Functions Need Special Design Conditions
- **Decision:** Free functions like `len(list)`, `sorted(iter)`, `str(obj)` are acknowledged as an open design problem, not assumed to work automatically. They require:
  1. A protocol system (`@`-prefixed methods) — see D-07
  2. A resolution mechanism that maps free function calls to protocol methods
  3. Clear boundary between language-provided functions and user-defined ones
- **Status:** Open design question — deferred to Phase 4/5 (Derived Features/Syntax), but the `@` protocol convention is locked now (D-07).

### D-09: Blocks `{ }` Require Explicit `return`
- **Decision:** Blocks delimited by `{ }` require an explicit `return` keyword to produce a value. The "last expression is the block's value" rule (Phase 2 D-04's expression-oriented model) applies only to expression-level constructs like `if`, `match`, `when` — NOT to `{ }` block syntax.
  ```orthon
  let x = {
      let tmp = compute()
      return tmp + 1   // explicit return required in { } blocks
  }
  ```
- **Rationale:** Distinguishes block-as-scope (where side effects happen) from block-as-expression (where the last expression is the value). `{ }` blocks are primarily for scoping and sequencing; making return explicit avoids ambiguity when a block contains multiple statements and only one is intended as the result.

### D-10: Deferred to Phase 3 — Open Items Carried from Phase 2
- **Interior mutability (`Cell`/`RefCell`):** To be addressed as a primitive-block level question — whether interior mutability is a primitive concept or a derived feature built on `reference` + mutation semantics.
- **Mutation in closures:** Whether closures capture variables as immutable by default (matching D-03 immutability principle) or if explicit `mut` capture is needed.
- **`mut` vs `&mut`:** Whether one keyword or two — a primitive-block level syntactic distinction that must be resolved before Phase 4 features can use it.

### The Agent's Discretion
- Exact naming and ordering of the candidate primitives in `what/PRIMITIVE_BLOCKS.md`.
- Whether `composition` (pack/unpack as symmetric pair) is listed as one primitive with two operations, or two primitives documented together.
- Whether `reference` includes both shared (`&T`) and exclusive (`&mut T`) forms as one primitive or two.

### Folded Todos
None.
</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

- `what/PRIMITIVE_BLOCKS.md` — target document (current DRAFT placeholder with 11-primitive starting hypothesis)
- `what/SEMANTIC_MODEL.md` — accepted semantic foundation (EDR-013); primitives decompose from these dimensions
- `how/concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md` — Data/Data Modifiers organizing framework
- `how/concepts/research/essential/EXCLUSIVE_DECLARATIONS.md` — fun/proc/new declaration kind system
- `how/concepts/research/essential/STRUCT_AS_VALUE_TYPE.md` — struct as default value type
- `how/concepts/research/essential/CLASS_WITH_ACT.md` — class with concurrent isolation via act modifier
- `how/concepts/research/essential/ACT_AS_FUNCTION.md` — SUPERSEDED by DELEGATE.md; historical context
- `how/concepts/research/essential/FUNCTIONS.md` — first-class function model, uniform call syntax, explicit closure capture
- `how/concepts/research/essential/FINAL_BY_DEFAULT.md` — closed-by-default, open opt-in for inheritance
- `how/concepts/research/essential/NAMESPACES.md` — logical namespace declarations
- `how/concepts/research/essential/DELEGATE.md` — execution policy, not a primitive
- `how/concepts/research/essential/COMPOSITION_OVER_INHERITANCE.md` — composition-only, structural interface satisfaction
- `how/DESIGN_PRINCIPLES.md` — locked constitution, especially § Named Before Symbolic
- `what/GLOSSARY.md` § Primitive Operation — definition of what constitutes a primitive
- `how/concepts/research/important/UNPACKING.md` — symmetry principle for pack/unpack
- `.planning/notes/2026-07-26-tier-vs-phase-mapping.md` — phase/tier split classification
- `.planning/REQUIREMENTS.md` § Phase 3 requirements (PRIM-01, PRIM-02, PRIM-03)
- `.planning/ROADMAP.md` § Phase 3 — success criteria and escalation clause
</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `what/PRIMITIVE_BLOCKS.md` — exists as DRAFT placeholder with full 11-item starting hypothesis table and semantic justifications; ready to be filled with validated set
- 10 essential-tier research files — raw material for synthesis (listed in canonical refs above)
- `what/SEMANTIC_MODEL.md` — fully populated 6-dimension model (EDR-013), primitives decompose from these dimensions
- All research files follow the `_concept.md` 8-section template structure

### Established Patterns
- Concept Design Review: 5-step procedure (Idea/Problem → Minimal Solution → Principle Check → Examples → EDR)
- Every consequential decision requires EDR (Architecture category for Primitive Blocks set — EDR-NNN)
- DESIGN_PRINCIPLES.md is locked — any deviation requires Tier 1 EDR
- SEMANTIC_MODEL.md is accepted (EDR-013) — its 6 invariants and 6 dimensions constrain the primitive set

### Integration Points
- Phase 3 output feeds directly into Phase 4 (Derived Features & Decision Pipeline) — every concept must decompose onto primitives
- Phase 4's PRIM-02 verification (~132 files across all tiers) validates that the primitive set is minimal and complete
- Phase 5 (Syntax Design) derives concrete syntax from primitives
- Phase 2's six Semantic Dimensions constrain which primitives are valid (a primitive must serve at least one dimension)
</code_context>

<specifics>
## Specific Ideas

- The existing 11-primitive hypothesis (identifier, literal, assignment, function, call, attribute access, scope, reference, pack, unpack, operator definition) is a strong starting point — the validated set will be similar with key subtractions (operator definition out, delegate out, namespace out) and reorganizations (pack/unpack as symmetric composition primitive)
- Data/Data Modifiers as *organizing taxonomy* rather than *primitive set* — primitives categorized as "Data primitives" (literal, pack, identifier) vs "Data Modifier primitives" (assignment, function, call, attribute access, reference, scope, unpack/operator definition as derivatives)
- The three-kind declaration system (fun/proc/new) is a function-level annotation, not a new primitive
- Deferred open items from Phase 2 (interior mutability, closure mutation, mut vs &mut) must be settled before or during Phase 3 — they affect what primitives are needed
</specifics>

<deferred>
## Deferred Ideas

None — analysis stayed within phase scope.

### Reviewed Todos (not folded)
None.
</deferred>
