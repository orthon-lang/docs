# Phase 2: Semantic Model — Context

**Gathered:** 2026-07-27 (assumptions mode, interactive)
**Status:** Ready for planning

<domain>
## Phase Boundary

Define the semantic foundation of Orthon — what a program *means* at the foundational level. Six dimensions: Identity, Ownership, Mutation, Evaluation, Visibility, Lifetime.

Each dimension is defined as a language-semantic contract, independent of:
- Syntax (Phase 5) — how programs are written
- Primitive Blocks (Phase 3) — what constructs exist
- Derived Features (Phase 4) — what features are built on top
- Implementation Strategy (Phase 7) — how semantics are implemented
</domain>

<decisions>
## Implementation Decisions

### D-01: Identity Model
- **Decision:** Value semantics by default. Assignment copies structurally; `==` compares structurally.
- **Identity is not universal** — only exists for entities representing shared state or external resources.
- **Ownership is orthogonal to Identity** — copy/move/borrow is a separate concept.
- **Implementation freedom:** compiler may use CoW, copy elision, SSA, NRVO, register promotion — as long as observable behavior remains as if values are independent.
- **Contract:** Value semantics is a language contract, not a promise of immediate memory copying.
- **Source:** DATA_MODEL.md, VALUE_SEMANTICS.md

### D-02: Ownership Model
- **Decision:** Ownership applies only where exclusive responsibility exists (not only external resources — any case of exclusive accountability).
- **95% of code doesn't think about ownership** — ordinary values (`Int`, `String`, `Point`, `List`, `Map`, etc.) use pure value semantics.
- **For resources:** Rust-like model — single owner, move semantics, borrowing (`&T` shared read, `&mut T` exclusive write), lifetime checking.
- **Move syntax** — deferred to Phase 5 (`move` / `$` / `@ownership`).
- **`delegate`** takes ownership (isolation through mailbox). **`release`** returns ownership from delegate.
- **Fresh values** don't need explicit `move` — compiler applies move automatically.
- **No GC**, no reference counting by default. Deterministic destruction.
- **Source:** OWNERSHIP.md, OWNERSHIP_METAPROPERTY.md, OWNERSHIP_TRANSFER_OPERATOR.md, IDENTITY_BASED_SAFETY.md (rejected for Phase 2)

### D-03: Mutation Model
- **Decision:** Immutable by default. Mutation is explicit.
- **Four principles:**
  1. Immutable by default — all bindings immutable unless `val`/`var`/`mut` specified
  2. Explicit mutation — mutation must be syntactically visible
  3. Aliasing control — compiler tracks whether a value is mutated through multiple references
  4. No hidden mutation — no implicit mutation via method calls, property setters, or operator overloading
- **Function contract defines mutation** — three declaration kinds (`fun`/`proc`/`new`):
  - `fun` — pure, read-only, returns value
  - `proc` — mutates `self`, identity preserved, may return value or nothing
  - `new` — changes identity, creates new object, original unchanged
- **No `mut` on calling site** — contract is in the declaration kind.
- **Binding keywords:**
  - `val x = 42` — immutable binding (explicit keyword)
  - `var x = 42` — mutable binding
  - `x = 42` — declaration by assignment in local context, creates immutable binding. Compiler may assist.
  - `const PI = 3.14` — compile-time constant, separate concept
- **Open questions (deferred to Phase 3):** Interior mutability (`Cell`/`RefCell`)? Mutation in closures? `mut` vs `&mut` — one keyword or two?
- **Source:** MUTABILITY.md, EXCLUSIVE_DECLARATIONS.md, IDENTITY_BASED_SAFETY.md (rejected approach), DECLARATION_BY_ASSIGNMENT.md

### D-04: Evaluation Model
- **Decision:** Eager by default. Laziness must be explicit.
- **Eager:** Expressions are evaluated immediately. Function arguments are evaluated before call.
- **Lazy marker:** Explicit `sec` or similar keyword for deferred computation.
- **Expression-oriented:** All control flow constructs produce values. No statement/expression split.
  - `if`, `when`, `try`, blocks — all return values
  - Last expression in block is the block's value
  - No ternary operator needed (`if` is already an expression)
- **Sequence production:** `emit` chosen over `yield`. `yield` rejected (too many responsibilities in Python).
- **Source:** EXPRESSION_ORIENTED_LANGUAGE.md, DATA_MODEL.md, ITERATOR_PROTOCOL.md, LAZY_SEQUENCE_GENERATORS.md

