# EDR-013: Orthon Semantic Model — Identity, Ownership, Mutation, Evaluation, Visibility, Lifetime

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

**Supersedes:** *None*

---

### Context

Phase 1 (Concerns Remediation) and Phase 1.1 (Foundation Completion)
established the project's process infrastructure and locked
`DESIGN_PRINCIPLES.md` as the constitutional design authority, but left
`what/SEMANTIC_MODEL.md` as a DRAFT placeholder with six empty sections
(Identity, Ownership, Mutation, Evaluation, Visibility, Lifetime) and no
formal answer to the question every subsequent phase depends on: **what
does an Orthon program mean?**

Ten essential-tier research documents already existed as raw material,
each independently exploring one facet of this question:

- `DATA_MODEL.md`, `VALUE_SEMANTICS.md` — data representation and
  copy-vs-reference semantics
- `OWNERSHIP.md`, `OWNERSHIP_METAPROPERTY.md`,
  `OWNERSHIP_TRANSFER_OPERATOR.md` — memory safety without GC
- `MUTABILITY.md`, `EXCLUSIVE_DECLARATIONS.md` — mutation visibility and
  the `fun`/`proc`/`new` declaration-kind hypothesis
- `IDENTITY_BASED_SAFETY.md` — a competing, rejected hypothesis
  (`.`/`!` operators with compiler-inferred mutability)
- `VISIBILITY_AND_ENCAPSULATION.md` — module-scoped access control
- `SCOPED_RESOURCE_LIFECYCLE.md` — RAII-style deterministic destruction
- `EXPRESSION_ORIENTED_LANGUAGE.md`, `DECLARATION_BY_ASSIGNMENT.md`
  (important tier) — expression-oriented control flow

These documents were never reconciled into one coherent model, and two
of them (`OWNERSHIP_METAPROPERTY.md` and `OWNERSHIP_TRANSFER_OPERATOR.md`)
explicitly compete for the same concrete syntax slot.
`IDENTITY_BASED_SAFETY.md` proposes an entirely different foundational
approach (implicit mutability inference) that is incompatible with
several `DESIGN_PRINCIPLES.md` commitments (Explicitness, Explicit
Semantics).

Phase 2's user, working in assumptions-mode interactive context
gathering (`02-CONTEXT.md`), resolved the open questions across all ten
source documents into six locked decisions (D-01 through D-06). This EDR
formally accepts the resulting synthesis as `what/SEMANTIC_MODEL.md`,
Orthon's Core-Language semantic contract.

### Decision

Adopt **six semantic dimensions** — Identity, Ownership, Mutation,
Evaluation, Visibility, and Lifetime — as the complete, minimal set of
foundational semantic contracts an Orthon program must satisfy,
independent of syntax (Phase 5), Primitive Blocks (Phase 3), Derived
Features (Phase 4), and Implementation Strategy (Phase 7). Full
specification lives in `what/SEMANTIC_MODEL.md`; summarized here:

1. **Identity** — Value semantics by default (structural equality, copy
   on assignment). Identity is not universal; it exists only for
   explicit, opt-in shared/reference types. Binding identity (aliasing)
   is distinct from value identity (persistent entity across mutation).
   Orthogonal to Ownership. Fresh (unbound) temporaries are
   identity-exempt.

2. **Ownership** — Every value has exactly one owner at any point.
   Ownership applies wherever exclusive responsibility exists (not only
   "external resources") — in practice, ~95% of ordinary-value code
   never needs to reason about it, because plain value semantics
   eliminates the question. Move transfers ownership and invalidates
   the source; borrowing grants temporary shared-XOR-mutable access.
   Transfer must be syntactically explicit; the concrete marker
   (`@ownership` / `$` / `move`) is deferred to Phase 5. No GC, no RC by
   default. The enforcement mechanism (borrow checker vs. escape
   analysis vs. other) is deliberately **not** prescribed — that is an
   Implementation Strategy (Phase 7) decision.

