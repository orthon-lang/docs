# Decision Log

> The detailed reasoning trail behind Tier 1–2 gate validations.
>
> **Relationship to other records:** `DECISION_VALIDATION.md`'s
> [Decision Journal](DECISION_VALIDATION.md#decision-journal) is a
> one-row-per-decision summary (date, gates applied, verdict). An
> accepted artifact's own "Validation" section (e.g.
> `what/SEMANTIC_MODEL.md` § Validation) records each gate's
> **verdict and conclusion**. Neither shows the **reasoning that
> produced the verdict**. This log is where that reasoning lives: one
> entry per validated decision, one subsection per gate, each showing
> the method applied (per `DECISION_VALIDATION.md` § Methods Overview)
> worked through against the actual proposal — not just its result.
>
> Established by [EDR-014](../decision_records/process/EDR-014-decision-log.md).

---

## Entry: Semantic Model (EDR-013)

**Date:** 2026-07-27
**Artifact validated:** [`what/SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md)
**Decision recorded as:** [EDR-013](../decision_records/architecture/EDR-013-semantic-model.md)
**Gates applied:** All 7 (a new Core Language semantic requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon language designer — and, by extension,
every future Orthon programmer and code-generating LLM — I want one
unified answer to "what does a program mean" so that every downstream
design decision (Primitive Blocks, Derived Features, Syntax) has
stable ground instead of re-litigating basic semantics per feature.

**Press release.** *Orthon's semantic foundation moves from six
independent, occasionally conflicting research drafts to one coherent
Semantic Model. Programmers and LLMs now have a single source of
truth for identity, ownership, mutation, evaluation, visibility, and
lifetime — no more guessing whether a construct's behavior is an
accident of whichever research document proposed it first.*

**FAQ.**
- *How is this different from the individual research docs
  (`DATA_MODEL.md`, `OWNERSHIP.md`, etc.)?* — Those were independent,
  sometimes-competing proposals (e.g. `IDENTITY_BASED_SAFETY.md`'s
  `.`/`!` operator model directly conflicts with value-semantics-by-
  default). `SEMANTIC_MODEL.md` reconciles them into one accepted
  model; the research docs become historical inputs, cited as
  `Source:` per dimension.
- *When would I use this instead of a research doc?* — Always;
  `SEMANTIC_MODEL.md` is now canonical.
- *What do I lose without it?* — Phase 3 (Primitive Blocks) and Phase
  4 (Derived Features) would have no stable foundation to decompose
  against, and would re-derive basic semantic questions ad hoc, per
  feature, with no guarantee of consistency between features.
- *Is there a migration path?* — Not applicable; this is a
  pre-implementation specification with no existing code to migrate.

**Requirements derived.** A canonical semantic reference document;
explicit resolution of the three competing ownership-transfer
proposals (`OWNERSHIP.md`, `OWNERSHIP_METAPROPERTY.md`,
`OWNERSHIP_TRANSFER_OPERATOR.md`); explicit disposition of
`IDENTITY_BASED_SAFETY.md`'s rejected model. All three are satisfied
in the delivered document.

**Verdict: Pass.** The benefit is concrete (a stable dependency for
Phases 3–5) and stated in terms of who needs it, not merely "it is
technically interesting."

---

### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Binding identity vs. value identity are
precisely and separately defined (§ Identity); `fun`/`proc`/`new` are
three mutually exclusive declaration kinds (§ Mutation); `priv`/
default/`pub` are three levels with exactly one meaning each (§
Visibility). All seven new terms are now registered in
`what/GLOSSARY.md`, satisfying "is this term defined in the project
Glossary?"

**Test with counterexamples.**
- *What happens when a `priv` type appears in a `pub` function's
  signature?* — Explicitly resolved (Cross-Dimension Consistency pair
  8): legal and expected, not a special case requiring reconciliation.
- *What happens when a `proc` call sits inside an `if` condition?* —
  Resolved via the defined left-to-right evaluation order (pair 10):
  side-effect order is deterministic regardless of nesting depth.
- *Does structural equality break when a shared/reference type
  mutates?* — Resolved (pair 2): identity survives mutation by
  construction, because identity is defined independently of
  structure for reference types.

**Follow the contradiction.** Apparent tension: "no GC by default"
(Lifetime) vs. "opt-in reference/shared types" (Identity) — doesn't a
shared type need *something* to manage its lifetime? Resolved
principled, not patched: a reference type still has exactly one owner
of *the reference itself* (Ownership's invariant applies uniformly);
opt-in RC/GC becomes an Implementation Strategy concern for the
*referent*, not a Core semantic exception. The higher principle
(Semantic Invariant 1) gives the answer; no arbitrary carve-out was
introduced.

**Play devil's advocate.** Strongest attack: "If Identity and
Ownership are truly orthogonal, why does Ownership's single-owner
rule constrain reference types at all?" The document pre-empts this
directly (Cross-Dimension Consistency, pair 1 detail): Ownership's
invariant applies to *every* value, independent of whether it carries
identity — the rule isn't derived from Identity's properties, so
applying it to identity-bearing values isn't a violation of
orthogonality, it's the same universal rule applied uniformly.

**Verdict: Pass.** No term is left ambiguous, the one apparent
contradiction resolves to a documented principle rather than an ad
hoc exception, and the strongest counterargument found is already
answered in the text rather than left standing.

---

### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Six dimensions are the minimum needed to fully
characterize what an Orthon program means — no seventh dimension is
required."

**What is known.** Two candidate seventh dimensions surface in the
source research: **Aliasing** (implicit in
`IDENTITY_BASED_SAFETY.md`'s escape-analysis machinery) and
**Concurrency** (implicit in actor-model research referenced
elsewhere in the project).

**What is unknown (pre-test).** Whether Aliasing and Concurrency
carry independent semantic content, or reduce fully to composition of
the existing six.

**Simplest experiment.** Express "two mutable references to the same
resource must never coexist" (an Aliasing concern) using only
Ownership's shared-XOR-mutable borrowing rule plus Mutation's
exclusive-access requirement for `proc` — no new vocabulary is
needed; both are already stated as Semantic Invariant 2's two halves.
Express "concurrency-safe mutation" the same way: a `proc` call still
requires exclusive access to `self` whether the second reader/writer
is a second thread or the same thread — no additional invariant is
introduced to make the rule concurrency-safe, because the rule was
never expressed in terms of "thread" in the first place.

**Evaluate the evidence.** Both experiments confirm the candidates
are already expressible via composition of Ownership + Mutation; the
hypothesis holds. This is exactly the "Minimal Core check" the
document itself records under Relationship to Design Principles.

**Verdict: Pass.** Composition of the existing six dimensions
produces both candidate sevenths without residue; nothing was
retained that composition could already deliver.

---

### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Core Language has no Standard Library or
Implementation Strategy dependencies (`ARCHITECTURE.md`). (2) The
Semantic Model occupies Level 0/1 of the Semantic Dependency
Architecture (EDR-012's six-level pyramid).

**Deduce the consequences.** If accepted: every dimension's rules
must hold under any Strategy (Default / Embedded / High-Performance)
without modification; Phase 3 must decompose Primitive Blocks without
re-deriving semantics; Phase 5 must express these semantics in syntax
without altering them.

**Test for contradiction.** Does any dimension secretly require one
specific Strategy? Checked every "Implementation freedom" callout:
Identity permits CoW/SSA/NRVO/register promotion; Ownership is
deliberately silent on borrow-checker vs. escape-analysis
enforcement; Lifetime treats stack/region/arena/GC as Allocation
Policy choices. No dimension's *observable* semantics changes across
Strategies — none found in violation of premise (1).

**Identify hidden premises.** Does the model quietly assume
compile-time type information is available (needed for Evaluation's
exhaustiveness/definite-assignment checks)? Yes — but this is stated
explicitly (compile-time enforcement is called out directly for both
Visibility and Evaluation), so it is an accepted, documented
constraint, not a hidden one.

**Verdict: Pass.** The model sits cleanly at the Core layer; the one
non-trivial cross-dimension coupling found (Ownership ↔ Mutation,
pair 6) is explicitly identified and justified as one invariant with
two names, not a layering leak between dimensions.

---

### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** To guarantee deterministic destruction
(Lifetime) without a runtime GC, the compiler must statically know a
value's exact death point — but the Embedded Strategy has a
minimal compile-time analysis budget, and the High-Performance
Strategy wants allocation flexibility (pools, arenas). These appear
to pull toward different, Strategy-specific implementations of the
same guarantee.

**Apply separation principles.** **Separation by scale**: define
lifetime purely at the semantic level (scope-bound, deterministic
end-point) and push *mechanism* (stack pop, region reset, arena free,
GC sweep) down into the Allocation Policy — a Strategy-level concern.
**Separation on condition**: the semantic guarantee (exactly-once,
statically-known destruction point) never changes per Strategy; only
*how* each Strategy realizes that transition does.

**Consult known patterns.** This mirrors `DESIGN_PRINCIPLES.md` §
Semantics Before Optimization directly, and the same
separation-of-concerns shape the project already uses for Data
Modifiers.

**Formulate the abstract solution.** *"A value transitions from live
to destroyed at a statically-determined point derived from lexical
scope; each Strategy realizes that transition using its own
allocation mechanism without changing the observable transition
point."* This formulation holds for all six dimensions — none require
Strategy-specific language to state.

**Verdict: Pass, with one already-tracked exception.** All six
Core semantic definitions pass the abstraction test cleanly.
Ownership's *concrete transfer syntax* does not yet fully abstract
away — `@ownership` / `$` / `move` read differently under different
mental models of compilation, and none is chosen — so this one item
remains an open Flag, identical to the Flag already recorded in the
Design Principles table, deferred to Phase 5.

---

### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain each dimension in one sentence** (none may contain "and",
"but", "except", "however", "unless"):

- Identity: *"This lets the programmer treat data as copies by
  default and opt into sharing only when they say so explicitly."*
- Ownership: *"This lets the programmer know, from the type alone,
  that resources are never silently duplicated or leaked."*
- Mutation: *"This lets the programmer see whether a call can change
  its receiver just by reading its declaration keyword."*
- Evaluation: *"This lets the programmer treat every construct as a
  value-producing expression."*
- Visibility: *"This lets the programmer control what's reachable
  from outside a module using three explicit levels."*
- Lifetime: *"This lets the programmer know exactly when a value is
  destroyed just by reading the surrounding scope."*

All six pass the conjunction test unmodified.

**Explain to a non-expert.** A programmer coming from Python or Java
recognizes each sentence against a familiar parallel: value-vs-
reference semantics (Swift's `struct`/`class`), `val`/`var`
(Kotlin/Swift), expression-oriented control flow (Kotlin's `if`,
Rust's blocks). No unfamiliar concept is required to parse the
one-sentence explanation.

**Remove one thing.** If the three-way `fun`/`proc`/`new` split
collapsed to two (drop `new`), the model loses the ability to
distinguish "produces a *transformed* value with new identity" from
"mutates in place" — exactly the distinction the document already
tested and confirmed necessary when it rejected a single-keyword-
plus-`mut`-modifier design. Removing the most complex rule breaks the
concept; three is confirmed as the floor, not a padded ceiling.

**Check for obviousness.** Each rule reads as "the natural way," not
"clever": scope-bound lifetime and expression-oriented control flow
are established patterns (Rust, Kotlin, Swift), not novel invention —
itself evidence of low long-term conceptual risk, since proven
patterns age better than experimental ones.

**Verdict: Pass.** The two open Flags (Ownership transfer syntax,
whether `pub` on a type implies `pub` on its members) are both
explicitly reversible — Phase 5 retains full freedom among candidate
forms without unwinding any Phase 2 commitment. No permanent
complexity debt is accepted.

---

### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis** (scan each dimension for known LLM-unfriendly
patterns — ambiguity, special cases, context-dependent syntax):

- Identity: opt-in reference semantics only — no ambiguity, no
  context-dependent parse. Clean.
- Ownership: the *semantic* rule (transfer must be visible) is
  unambiguous, but three competing concrete syntaxes remain open —
  exactly the "context-dependent syntax not yet fixed" structural
  risk pattern this analysis looks for.
- Mutation: three keywords, zero combination forms, zero ambiguity.
- Evaluation: expression-orientation removes the entire
  statement-vs-expression ambiguity class that historically confuses
  code generation (no "should this emit a `return`, or is it already
  a value?" decision).
- Visibility: three explicit levels, no naming-convention fallback
  (naming conventions, e.g. Python's leading underscore, are
  notoriously unreliable for LLMs to apply consistently).
- Lifetime: deterministic, scope-derived — no allocation-strategy
  decision the generator must reason about.

**Schema round-trip** (can each dimension be fully expressed in a
machine-readable grammar/type schema?): Identity, Mutation,
Evaluation, Visibility, and Lifetime — yes, cleanly; each already has
a fully-fixed grammar shape (see each dimension's code example).
Ownership — the *semantic* contract (owner uniqueness, move
invalidation) is schema-serializable, but no canonical schema
round-trip example can be written yet for the transfer-marker case,
because no concrete marker is chosen.

**Apply the gate's five-criterion table directly:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass (5 dims) / Flag (Ownership) | Semantic content fully representable everywhere; Ownership's transfer notation is the one partial-representation case |
| Predictable generation (≥90%) | Pass (5 dims) / Flag (Ownership) | Cannot yet be empirically measured for Ownership transfer without a fixed syntax to test against |
| No hallucination surface | Pass | Every form has exactly one meaning (Mutation, Visibility), or removes an ambiguity class outright (Evaluation) |
| Strategy-aware default | Pass | Every "Implementation freedom" callout confirms default-generated code is valid under any Strategy without extra annotation |
| Self-correctable | Pass | Violations (moved-value reuse, exclusive-access violation, visibility breach) are all static-analysis-detectable per the model's compile-time-enforcement framing |

**Verdict: Flag.** Four of five criteria Pass cleanly across all six
dimensions; the fifth (schema-serializability / predictable
generation) Flags specifically and only on Ownership's open transfer
syntax — the identical root cause already recorded for this item in
the Design Principles table and the `IMPLEMENTATION_INDEPENDENCE_GATE`
entry above. Per `DECISION_VALIDATION.md`'s Gate Flow rules, a Flag
"may proceed" — resolution is deferred to Phase 5.

---

### Overall

Six gates Pass outright; one gate (`LLM_GENERABILITY_GATE`) Flags on
a single root cause (Ownership's undecided transfer syntax) that is
independently surfaced by three different methods above (TRIZ,
Einstein's Method, and this gate's own criteria) — convergent
evidence, not three unrelated concerns. No gate returned Fail. The
model is accepted via EDR-013 with this one item explicitly deferred
to Phase 5's validation cycle.

---

## Entry: Primitive Blocks (EDR-016)

**Date:** 2026-07-27
**Artifact validated:** [`what/PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md)
**Decision recorded as:** [EDR-016](../decision_records/architecture/EDR-016-primitive-blocks.md)
**Gates applied:** All 7 (a new architectural foundation at Level 1 of the Semantic Dependency Architecture requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon language designer working on Phase 4 (Derived
Features), I want one irreducible set of primitive operations that every
language feature decomposes onto, so that I can verify a new feature is
not secretly adding a new primitive when composition of existing ones
would suffice, and so that syntax designers (Phase 5) have a stable target
to render.

**Press release.** *Orthon's design pipeline now has a settled foundation.
Phase 4 concepts — pattern matching, error handling, generics, traits,
iterators — all decompose onto exactly 9 primitive operations, down from
the earlier 11-item hypothesis. The set is verified minimal: removing any
one makes at least one known feature inexpressible. Phase 5 and Phase 6
can proceed with confidence that no hidden atomic operation will be
discovered halfway through derived feature design.*

**FAQ.**
- *How is this different from the 11-item hypothesis in the DRAFT
  PRIMITIVE_BLOCKS.md?* — `operator definition` removed (it is syntactic
  sugar for `function` per Named Before Symbolic). `struct`, `class`,
  `delegate`, `namespace` removed (they are compositions, not primitives).
  `pack`/`unpack` unified into one symmetric pair.
- *When would I use this instead of the Semantic Model?* — The Semantic
  Model (EDR-013) answers *what programs mean*; the Primitive Blocks
  answer *how the language makes that meaning operational*. They are
  complementary — Phase 3 maps each primitive to its Semantic Dimensions.
- *What do I lose without it?* — Phase 4 would have no stable foundation
  to decompose against, and would either re-derive basic operations ad
  hoc per feature (inconsistency risk) or introduce hidden primitives
  disguised as derived features (non-orthogonality risk).

**Requirements derived.** A 9-primitive specification document
(`what/PRIMITIVE_BLOCKS.md`); verification that all ~132 concept research
files decompose onto this set; an EDR formally accepting the set. All
three are satisfied.

**Verdict: Pass.** The benefit is concrete (a stable decomposition target
for Phases 4-6) and the cost is bounded (9 primitives is a
comprehensible set).

---

### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Each of the 9 primitives has a precise, single-
sentence definition and an explicit orthogonal-to statement listing every
other primitive it does NOT overlap with.

**Test with counterexamples.**
- *What happens when `call` and `function` appear to overlap?* — They
  are explicitly separated: `function` is declaration (defines *what*);
  `call` is invocation (triggers *when*). A function declaration without
  a call is a valid (if unused) construct; a call without a declaration
  (to a known function) is a compile-time error. The overlap is
  intentional composition, not ambiguity.
- *What happens when `pack` and `unpack` are the same syntax?* — Per the
  symmetry principle (UNPACKING.md), construction and destruction follow
  the same structural syntax — the *direction* is determined by context
  (expression position vs. binding position). This is a single primitive
  with two operations, not two primitives that accidentally share syntax.
- *Does `scope` overlap with `reference` for lifetime?* — No. Scope
  defines *where a binding lives* (lexical region). Reference defines
  *how to point to a value without owning it* (indirection). Reference
  lifetime is bounded by scope, but they are independent operations — a
  scope with no references is still a scope; a reference whose referent
  is outside the current scope (but still alive) is valid.

**Follow the contradiction.** Apparent tension: `assignment` is both an
Ownership operation (establishing ownership) and an Evaluation operation
(storing a value for later). Is this a dimension leak? Resolved:
assignment's single mechanism (bind name to value) produces *both*
effects — a binding necessarily establishes ownership *and* makes the
value available. This is one operation with two natural consequences,
not two operations conflated into one primitive.

**Play devil's advocate.** Strongest attack: "If `pack`/`unpack` is one
primitive, why isn't `function`/`call` also one primitive?" The document
pre-empts this directly: `pack`/`unpack` are one primitive because
construction and destruction of the *same structural shape* are inverse
operations that share syntax. `function`/`call` are separate because
declaration and invocation serve *different semantic dimensions*
(Evaluation for function, Evaluation + Lifetime for call) and have
different orthogonalities.

**Verdict: Pass.** Every term has a single, clear definition. The one
apparent contradiction (assignment serving two dimensions) is documented
as a natural consequence, not an ambiguity. Edge cases at overlap
boundaries are explicitly addressed.

---

### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "9 primitives are the minimum needed — no primitive can
be removed without making some concept inexpressible."

**What is known.** The starting 11-primitive hypothesis (from the DRAFT
PRIMITIVE_BLOCKS.md) includes `operator definition`, `struct`, `class`,
`delegate`, `namespace` — all of which can be expressed as compositions
of simpler primitives.

**What is unknown (pre-test).** Whether any of the remaining 9 primitives
is redundant (expressible as composition of the other 8).

**Simplest experiment.** For each primitive, attempt to express every
concept in the ~132-file research catalog using only the other 8
primitives. If a concept becomes inexpressible, the removed primitive is
necessary.

**Evaluate the evidence.** All 9 tests confirmed necessity (documented in
PRIMITIVE_BLOCKS.md §10.6 Minimality Verification). For example:
removing `reference` makes class identity semantics (`CLASS_WITH_ACT.md`)
and borrowing (`OWNERSHIP.md`) inexpressible; removing `scope` makes
lifetime management and lexical boundaries inexpressible; removing
`assignment` makes variable binding and parameter passing inexpressible.

**Verdict: Pass.** The hypothesis holds. Nine is the floor.

---

### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Level 1 (Primitive Operations) sits between
Level 0 (Data Model) and Level 2 (Language Patterns) in the Semantic
Dependency Architecture (EDR-012). (2) Primitives must not reference
Level 2 constructs in their definitions. (3) Primitives must serve at
least one of the six Semantic Dimensions (EDR-013).

**Deduce the consequences.** If accepted: every Phase 4 (Level 2) feature
must decompose into these Level 1 primitives; no Phase 4 feature can
introduce a primitive-level operation without revisiting this set. Phase 5
(Syntax) renders these primitives in concrete syntax.

**Test for contradiction.** Does any primitive secretly require a Level 2
construct? Checked every definition and composition rule: `function`
defines a computation boundary but its body is composed of other
primitives (scope, assignment, call) — not of derived features.
`attribute access` defines member selection — no trait or class concept
is needed to understand it. No primitive references a Level 2 pattern.

**Identify hidden premises.** Does the set assume a particular execution
model (eager evaluation, stack-based) to make `scope` work? No — `scope`
is defined lexically; the *mechanism* of scope exit (stack pop, arena
reset, region exit) is delegated to the Allocation Policy (Phase 7).

**Verdict: Pass.** The primitive set sits cleanly at Level 1, requires
no Level 2 constructs, and serves only the six Level 0 Semantic
Dimensions.

---

### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Primitives must be concrete enough for Phase 4
decomposition verification, yet abstract enough to be independent of any
Implementation Strategy (Default, Embedded, High-Performance).

**Apply separation principles.** **Separation in space**: the *semantic
definition* of each primitive is strategy-independent (what it does); the
*policy values* that govern how it is realised (Allocation Policy for
scope lifetimes, Evaluation Policy for call semantics) are delegated to
Phase 7. **Separation on condition**: the observable behaviour of each
primitive never changes per Strategy; only the performance characteristics
of *how* it is implemented do.

**Consult known patterns.** This mirrors `DESIGN_PRINCIPLES.md` §
Semantics Before Optimization — define *what* before *how* — the same
separation the project already uses for all Core Language concepts.

**Formulate the abstract solution.** *"A primitive's observable semantics
are defined independently of any Implementation Strategy; each Strategy
realises the same semantics using its own Policy values, without changing
the observable behaviour."* Verified for all 9 primitives. Example:
`reference` is always "indirection without ownership transfer" — the
borrow checker (Default), escape analysis (Embedded), or region inference
(High-Performance) are all valid mechanisms for the same semantics.

**Verdict: Pass.** All 9 primitives pass the abstraction test. No
primitive's definition requires a specific Policy value.

---

### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain each primitive in one sentence** (none may contain "and", "but",
"except", "however", "unless"):