### D-05: Visibility Model
- **Decision:** Private by default (type-level). Module-scope is the default scope. `pub` is mandatory for export.
- **No `protected`** — replaced by sealed types / open modules (future concern).
- **No backdoors** — visibility is compile-time guaranteed. No reflection bypass, no name mangling tricks.
- **Levels:**
  - `priv` — visible only within the containing type
  - *(no keyword)* — visible within the module
  - `pub` — exported to all importing modules
- **Open question (Phase 5):** Does `pub` on a type imply `pub` on its members, or do members stay `priv` by default?
- **Source:** VISIBILITY_AND_ENCAPSULATION.md

### D-06: Lifetime Model
- **Decision:** Scope-based lifetime. Deterministic destruction. No GC.
- **Scope-based:** Values live until the end of their enclosing scope (`{}` block, function body). Exiting scope triggers deterministic destruction.
- **No GC** — garbage collection is not part of the semantic model.
- **Value semantics:** Copies are independent and live their own lifetime.
- **Implementation freedom:** Regions, arenas, stack, heap — these are Strategy/Policy decisions (Phase 7), not semantics.
- **Source:** SCOPED_RESOURCE_LIFECYCLE.md, OWNERSHIP.md
</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

- `how/concepts/research/essential/DATA_MODEL.md`
- `how/concepts/research/essential/OWNERSHIP.md`
- `how/concepts/research/essential/OWNERSHIP_METAPROPERTY.md`
- `how/concepts/research/essential/OWNERSHIP_TRANSFER_OPERATOR.md`
- `how/concepts/research/essential/MUTABILITY.md`
- `how/concepts/research/essential/VALUE_SEMANTICS.md`
- `how/concepts/research/essential/IDENTITY_BASED_SAFETY.md`
- `how/concepts/research/essential/VISIBILITY_AND_ENCAPSULATION.md`
- `how/concepts/research/essential/SCOPED_RESOURCE_LIFECYCLE.md`
- `how/concepts/research/essential/EXPRESSION_ORIENTED_LANGUAGE.md`
- `how/concepts/research/essential/EXCLUSIVE_DECLARATIONS.md`
- `how/concepts/research/important/DECLARATION_BY_ASSIGNMENT.md`
- `what/SEMANTIC_MODEL.md` — current DRAFT placeholder (target document)
- `how/DESIGN_PRINCIPLES.md` — locked constitution, all decisions verified against it
- `how/process/DECISION_PROCESS.md` — decision authority
- `how/process/DECISION_PIPELINE.md` — 10-question feature pipeline
- `how/concept-design-review.md` — 5-step design procedure
- `how/gates/DECISION_VALIDATION.md` — 7 validation gates
- `.planning/REQUIREMENTS.md` § Phase 2 requirements (SEM-01, SEM-02, SEM-03)
- `.planning/notes/2026-07-26-tier-vs-phase-mapping.md`
</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `what/SEMANTIC_MODEL.md` — exists as DRAFT placeholder with 6 empty sections; target document to fill
- 10 essential-tier research files — raw material for synthesis (listed above)
- All research files follow the `_concept.md` template structure (Issue/Why → Principles → Policy Footprint → Model → Default Strategy → Alternatives)

### Established Patterns
- Concept Design Review: 5-step procedure (Idea/Problem → Minimal Solution → Principle Check → Examples → EDR)
- Every consequential decision requires EDR (Architecture category for Principle/Tier 1)
- DESIGN_PRINCIPLES.md is locked — any deviation requires Tier 1 EDR

### Integration Points
- Phase 2 output feeds directly into Phase 3 (Primitive Blocks) — primitives decompose from semantic model
- Phase 4 (Derived Features) depends on both Phase 2 and Phase 3
- Phase 5 (Syntax Design) derives syntax from semantic model
</code_context>

<specifics>
## Specific Ideas

- **Fun/proc/new** as exclusive declaration kinds for mutation model (from EXCLUSIVE_DECLARATIONS.md)
- **Val/var** for bindings (Kotlin/Swift style) with declaration-by-assignment for local context
- **Emit** over yield for sequence production
- **Sec** (or similar) as explicit lazy marker
- Ownership syntax (`move`/`$`/`@ownership`) deferred to Phase 5
</specifics>

<deferred>
## Deferred Ideas

- Region/arena allocation model — deferred to Phase 7 (Implementation Strategy)
- Interior mutability (`Cell`/`RefCell`) — deferred to Phase 3 (Primitive Blocks)
- Mutation in closures — deferred to Phase 3
- Full Identity-Based Safety model (`.` vs `!`) — rejected for Phase 2; violates Explicitness principle
- `yield` keyword — rejected; `emit` chosen instead
</deferred>