3. **Mutation** — Immutable by default (`val`/`var`, declaration by
   assignment). Function-level mutation is expressed through three
   mutually exclusive declaration kinds — `fun` (read-only), `proc`
   (mutates `self`, identity preserved), `new` (transforms, identity
   changed) — replacing a `mut` modifier. No `mut` at the call site; the
   contract lives in the declaration. Mutation requires exclusive access
   (the same invariant Ownership enforces, viewed from the operation's
   perspective rather than the accessor's).

4. **Evaluation** — Expression-oriented: `if`/`when`/`try`/blocks all
   produce values; no statement/expression grammar split; no ternary
   operator. Eager by default; laziness requires an explicit marker
   (`sec` or equivalent, syntax deferred to Phase 5). Defined
   left-to-right sub-expression evaluation order. `emit` (not `yield`)
   for sequence production.

5. **Visibility** — Three levels: `priv` (type/function-local), default
   (module-scoped), `pub` (exported). Module is the encapsulation
   boundary, not the type. No `protected`, no runtime bypass —
   compile-time enforcement only. Whether `pub` on a type implies `pub`
   on its members is an open question deferred to Phase 5.

6. **Lifetime** — Scope-based: a value's lifetime is bound to its
   enclosing scope, with deterministic destruction at scope exit. No GC
   by default. Reference lifetime must not exceed referent lifetime.
   Regions/arenas are permitted as Implementation Strategy choices
   (Phase 7), not semantic requirements.

All fifteen pairwise interactions between these six dimensions were
checked for orthogonality (`what/SEMANTIC_MODEL.md` § Cross-Dimension
Consistency); the sharpest of several documented couplings — Ownership
and Mutation both instantiating a single shared invariant (exclusive
access) from two angles — is accepted as intentional, not as an
orthogonality violation. All seven `how/gates/DECISION_VALIDATION.md` gates were run
against the completed model (`what/SEMANTIC_MODEL.md` § Validation); no
gate failed.

### Rationale

Six dimensions were determined to be the minimal set that fully
characterizes a program's meaning, tested against
`DESIGN_PRINCIPLES.md` § Minimal Core's "Core changes only when new
semantics cannot be expressed through composition of existing Core
primitives" bar. Candidate seventh dimensions were considered and
rejected during synthesis:

- **A separate "Aliasing" dimension** — fully explained as the
  composition of Ownership's borrowing rules (shared XOR mutable) with
  Mutation's exclusive-access requirement; no additional primitive
  needed.
- **A separate "Concurrency" dimension** — concurrency-safe mutation is
  a *consequence* of Ownership and Mutation composing correctly (the
  same exclusivity invariant that prevents aliasing bugs in
  single-threaded code prevents data races across threads); Phase 7
  Implementation Strategies build concurrency policies on top of this
  foundation rather than the Core needing a seventh primitive dimension.

Each of the ten source documents' proposals was merged, modified, or
superseded as follows:

| Source Document | Disposition |
|---|---|
| `DATA_MODEL.md` | Merged — value semantics default and seven representations inform Identity and the broader data model context. |
| `VALUE_SEMANTICS.md` | Merged — Swift-style struct-by-default model adopted directly for Identity. |
| `OWNERSHIP.md` | Merged — single-owner, move, borrow model adopted for Ownership; enforcement-mechanism openness preserved rather than resolved. |
| `OWNERSHIP_METAPROPERTY.md` | Modified — `@ownership` retained as one of two live candidate syntaxes, decision deferred to Phase 5 rather than chosen here. |
| `OWNERSHIP_TRANSFER_OPERATOR.md` | Modified — `$` retained as the other live candidate syntax, same deferral. |
| `MUTABILITY.md` | Merged — immutable-by-default and the four governing principles adopted, restated in `fun`/`proc`/`new` terms per `EXCLUSIVE_DECLARATIONS.md`. |
| `EXCLUSIVE_DECLARATIONS.md` | Merged — `fun`/`proc`/`new` adopted as the primary mutation-declaration model, superseding the `mut fun` modifier form. |
| `IDENTITY_BASED_SAFETY.md` | Superseded — its `.`/`!` implicit-mutability-inference model rejected as the enforcement mechanism because it conflicts with Explicitness; its ownership + escape-analysis reasoning about uniqueness is preserved as one *compatible* future enforcement strategy, not mandated. |
| `VISIBILITY_AND_ENCAPSULATION.md` | Merged — three-level model (`priv`/default/`pub`) adopted directly. |
| `SCOPED_RESOURCE_LIFECYCLE.md` | Merged — scope-based deterministic destruction adopted directly for Lifetime. |
| `EXPRESSION_ORIENTED_LANGUAGE.md` | Merged — expression-orientation, no-ternary, and `emit`-over-`yield` adopted directly for Evaluation. |
| `DECLARATION_BY_ASSIGNMENT.md` | Merged — declaration-by-first-assignment and definite-assignment analysis adopted, cross-referenced from both Mutation and Evaluation. |

### Consequences

- **Positive:**
  - Provides a stable, syntax-independent foundation for Phase 3
    (Primitive Blocks), Phase 4 (Derived Features), and Phase 5 (Syntax
    Design) — all three can now decompose or derive against a fixed
    semantic contract instead of ten scattered, sometimes-competing
    research documents.
  - Resolves the `OWNERSHIP_METAPROPERTY.md` vs.
    `OWNERSHIP_TRANSFER_OPERATOR.md` syntax competition without
    prematurely choosing a winner — both remain viable Phase 5
    candidates, and the semantic requirement they must both satisfy is
    now fixed.
  - Formally rejects `IDENTITY_BASED_SAFETY.md`'s implicit-mutability
    approach, closing a foundational fork that would otherwise have
    remained open into Phase 3+ and risked contaminating primitive
    design with two incompatible mutation models.
  - Establishes Ownership's enforcement-mechanism independence
    explicitly, giving Phase 7 (Implementation Strategy) genuine freedom
    to choose a borrow checker, escape analysis, or another mechanism
    without a Phase 2 semantic change.
- **Negative:**
  - Two Flags remain open against the `LLM_GENERABILITY_GATE` and
    Explicitness checks, both rooted in the same cause: ownership
    transfer's concrete syntax is undecided until Phase 5. Until then,
    no canonical Orthon code example exists for ownership transfer to
    exercise the `LLM_GENERABILITY_GATE`'s schema-round-trip criterion
    end-to-end.
  - `pub`-on-type-implies-`pub`-on-members remains an open question,
    deferred to Phase 5 — Visibility's specification is not fully closed
    until that question resolves.
  - Deferring enforcement mechanism to Phase 7 means Phase 3/4 work
    building on Ownership must be written in enforcement-agnostic terms,
    which is a mild but real authoring discipline cost for every
    subsequent phase.

### Compliance

Compliance is verified through:

1. **Decision Validation** (`how/gates/DECISION_VALIDATION.md`) — all
   seven gates were run against the completed `what/SEMANTIC_MODEL.md`
   (§ Validation); results are recorded there and summarized in this
   EDR's Decision section.
2. **Cross-Dimension Consistency** (`what/SEMANTIC_MODEL.md` § that
   name) — all fifteen pairwise dimension interactions are documented;
   any future concept (Phase 3+) that appears to require a sixteenth
   interaction or a new dimension must first be checked against this
   section before being accepted.
3. **GLOSSARY.md** — all new terminology introduced by this synthesis
   (binding identity, value identity, exclusive access, the
   `fun`/`proc`/`new` declaration kinds, etc.) is registered per the
   Glossary maintenance workflow.
4. **Phase 3+ dependency check** — Phase 3 (Primitive Blocks) and
   Phase 4 (Derived Features) plans must cite the relevant
   `what/SEMANTIC_MODEL.md` dimension(s) their primitives/features
   decompose from or derive against; a primitive or feature that cannot
   be traced to one of the six dimensions is out of scope until this EDR
   is amended.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| **Identity-based safety model** (`.`/`!` operators, compiler-inferred mutability, no `mut`/lifetime annotations) | Rejected as the foundational model: implicit mutability inference means a reader cannot determine from a call site alone whether a method mutates its receiver, violating `DESIGN_PRINCIPLES.md` § Explicitness and § Explicit Semantics. Its escape-analysis-based uniqueness checking remains a *viable enforcement strategy* for Ownership (Phase 7), just not adopted as the semantic model itself. |
| **GC-based model** (tracing garbage collection as the default memory model) | Rejected as the default: contradicts the "No GC by default" decision underlying both Ownership and Lifetime, and reintroduces the stop-the-world-pause / unpredictable-latency trade-off `OWNERSHIP.md` identifies as the core problem Orthon's ownership model exists to avoid. Remains available as an opt-in Phase 7 Implementation Strategy. |
| **Statement-oriented evaluation** (Java/C/Go-style `if` as a statement, ternary operator for the expression case) | Rejected: forces mutable temporary variables for conditional assignment, increases the state an LLM must track across statements (contradicts the LLM Readiness vision pillar), and requires a redundant ternary operator that an expression-oriented `if` makes unnecessary — a Minimal Core violation. |
| **Seven dimensions, splitting Aliasing out of Ownership+Mutation** | Rejected: Aliasing control is fully derivable as the composition of Ownership's shared-XOR-mutable borrowing rule and Mutation's exclusive-access requirement; introducing a seventh named dimension for a fully composed concept would violate Minimal Core without adding expressive power. |
| **Seven dimensions, adding Concurrency** | Rejected for the same reason — concurrency-safe mutation is a consequence of Ownership + Mutation, to be built on top via Phase 7 concurrency Policies, not a new Core semantic primitive. |
| **Resolve `@ownership` vs. `$` now, in Phase 2** | Rejected: syntax choice is explicitly out of scope for the semantic model per the phase boundary (`02-CONTEXT.md`); forcing a premature choice here would couple a Phase 5 (Syntax Design) decision to Phase 2 deliverables and risk relitigating it once Phase 5's own validation process runs. |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-010 (Layered Architecture) | This EDR's six dimensions define the Core Language layer's semantic contract that EDR-010's Implementation Strategy layer must fulfill. |
| EDR-012 (Semantic Dependency Architecture) | The six dimensions operate at Level 0/1 (Data Model / Primitive Operations) of EDR-012's pyramid; Phase 3's Primitive Blocks (Level 1) must decompose onto this semantic model. |
| EDR-014 (LLM Generability Gate) | The `LLM_GENERABILITY_GATE` criteria from that EDR were applied during this model's validation (see Consequences § Negative for the two open Flags). |
