# Phase 2: Semantic Model — Plan

**Phase:** 2 of 9
**Goal:** Define identity, ownership, mutation, evaluation, visibility, and lifetime as one unified semantic model
**Status:** Ready for execution
**Total tasks:** 14
**Waves:** 4

---

## Wave 1: Foundations (parallelizable)

### Task 1.1 — Fix cross-references in SEMANTIC_MODEL.md scaffold
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Current cross-references point to `../how/concepts/research/DATA_MODEL.md` (no tier prefix). Fix all 7 cross-references to use the correct tiered paths (e.g., `../how/concepts/research/essential/DATA_MODEL.md`).
**Requirement:** SEM-01
**Estimate:** 5 min

### Task 1.2 — Write Semantic Invariants section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Add a new "Semantic Invariants" section at the top with 6 cross-cutting rules that apply to all dimensions:
1. Every value has exactly one owner at any point in the program.
2. Mutation requires exclusive access; read access may be shared.
3. Every value has a well-defined lifetime tied to its scope.
4. All control flow produces a value (expression-oriented).
5. Visibility is a compile-time guarantee with no runtime bypass.
6. Ownership transfer is semantically explicit (syntax TBD in Phase 5).
**Requirement:** SEM-01
**Estimate:** 20 min

### Task 1.3 — Write Identity section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Define identity semantics per D-01:
- Value semantics by default: structural equality, copy on assignment
- Identity is not universal — only for shared state/external resources
- Binding identity vs value identity distinction
- Identity is orthogonal to Ownership
- Fresh-value exemption: unbound temporaries have transient identity
- Reference semantics via explicit opt-in (shared types)
**Source:** `how/concepts/research/essential/DATA_MODEL.md`, `VALUE_SEMANTICS.md`, `IDENTITY_BASED_SAFETY.md`
**Requirement:** SEM-01
**Estimate:** 45 min

### Task 1.4 — Write Visibility section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Define visibility semantics per D-05:
- Three levels: `priv` (type/function-local), default (module-scoped), `pub` (exported)
- Module is encapsulation boundary
- No `protected`, no backdoors
- Compile-time enforcement only
- `pub` on type does not imply `pub` on members (open question for Phase 5)
- Visibility and Ownership are orthogonal — note the `priv` type + `pub` function signature edge case
**Source:** `how/concepts/research/essential/VISIBILITY_AND_ENCAPSULATION.md`
**Requirement:** SEM-01
**Estimate:** 30 min

### Task 1.5 — Write Lifetime section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Define lifetime semantics per D-06:
- Scope-based: value lifetime = enclosing scope
- Deterministic destruction at scope exit
- Reference lifetime ≤ referent lifetime
- Regions/arenas permitted as implementation strategy (Phase 7)
- No GC by default; opt-in GC/RC strategies are Phase 7
- Value semantics: copies are independent
**Source:** `how/concepts/research/essential/SCOPED_RESOURCE_LIFECYCLE.md`, `OWNERSHIP.md`
**Requirement:** SEM-01
**Estimate:** 30 min

---

## Wave 2: Core Dimensions (depends on Wave 1 invariants)