- `literal`: *"This lets the programmer create a value directly from
  source text."*
- `identifier`: *"This lets the programmer name a value and refer to it
  later."*
- `pack`/`unpack`: *"This lets the programmer combine values into a
  composite or decompose a composite into its parts."*
- `assignment`: *"This lets the programmer bind a name to a value."*
- `function`: *"This lets the programmer declare a reusable
  computation."*
- `call`: *"This lets the programmer trigger a declared computation."*
- `attribute access`: *"This lets the programmer select a named member
  of a composite value."*
- `scope`: *"This lets the programmer define a lexical boundary for
  names and lifetimes."*
- `reference`: *"This lets the programmer point to a value without
  owning it."*

All nine pass the conjunction test.

**Explain to a non-expert.** A programmer coming from Python, Java, or
Rust recognises each sentence against a familiar parallel: `literal` is
`42` or `"hello"`, `function`/`call` are `def` and `()`, `scope` is `{ }`,
`reference` is `&`. No unfamiliar concept is required to parse any
one-sentence explanation.

**Remove one thing.** Removing any primitive makes at least one concept
inexpressible (see CONCEPTUAL_SIMPLICITY_GATE). Nine is confirmed as
the floor, not a padded ceiling.

**Check for obviousness.** Each primitive reads as "the natural way, not
clever": `attribute access` via `.` is universal; `pack`/`unpack` as a
symmetric pair mirrors ML/Haskell pattern matching; `reference` as
indirection is the same model as Rust's `&T`/`&mut T`. Proven patterns
age better than experimental ones.