### Task 2.1 — Write Ownership section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Define ownership semantics per D-02:
- Single owner per value (invariant #1)
- Move semantics transfer ownership; source binding invalidated
- Borrowing creates temporary access without ownership transfer
- Aliasing constraints: shared XOR mutable
- No GC, no RC by default
- Fresh-value exemption: temporaries implicitly transferable
- Semantic invariant only — do NOT prescribe enforcement mechanism (borrow checker vs escape analysis vs other)
- Ownership applies only where exclusive responsibility exists
**Source:** `how/concepts/research/essential/OWNERSHIP.md`, `OWNERSHIP_METAPROPERTY.md`, `OWNERSHIP_TRANSFER_OPERATOR.md`
**Requirement:** SEM-01
**Estimate:** 60 min

### Task 2.2 — Write Mutation section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Define mutation semantics per D-03:
- Three-way semantic distinction: read-only / mutating / transforming (maps to `fun`/`proc`/`new` in syntax — Phase 5)
- Immutable by default for bindings (`val` vs `var`)
- No hidden mutation
- Aliasing control: compiler tracks mutation through multiple references
- Semantic invariant: mutation requires exclusive access
- No `mut` on calling site — contract is in the declaration kind
- Deferred to Phase 3: interior mutability, mutation in closures
**Source:** `how/concepts/research/essential/MUTABILITY.md`, `EXCLUSIVE_DECLARATIONS.md`
**Requirement:** SEM-01
**Estimate:** 45 min

### Task 2.3 — Write Evaluation section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Define evaluation semantics per D-04:
- Expression-oriented: all control flow produces values
- Eager by default; lazy explicit via `sec` (or equivalent marker)
- Defined evaluation order (left-to-right for sub-expressions)
- Side-effect visibility rules within expressions
- No statement/expression grammar split
- Explicit discard for unused expression values (`_ = expr`)
- Definite assignment analysis
- Sequence production via `emit` (semantic: intermediate results in a sequence)
**Source:** `how/concepts/research/essential/EXPRESSION_ORIENTED_LANGUAGE.md`, `how/concepts/research/important/DECLARATION_BY_ASSIGNMENT.md`
**Requirement:** SEM-01
**Estimate:** 45 min

---

## Wave 3: Synthesis (depends on Wave 2)

### Task 3.1 — Write Cross-Dimension Consistency section
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Document all pairwise interactions between the 6 dimensions. For each pair (15 total), state whether they are orthogonal, where they interact, and how conflicts are resolved. Key interactions to document in detail:
- Identity ↔ Ownership (orthogonal — binding identity vs value identity distinction)
- Ownership ↔ Mutation (exclusive access invariant bridges both)
- Mutation ↔ Evaluation (mutating expressions in expression position — evaluation order and side-effect visibility)
- Visibility ↔ Ownership (edge case: `priv` type in `pub` function signature)
- Lifetime ↔ All (scope-based lifetime touches every dimension)
**Requirement:** SEM-02
**Estimate:** 60 min

### Task 3.2 — Write Design Principles verification
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Verify each semantic dimension against DESIGN_PRINCIPLES.md. Create a table per dimension showing principles satisfied and any potential conflicts. Key checks:
- Explicitness: ownership transfer, mutation, lifetime changes must be syntactically visible
- Orthogonality: all 6 dimensions compose freely
- Semantic Purity: each dimension has exactly one meaning
- Minimal Core: 6 dimensions are the minimum needed
- LLM Generability: each dimension passes the LLM Generability Gate
**Source:** `how/DESIGN_PRINCIPLES.md`
**Requirement:** SEM-02
**Estimate:** 30 min

### Task 3.3 — Create EDR (Architecture category)
**Files:** `how/decision_records/architecture/EDR-NNN-semantic-model.md`
**Description:** Create EDR accepting the Orthon Semantic Model. Use `how/templates/_edr-architecture.md` template. Must include:
- Category: Architecture (Tier 1)
- Context: Phase 1 + 1.1 complete, 10 essential-tier research files synthesized
- Decision: All 6 semantic dimensions as defined in SEMANTIC_MODEL.md
- Rationale: 6 dimensions are the minimal set that fully characterizes a program's meaning
- Consequences: Positive (stable foundation) and negative (enforcement mechanisms deferred)
- Alternatives: Identity-based safety (rejected), GC-based model (rejected), Statement-oriented evaluation (rejected)
- Related documents: All source files listed
**Requirement:** SEM-03
**Estimate:** 30 min

### Task 3.4 — Index EDR in INDEX.md
**Files:** `how/decision_records/INDEX.md`
**Description:** Add the new EDR entry to the unified EDR journal. Determine next sequential EDR number from INDEX.md.
**Requirement:** SEM-03
**Estimate:** 5 min

---

## Wave 4: Validation

### Task 4.1 — Run 7 Decision Validation gates
**Files:** `what/SEMANTIC_MODEL.md`
**Description:** Run all 7 Decision Validation gates (from `how/gates/DECISION_VALIDATION.md`) on the completed SEMANTIC_MODEL.md:
1. Correctness Gate — is the model internally consistent?
2. Implementation Independence Gate — is the model strategy-agnostic?
3. Orthogonality Gate — do dimensions compose freely?
4. Learnability Gate — can a programmer learn 6 dimensions?
5. LLM Generability Gate — can an LLM generate correct code from this model?
6. Principle Alignment Gate — does it satisfy DESIGN_PRINCIPLES.md?
7. Completeness Gate — are all 6 dimensions fully specified?
**Requirement:** SEM-01, SEM-02
**Estimate:** 45 min

### Task 4.2 — Update GLOSSARY.md with new terms
**Files:** `what/GLOSSARY.md`
**Description:** Add any new terminology introduced in the semantic model (e.g., "binding identity", "value identity", "three-way mutation distinction" if not already defined). Cross-reference back to SEMANTIC_MODEL.md.
**Requirement:** SEM-01
**Estimate:** 15 min

---

## Dependencies

```
Wave 1 (T1.1–T1.5) ──► Wave 2 (T2.1–T2.3) ──► Wave 3 (T3.1–T3.4) ──► Wave 4 (T4.1–T4.2)
     (parallel)                  (parallel)               (serial)              (serial)
```

**Critical path:** T1.2 → T2.1 → T2.2 → T2.3 → T3.1 → T3.2 → T3.3 → T4.1
**Total estimated effort:** ~7 hours

## Verification

After execution, confirm:
- [ ] `what/SEMANTIC_MODEL.md` has all 6 dimensions filled (not empty sections)
- [ ] Cross-Dimension Consistency section resolves all 15 pairwise interactions
- [ ] Design Principles verification table shows each dimension satisfies DESIGN_PRINCIPLES.md
- [ ] EDR filed in `how/decision_records/architecture/EDR-NNN-semantic-model.md`
- [ ] EDR indexed in `how/decision_records/INDEX.md`
- [ ] All 7 Decision Validation gates pass
- [ ] GLOSSARY.md updated with any new terms