**Verdict: Pass.** No permanent complexity debt is accepted. The
9-primitive set is comprehensible, teachable, and founded on established
patterns.

---

### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis** (scan each primitive for known LLM-unfriendly
patterns — ambiguity, special cases, context-dependent syntax):

- `literal`: inline value notation — zero ambiguity, zero special cases.
- `identifier`: naming convention — minimum identifier length.
- `pack`/`unpack`: symmetric syntax — context determines direction, which
  is a single binary distinction. LLMs handle this pattern well (Python's
  star expressions, JavaScript's destructuring).
- `assignment`: `let`/`var` — clear keyword distinction for immutability
  vs. mutability.
- `function`: `fun`/`proc`/`new` — three keywords, zero combination
  forms, zero ambiguity. Each maps to one declaration kind.
- `call`: `()` — universal call syntax. No distinction between method
  call and function call at the primitive level.
- `attribute access`: `.` — single unambiguous operator.
- `scope`: `{ }` — universal block syntax. No ambiguity.
- `reference`: `&`/`&mut` — two-mode design with one prefix symbol and
  one modifier keyword.

**Schema round-trip** (can each primitive be fully expressed in a
machine-readable grammar/type schema?): All 9 primitives — yes, cleanly.
The abstract grammar in PRIMITIVE_BLOCKS.md (§3) provides concrete
examples for each.

**Apply the gate's five-criterion table directly:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | All 9 primitives have fixed, unambiguous grammar shapes in PRIMITIVE_BLOCKS.md |
| Predictable generation (≥90%) | Pass | Every primitive has one canonical form — no "which syntax should I use?" decision for the LLM |
| No hallucination surface | Pass | Every primitive has exactly one meaning — no context-dependent behaviour |
| Strategy-aware default | Pass | All primitives defined semantically, not mechanically — default-generated code is valid under any Strategy |
| Self-correctable | Pass | Violations (e.g., `attribute access` on non-composite, dangling `reference`) are all statically detectable per SEMANTIC_MODEL.md's compile-time enforcement framing |

**Verdict: Pass.** All five criteria pass cleanly across all 9
primitives. The `pack`/`unpack` symmetric syntax is the only potential
LLM confusion point, but the binary context distinction (expression
vs. binding) is well-understood by existing LLMs through analogous
patterns in Python, JavaScript, and Rust.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The 9-primitive
set is accepted via EDR-016, verified complete and minimal against the
~132-file concept research catalog, and ready for Phase 4 (Derived Features)
where every concept must decompose onto these primitives.
