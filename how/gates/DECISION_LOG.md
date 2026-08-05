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

---

## Entry: EQUALITY (EDR-017)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/EQUALITY.md`](../../what/concepts/EQUALITY.md)
**Decision recorded as:** [EDR-017](../decision_records/architecture/EDR-017-equality.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Programmers need predictable, unambiguous equality semantics. Reference vs. value equality is the #1 source of bugs in Java/Python/JS. |
| Q2 | Is this a language problem or a library problem? | **Language.** The compiler must know equality semantics for trait constraint checking and code generation (structural comparison). Three distinct operators (`===`, `==`, `is`) require parser and type-system support. |
| Q3 | Can it be solved with existing primitives? | No. `===` (structural value equality) requires compiler-generated field-by-field comparison — not expressible via composition of existing 9 primitives. `is` (identity) requires a runtime concept of reference identity not present in primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (different operators for different semantics), Consistency (same operator = same semantics for all types), Data First (structural by default). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Three distinct semantic operations: structural comparison, user-defined comparison, identity check. |
| Q6 | Can it be expressed through composition? | No. Structural equality requires compiler support to recurse into fields. Identity comparison requires a runtime concept of object identity. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Equality is a semantic operation, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Every programming task involves comparison. |

**Classification per D-03:** Language. Semantic uniqueness of `===` structural comparison not expressible via composition. Compiler must know equality for trait constraint checking.

**Primitive decomposition path:** `===` → compiler-generated field-by-field comparison of `pack`/`unpack` structure; `==` → `function` + trait dispatch (user-defined); `is` → `reference` identity check. None of these decompose to a single primitive without new semantics.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to compare two values without wondering whether I'm comparing their content or their memory location. I should never have to ask "does `==` do what I think it does for this type?".

**Press release.** *Orthon eliminates the #1 source of bugs in mainstream languages — reference vs. value equality confusion. Three operators, three clear meanings: `===` compares structure, `==` compares meaning, `is` compares identity. The default (`===`) always compares data by its content, matching programmer intuition.*

**FAQ.**
- *How is this different from Python where `==` is reference equality for custom objects?* — Orthon flips the default: structural by default, identity opt-in. This matches what programmers expect (data should compare by its content).
- *What if I need domain-specific comparison (e.g., two Person objects with the same ID)?* — Use `==`. It falls back to `===` by default but can be overridden via the `Eq` trait.
- *What about NaN?* — Deferred to the Standard Library. `NaN != NaN` violates the Transitivity Invariant, so it's excluded from core-language operators.

**Requirements derived.** Three distinct operators with documented semantics; Transitivity Invariant specification; NaN exclusion; `Eq` trait for `==` override. All satisfied in `what/concepts/EQUALITY.md`.

**Verdict: Pass.** The problem is stated in programmer terms, serves VISION.md's Comfortable by Design pillar, and the benefit (eliminating an entire bug class) clearly outweighs the cognitive cost of three operators vs. one.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `===` = structural value equality (compiler-generated field-by-field comparison). `==` = semantic equality (user-defined via `Eq` trait, falls back to `===`). `is` = identity equality (pointer comparison for reference types). Each operator has a precise, non-overlapping definition. The Transitivity Invariant is stated as a hard rule: if `a == b` and `b == c` then `a == c`.

**Test with counterexamples.**
- *What happens when `==` is not overridden?* — It falls back to `===` — a documented, explicit rule. Not an edge case.
- *What happens when `is` is used on a value type?* — Always returns `false`. The compiler emits a warning. This is a deterministic, predictable behaviour, not a silent surprise.
- *Can `===` and `==` produce different results for the same comparison?* — Yes — that's the intended design. `===` always compares structure; `==` may override for domain-specific equivalence.

**Follow the contradiction.** Apparent tension: `==` falls back to `===` by default, so they could be the same operator. Why have two? Resolved: they serve different *purposes* — `===` is always structural (data comparison), `==` is semantic (domain equivalence). Making them distinct forces the programmer to choose which kind of comparison they mean, eliminating the ambiguity that causes bugs.

**Play devil's advocate.** Strongest attack: "Three operators for comparison is over-engineered. Python and Java manage with one (plus `is`)." The attack fails because Orthon's explicitness principle *requires* that different semantics have different syntax — conflation of reference and value equality is precisely what causes the bugs Orthon aims to eliminate. The three-operator model is the minimal set that satisfies Explicitness without losing expressiveness.

**Verdict: Pass.** All three operators have precise, non-overlapping definitions. The one apparent contradiction (why both `===` and `==`?) is resolved by their different purposes. The fallback rule creates a clean hierarchy without ambiguity.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Three equality operators are minimal — removing any one makes some necessary comparison pattern inexpressible."

**What is known.** Most mainstream languages have 1-2 comparison operators. Orthon's explicitness principle may require more. The question is how many.

**What is unknown (pre-test).** Whether two operators (structural + semantic, or structural + identity) would cover all necessary use cases.

**Simplest experiment.** Remove `===`: structural comparison becomes dependent on `Eq` trait implementation — every type must opt in to structural comparison. This violates Data First (data should compare structurally without ceremony). Remove `==`: domain-specific equivalence (e.g., Person equality by ID) becomes impossible — every comparison is either structural or identity, with no middle ground. Remove `is`: identity checks for reference types become impossible — a severe limitation for shared-mutable patterns.

**Evaluate the evidence.** Each removal creates an inexpressible gap. Three operators are confirmed as the floor — each serves a distinct, necessary purpose.

**Verdict: Pass.** The hypothesis holds: three operators are minimal. Removing any operator makes some necessary comparison pattern inexpressible.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Equality operates at Level 2 (Language Patterns) in the Semantic Dependency Architecture. (2) It composes Primitive Operations (Level 1) into higher-level comparison patterns. (3) The `Eq` trait is Standard Library (Level 3).

**Deduce the consequences.** `===` generates field-by-field comparison using `pack`/`unpack` primitives — Level 1 composition. `==` delegates to the `Eq` trait (Level 3) with a Level 1 fallback. `is` uses the `reference` primitive (Level 1) for identity checking.

**Test for contradiction.** Does any operator violate layer separation? `===` uses `pack`/`unpack` — both Level 1 primitives. `==` bridges Level 2 (operator) to Level 3 (`Eq` trait) — this is documented layer crossing with clear justification. `is` uses `reference` — Level 1. No operator references constructs above its layer without justification.

**Identify hidden premises.** Does `===` assume a specific data layout to generate field-by-field comparison? Yes — it assumes compound types have known fields. This is already guaranteed by the Semantic Model's Identity dimension (types have fixed structure).

**Verdict: Pass.** All three operators operate cleanly within their architecture layers. The `==` → `Eq` trait bridge is documented and justified.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Structural comparison must know the type's structure (field layout, nesting), which seems strategy-dependent. Yet equality semantics must be strategy-independent.

**Apply separation principles.** **Separation in space**: the *semantic definition* of `===` is "field-by-field structural comparison" — independent of strategy. The *mechanism* (compile-time generated comparison for Default, runtime dispatch for High-Performance, inline expansion for Embedded) is a Strategy choice. **Separation on condition**: observable behaviour (same comparison result for the same values) never changes per Strategy; only the *mechanism* producing that result does.

**Consult known patterns.** This mirrors `DESIGN_PRINCIPLES.md` § Semantics Before Optimization — define *what* equality means before *how* it's implemented. C++'s `operator==` follows the same pattern (semantic definition abstracted from implementation).

**Formulate the abstract solution.** *"Values are equal when their structure is recursively equivalent; each Strategy realises this comparison using its own mechanism without changing the observable result."* All three strategies (Default, Embedded, High-Performance) can implement structural comparison — via monomorphised code generation, trait dispatch, or inline expansion respectively.

**Verdict: Pass.** Semantic definition is fully strategy-agnostic. Mechanism is delegated to the Strategy system.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain each operator in one sentence** (none may contain "and", "but", "except", "however", "unless"):
- `===`: *"This compares two values by their structure, field by field."*
- `==`: *"This compares two values using their type's definition of equivalence."*
- `is`: *"This checks whether two references point to the same object."*

All three pass the conjunction test.

**Explain to a non-expert.** A programmer coming from Python recognises `is` for identity (same as Python). `===` is like Python's `==` *should* work — structurally. `==` is like overriding `__eq__` but explicitly declared. No unfamiliar concepts required.

**Remove one thing.** Removing any operator makes some comparison pattern inexpressible (see CONCEPTUAL_SIMPLICITY_GATE). Three is the floor.

**Check for obviousness.** The three-operator model reads as "the natural way" — structural by default (what data should do), semantic override when needed (business logic), identity for shared references. Each operator's purpose is self-evident from its syntax. No conceptual debt accepted.

**Verdict: Pass.** Three operators are the minimal, obvious, and maintainable model. NaN exclusion is a documented deferral, not a complexity debt.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis** (scan each operator for known LLM-unfriendly patterns):
- `===`: always structural — no context-dependent behaviour. Follows JavaScript convention which LLMs already handle well.
- `==`: always delegates to `Eq` trait — unambiguous. No hidden default that varies by type.
- `is`: always identity — same as Python `is`. Universal across LLM training data.

**Schema round-trip** (can all three operators be expressed in a machine-readable grammar?): Yes — `===`, `==`, and `is` are all binary operators expressible in the abstract grammar with fixed types (`Value, Value -> Bool` for `===`/`==`; `Reference, Reference -> Bool` for `is`).

**Apply the gate's five-criterion table directly:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | All three operators have fixed, unambiguous grammar shapes |
| Predictable generation (≥90%) | Pass | Each operator has exactly one meaning — no "which operator should I use?" ambiguity |
| No hallucination surface | Pass | Every operator has exactly one meaning — no context-dependent semantics |
| Strategy-aware default | Pass | Default-generated code uses `===` (structural), which is valid under any Strategy |
| Self-correctable | Pass | Using `is` on a value type produces a warning; using `==` without `Eq` trait defaults to `===` — both detectable |

**Verdict: Pass.** All five criteria pass cleanly. The three-operator model is one of the most LLM-friendly equality designs across modern languages — each operator has a single, unambiguous meaning.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The three-operator equality model is accepted via EDR-017 with the Transitivity Invariant as a hard constraint and NaN equality deferred to the Standard Library.

**Gates not applied:** None.

---

## Entry: NULL_SAFETY (EDR-018)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/NULL_SAFETY.md`](../../what/concepts/NULL_SAFETY.md)
**Decision recorded as:** [EDR-018](../decision_records/architecture/EDR-018-null-safety.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Null pointer errors — the "billion-dollar mistake." Every reference of type `T` can silently be `null`, turning every dereference into a potential crash. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `Option<T>` type requires compiler-enforced exhaustive matching, nullable syntax (`?.`, `??`, `!`), and type-system integration (a `None` value cannot be assigned to a non-optional `T`). |
| Q3 | Can it be solved with existing primitives? | No. The `?` semantics (optional chaining short-circuit, forced unwrap with panic) require compiler support. `Option` as a sum type requires pattern matching exhaustiveness checking. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (absence is statically tracked), Explicitness (forced unwrap `!` is visible), Minimal Core (one concept — `Option<T>` — replaces an entire class of bugs). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** `Option<T>` introduces sum-type absence tracking. `?.` introduces short-circuit evaluation. `!` introduces a compile-time-checkable unwrap with panic contract. |
| Q6 | Can it be expressed through composition? | No. Pattern matching exhaustiveness requires compiler checking. Optional chaining short-circuit behaviour is not expressible via composition of existing primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Null safety is a semantic guarantee. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential. Eliminates the most common class of runtime crashes. Orthon cannot ship without it. |

**Classification per D-03:** Language. Option type adds `?` semantics not decomposable to primitives. Compiler must track nullable state.

**Primitive decomposition path:** `Option<T>` decomposes to `literal` (None/Some variants) + `pack`/`unpack` + pattern matching; `?.` adds compiler-enforced short-circuit semantics; `??` adds default-value desugaring. The exhaustiveness check adds compiler semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to express "this value may be absent" without risking a null pointer crash. I should never have to ask "can this value be null?" when I dereference it.

**Press release.** *Orthon eliminates the billion-dollar mistake. There is no `null` value in the language. Absence is represented as `Option<T>` — a first-class type the compiler enforces. The `?.` operator chains safely, `??` provides defaults, and `!` offers a visible escape hatch. If it compiles, it's not null.*

**FAQ.**
- *What about interop with languages that have null?* — At the FFI boundary, a foreign null pointer maps to `None`. This is an explicit conversion, not an implicit null leak.
- *What if I'm sure a value is present and don't want to match?* — Use `!` — the forced unwrap. It panics on `None`, so it's a deliberate assertion, not a silent assumption.
- *How is this different from Java's `Optional`?* — Orthon's `Option` is compiler-enforced. Java's `Optional.get()` throws NoSuchElementException at runtime; Orthon's `!` panics with a visible contract.

**Requirements derived.** No `null` sentinel. `Option<T>` as core type. `?.`, `??`, `!` operators. Exhaustive match checking. All satisfied in `what/concepts/NULL_SAFETY.md`.

**Verdict: Pass.** The problem is stated in programmer terms every developer has experienced. Directly serves VISION.md's Comfortable by Design pillar.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `Option<T>` = sum type with variants `Some(T)` and `None`. `?.` = optional chaining (short-circuits to `None` on `None`). `??` = unwrap or default. `!` = forced unwrap (panics on `None`). Each term has a single, precise definition. Distinction from `Result<T,E>` is documented.

**Test with counterexamples.**
- *What happens when `?.` is applied to a non-Option type?* — Compile-time error. `?.` is only valid on `Option<T>`. No silent coercion.
- *Can `None` be assigned to a non-optional variable?* — Compile-time error. `None` is of type `None`, not `T`.
- *What happens if a match on `Option` doesn't cover `None`?* — Compiler rejects the match as non-exhaustive.

**Follow the contradiction.** Apparent tension: `?` on `Result<T,E>` and `?.` on `Option<T>` are different operators. Why not unify? Resolved: they serve different purposes — `?` propagates errors (failure is exceptional), `?.` propagates absence (absence is normal). Different syntax signals different intent.

**Verdict: Pass.** Every term has a single, clear definition. No ambiguity, no context-dependent semantics. The distinction from `Result` is explicit.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "The Option type model is minimal — removing any component makes null safety incomplete."

**What is known.** Many languages manage with runtime null checks (Java, Python). Orthon aims for compile-time safety.

**What is unknown (pre-test).** Whether a subset of the Option model (e.g., `Option` type + match only, without `?.` or `??`) would be sufficient.

**Simplest experiment.** Remove `?.`: accessing a value inside `Option` forces an explicit `match` for every chained access. Deep access chains become deeply nested matches — unacceptable ergonomics. Remove `??`: providing a default value forces an explicit `match` or `unwrap_or` call — more ceremony than necessary. Remove `!`: programmers working with code they know is safe would resort to `match` + `panic` manually — worse ergonomics without semantic change. Remove exhaustiveness checking: matches on `Option` that forget `None` compile silently — null pointer errors return.

**Evaluate the evidence.** Each component removal degrades either safety or ergonomics. All five components (Option type, `?.`, `??`, `!`, exhaustiveness) are necessary.

**Verdict: Pass.** The Option model is confirmed minimal.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) `Option<T>` operates at Level 2 (Language Patterns). (2) It composes primitive operations (`pack`/`unpack` for sum types, `function` for combinators, `call` for invocation) into a higher-level pattern. (3) Exhaustiveness checking is a compiler feature, not a new layer.

**Deduce the consequences.** `?.` desugars to `match` + short-circuit — Level 2 pattern on Level 1 primitives. `??` desugars to `match` + default value. `!` desugars to `match` + `panic`. Exhaustiveness checking adds no new layer — it's an analysis pass on existing pattern matching.

**Test for contradiction.** Does `Option<T>` require Standard Library types for its definition? No — `Option`, `Some`, `None` are core language constructs. The `map`/`and_then` combinators are StdLib, but they call through to core match patterns.

**Verdict: Pass.** Clean layer separation. Core layer defines `Option<T>`; StdLib provides combinators on top.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** `Option<T>` requires runtime representation (discriminated union or nullable pointer), which seems strategy-dependent. Yet null safety semantics must hold regardless of strategy.

**Apply separation principles.** **Separation in space**: the *semantic definition* of `Option<T>` is "a value that is either `Some(T)` or `None`" — independent of representation. The concrete encoding (tagged union, nullable pointer with sentinel, NaN-boxing) is a Strategy choice. **Separation on condition**: observable behaviour (exhaustiveness, short-circuit, panic-on-`!`) never changes per Strategy; only the memory layout does.

**Formulate the abstract solution.** *"A value of type `Option<T>` is either present or absent; each Strategy realises this using its own representation without changing the observable behaviour."* A GC-backed strategy can use a null pointer; an arena strategy can use a discriminator + union.

**Verdict: Pass.** Semantic definition is fully strategy-agnostic. Representation is delegated.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain each operator in one sentence:**
- `Option<T>`: *"A value that may be absent."*
- `?.`: *"If absent, stay absent."*
- `??`: *"If absent, use this default."*
- `!`: *"I know it's present — give me the value."*

All pass the conjunction test.

**Explain to a non-expert.** A programmer coming from Kotlin recognises `?.` (safe call), `?:` (elvis → `??`), and `!!` (force → `!`). Rust programmers recognise `Option<T>` and `unwrap()`. No unfamiliar concepts.

**Remove one thing.** Removing any component degrades the model (see CONCEPTUAL_SIMPLICITY_GATE). All five are necessary.

**Verdict: Pass.** No conceptual debt. Evolution path is clear: more combinators (`and_then`, `or`, `zip`) can be added to StdLib.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `Option<T>` is unambiguous — `Some(T)` wraps, `None` marks absence. `?.` always short-circuits, `??` provides fallback, `!` panics. No context-dependent semantics.

**Schema round-trip:** `Option<T>` is fully expressible as a generic sum type in the type system. All operators have fixed syntax.

**Apply the gate's five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `Option<T>` is a generic sum type — fully expressible in grammar |
| Predictable generation (≥90%) | Pass | `?.` and `??` follow Kotlin/TypeScript patterns LLMs already handle |
| No hallucination surface | Pass | Each operator has exactly one meaning |
| Strategy-aware default | Pass | `Option<T>` is strategy-independent by definition |
| Self-correctable | Pass | Missing match arms are compile-time errors — LLM gets immediate feedback |

**Verdict: Pass.** The Option model is one of the most LLM-friendly null safety approaches. A common LLM mistake (forgetting `None`) is caught by the compiler, making it self-correcting.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The Option-based null safety model is accepted via EDR-018.

**Gates not applied:** None.

---

## Entry: TRAITS (EDR-019)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/TRAITS.md`](../../what/concepts/TRAITS.md)
**Decision recorded as:** [EDR-019](../decision_records/architecture/EDR-019-traits.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Polymorphism without the fragility of class inheritance. How do different types express shared behaviour (interfaces/contracts)? |
| Q2 | Is this a language problem or a library problem? | **Language.** Trait bounds on generics, static vs. dynamic dispatch selection, and the orphan rule require compiler support. The `impl Trait for Type` syntax, `where` clauses, and `dyn Trait` dispatch are parser/type-system features. |
| Q3 | Can it be solved with existing primitives? | No. Trait dispatch (vtable for `dyn`, monomorphisation for static) requires compiler code generation. Trait bound resolution and coherence checking are type-system operations not expressible via primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (behaviour separate from data), Composition Over Inheritance (traits compose via `where T: A + B`), Minimal Core (traits replace multiple constructs: interfaces, abstract classes, mixins). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Nominal trait system: explicit `impl`, coherence (orphan rule), static dispatch via monomorphisation, dynamic dispatch via vtable, trait bounds on generics, associated types. |
| Q6 | Can it be expressed through composition? | No. Trait bounds and dispatch semantics require type-system support. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Polymorphism is a semantic concept. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Required for standard library abstractions (Iterator, Collection, Ord, Eq, etc.). |

**Classification per D-03:** Language. Nominal trait system adds interface semantics not decomposable to primitives. Compiler must resolve trait bounds.

**Primitive decomposition path:** Trait declaration → `function` signatures + `identifier` + `scope` (trait block); `impl` block → `function` implementations + `scope`; `dyn Trait` → `reference` + vtable dispatch; static dispatch → monomorphisation of generics (`function` + type parameters). The coherence rule, bound resolution, and dispatch selection add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to write one function that works with many types that share a behaviour — without duplicating code for each type, and without the fragility of class hierarchies.

**Press release.** *Orthon provides principled polymorphism through traits — behavioural contracts that types explicitly implement. Static dispatch by default means zero overhead. Dynamic dispatch via `dyn Trait` is an opt-in, syntactically visible choice. No fragile base classes, no deep hierarchies, no hidden vtable costs.*

**FAQ.**
- *How is this different from Java interfaces?* — Traits have associated types, default implementations, and support both static and dynamic dispatch. Java interfaces only support dynamic dispatch (vtable).
- *How is this different from Rust traits?* — Orthon traits follow the same model with one difference: the orphan rule is stricter (no downstream implementations of foreign traits on foreign types).
- *Why explicit `impl` instead of structural satisfaction (Go-style)?* — Explicitness principle: a type should declare that it satisfies a contract, not accidentally conform. Structural matching can be opted into via the `structural` keyword (EDR-044).

**Requirements derived.** Trait declaration syntax, `impl` blocks, `where` clauses, `dyn Trait` dispatch, orphan rule, associated types, default methods. All satisfied in `what/concepts/TRAITS.md`.

**Verdict: Pass.** The problem is stated in programmer terms. Directly serves VISION.md's Architectural Integrity pillar (principled polymorphism without hierarchy).

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Trait = method signatures + associated types + optional default implementations. `impl` = concrete method bodies for a specific type. Static dispatch = monomorphisation at compile time. Dynamic dispatch = vtable resolution at runtime. Orphan rule = implementation must be in the same module as trait or type. Each term has a precise, non-overlapping definition.

**Test with counterexamples.**
- *What happens when two traits have a method with the same name for the same type?* — Both implementations coexist. Call-site ambiguity is resolved by fully qualified syntax (`Trait::method(self)`). Explicit, not ambiguous.
- *Can a type implement a trait it doesn't know about (downstream implementation)?* — No — the orphan rule prevents this. The type must be local to the implementing module.

**Follow the contradiction.** Apparent tension: traits provide behaviour without data, but many useful patterns require state. Resolved: data belongs to types (structs, enums), not traits. A trait can require getter methods that access data, but it never owns data itself.

**Verdict: Pass.** Every term has a single, clear definition. No self-referential paradoxes. The orphan rule prevents incoherence.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "The trait system is minimal — removing any component makes polymorphism incomplete."

**What is known.** Alternative approaches (class inheritance, structural typing, typeclasses) each have documented flaws.

**What is unknown (pre-test).** Whether a simplified trait model (e.g., without associated types or default methods) would be sufficient.

**Simplest experiment.** Remove explicit `impl`: requires structural typing (Go model) — violates Explicitness. Remove static dispatch: forces vtable overhead on all generic code — violates Zero-Cost principle. Remove orphan rule: allows incoherent implementations — violates Deterministic Behaviour. Remove associated types: forces traits to carry more generic parameters — increases syntactic noise. Remove default methods: requires separate utility functions for Template Method pattern.

**Evaluate the evidence.** Each removal degrades the model in a measurable way. All components are necessary.

**Verdict: Pass.** The trait system is confirmed minimal.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Traits operate at Level 2 (Language Patterns) in the Semantic Dependency Architecture. (2) They compose Level 1 primitives (`function`, `call`, `scope`, `identifier`) into polymorphism patterns. (3) Trait bounds on generics operate at the type-system level.

**Deduce the consequences.** Trait declaration = `function` signatures + `scope`. `impl` block = `function` implementations + `scope`. Static dispatch = monomorphisation of `function` call. Dynamic dispatch = vtable lookup at `call` site.

**Test for contradiction.** Do traits require Standard Library types? No — traits are defined before any StdLib concepts. The `Eq`, `Ord`, `Iterator` traits are StdLib, but the trait *mechanism* itself is core language.

**Verdict: Pass.** Clean layer separation. Traits are core; trait implementations are StdLib.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Traits need dispatch mechanisms (vtable, monomorphisation) that seem strategy-dependent. Yet trait semantics must be strategy-independent.

**Apply separation principles.** **Separation in space**: the *semantic definition* (behavioural contract) is strategy-independent. The *mechanism* (static monomorphisation, dynamic vtable, compile-time code generation) is a Strategy choice. **Separation on condition**: observable behaviour (which method runs for which type at which call site) never changes per Strategy; only the dispatch mechanism does.

**Verdict: Pass.** Semantic definition is strategy-agnostic. Dispatch mechanism is delegated to Strategy.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain each component in one sentence:**
- `trait`: *"A set of method signatures that types can implement."*
- `impl`: *"Provides the method bodies for a type implementing a trait."*
- `where`: *"Requires that a type parameter satisfies a trait."*

All pass the conjunction test.

**Remove one thing.** Removing any component degrades the model (see CONCEPTUAL_SIMPLICITY). All are necessary.

**Verdict: Pass.** The trait model matches Rust's proven design. No conceptual debt. Evolution path is clear (specialisation, negative bounds).

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `trait`, `impl`, `where T: Trait`, and `dyn Trait` each have a single, unambiguous meaning. No context-dependent syntax.

**Schema round-trip:** Fully expressible in the type system — traits are generic constraints, implementations are named blocks.

**Apply the gate's five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Traits are generic constraints — fully expressible in grammar |
| Predictable generation (≥90%) | Pass | Pattern matches Rust traits, which LLMs generate reliably |
| No hallucination surface | Pass | Each keyword has exactly one meaning |
| Strategy-aware default | Pass | Default dispatch (static via monomorphisation) works under all Strategies |
| Self-correctable | Pass | Missing trait implementations and orphan rule violations are compile-time errors |

**Verdict: Pass.** The trait model follows Rust's established pattern — LLMs already demonstrate reliable generation. Missing implementations are caught by the compiler.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The trait system is accepted via EDR-019.

**Gates not applied:** None.

---

## Entry: ERROR_HANDLING (EDR-020)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/ERROR_HANDLING.md`](../../what/concepts/ERROR_HANDLING.md)
**Decision recorded as:** [EDR-020](../decision_records/architecture/EDR-020-error-handling.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How does a program react to failure without crashing or producing silent corruption? Errors must be visible in the function contract and handling must be composable. |
| Q2 | Is this a language problem or a library problem? | **Language.** `Result<T,E>` is a type with compiler-level propagation mechanism (`?` operator). The compiler must enforce exhaustive handling — unhandled `Result` values are a compile-time error. |
| Q3 | Can it be solved with existing primitives? | No. Short-circuit propagation (`?`) requires compiler support for early return. Exhaustiveness checking on `Result` matches requires pattern-matching completeness checking. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (errors declared in signatures, `?` is visible), Declarative With Static Guarantees (compiler enforces handling), Minimal Core (one concept replaces exceptions, error codes, and checked exceptions). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** `Result<T,E>` is a monadic type with `Ok`/`Error` variants. `?` introduces short-circuit propagation. Combinators (`map`, `and_then`, `or_else`) define error transformation semantics. No exceptions — all fallibility is declared. |
| Q6 | Can it be expressed through composition? | No. `?` operator for automatic propagation is not expressible via composition of existing primitives. Exhaustiveness checking requires compiler support. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `?` could desugar to a `match` + early return, but the pattern-matching exhaustiveness check and type-level `Result` constraint require compiler semantics. |
| Q8 | Is this an optimisation, not semantics? | No. Error handling is a semantic operation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Errors are inevitable; the language must provide a principled mechanism. Result model is the proven modern approach (Rust, Swift, OCaml). |

**Classification per D-03:** Language. `Result<T,E>` is a type with compiler-level propagation mechanism (`?` operator). Not expressible via composition of primitives. Compiler must enforce handling.

**Primitive decomposition path:** `Result<T,E>` → sum type via `pack`/`unpack` + `literal` (Ok/Error variants) + pattern matching; `?` → compiler-enforced short-circuit propagation (match + early return) beyond primitive composition; combinators (`map`, `and_then`, `or_else`) → `function` + pattern matching. Exhaustiveness checking adds compiler semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to call a function that might fail and handle the error without crashing. I should never have to wonder "did I handle that error?" at 3 AM.

**Press release.** *Orthon makes error handling visible and composable. `Result<T, E>` declares fallibility in every function signature. The `?` operator propagates errors concisely. Combinators transform and chain. The compiler ensures every error is handled — if it compiles, no unhandled failure lurks.*

**FAQ.**
- *How is this different from exceptions?* — Errors are values, not hidden control flow. The function signature tells you it can fail. There's no `try-catch` or `throw`.
- *What about unrecoverable errors?* — Use `panic` for bugs and invariants. `Result` is for expected failures.
- *How does this compose with ERROR_UNION?* — ERROR_UNION (EDR-023) extends `Result` with inferred error sets — the same propagation mechanism, richer error typing.

**Verdict: Pass.** Serves VISION.md's Comfortable by Design pillar. Errors are visible, composable, and compiler-enforced.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `Result<T, E>` = sum type with `Ok(T)` and `Error(E)`. `?` = short-circuit propagation (match + early return). Combinators (`map`, `and_then`, `or_else`) = transformation functions. Each term has a precise definition. `Result` is distinct from `Option` — absence vs. failure.

**Test with counterexamples.**
- *What happens when `?` is used in a non-Result function?* — Compile-time error. `?` is only valid in Result-returning functions.
- *Can a `Result` be ignored?* — Compiler warning (configurable to error). Unhandled `Result` values are a diagnostic.
- *What happens when combinators are chained on a `Result`?* — Each combinator returns a new `Result` — the chain continues or short-circuits on first `Error`.

**Verdict: Pass.** All terms precisely defined. Distinction from `Option` is explicit. `?` scope is clearly bounded.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "The Result model is minimal — removing any component makes error handling incomplete."

**Simplest experiment.** Remove `?`: forces explicit `match` at every call site — the Go problem returns. Remove the error type `E`: loses diagnostic information — becomes `Option<T>`, conflating absence with failure. Remove combinators: forces nested `match` blocks for error transformation. Remove exhaustiveness checking: unhandled errors compile silently — null pointer equivalent.

**Verdict: Pass.** All components are necessary. The Result model is minimal.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** `Result<T, E>` operates at the type-system level (generic sum type). `?` composes Primitive Operations (match + return). Combinators are Level 2 StdLib patterns.

**Test for contradiction.** Does `Result` require StdLib types? No — `Result`, `Ok`, `Error` are core language constructs. Combinators are StdLib methods that call through to core match patterns.

**Verdict: Pass.** Clean layer separation. Core layer defines `Result`; StdLib provides combinators.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** `?` propagation requires control-flow manipulation, which seems strategy-dependent. Yet error semantics must be strategy-independent.

**Apply separation.** The *semantic definition* of `?` is "match + early return" — strategy-independent. Stack unwinding (Default), setjmp/longjmp (Embedded), or CPS transformation (High Performance) are all valid mechanisms for the same semantics.

**Verdict: Pass.** Semantic definition is strategy-agnostic. Mechanism is delegated.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"`Result<T, E>` is a type-safe way to say an operation might fail — `?` propagates the error, and combinators transform it."*

**Remove one thing.** Removing `Result` forces the language back to exceptions or error codes. All components necessary.

**Verdict: Pass.** Matches Rust/Swift/OCaml patterns. Evolution path: ERROR_UNION extends `Result` with union error types.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `Result<T, E>`, `Ok(T)`, `Error(E)`, and `?` each have a single, unambiguous meaning. No context-dependent syntax.

**Apply the gate's five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `Result` is a generic sum type — fully expressible |
| Predictable generation (≥90%) | Pass | Pattern matches Rust Result — well-established |
| No hallucination surface | Pass | Each construct has exactly one meaning |
| Strategy-aware default | Pass | `?` propagation works identically under all Strategies |
| Self-correctable | Pass | Unhandled `Result` values are compile-time warnings/errors |

**Verdict: Pass.** The Result model follows Rust's established pattern. Unhandled errors are compiler-detectable.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The Result-based error handling model is accepted via EDR-020.

**Gates not applied:** None.

---

## Entry: LAZY_SEQUENCE_GENERATORS (EDR-021)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/LAZY_SEQUENCE_GENERATORS.md`](../../what/concepts/LAZY_SEQUENCE_GENERATORS.md)
**Decision recorded as:** [EDR-021](../decision_records/architecture/EDR-021-lazy-sequence-generators.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Manually implementing an iterator requires writing a stateful class or object with explicit iteration methods — too much boilerplate. The language should eliminate manual iterator-implementation boilerplate by providing generators and lazy sequences as a core feature. |
| Q2 | Is this a language problem or a library problem? | **Language.** Lazy sequence semantics (`emit`) are a compiler-recognized pattern with special evaluation guarantees (lazy-by-default, per Phase 3 D-06). Not expressible via primitives alone. |
| Q3 | Can it be solved with existing primitives? | No. Lazy evaluation of a generator body — pausing and resuming execution — requires a coroutine or continuation mechanism not present in primitives. The `emit` keyword is a new syntactic form with special semantics. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Intent Over Implementation (programmer describes what to produce, compiler handles state machine), Minimal Core (generators replace manual iterator classes), Explicitness (`emit` makes lazy production syntactically visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Generator functions have lazy evaluation semantics — the body does not execute eagerly but returns an iterator that produces values on demand. The `emit` keyword is a yield-like operation with resumable semantics. Infinite sequences are valid. |
| Q6 | Can it be expressed through composition? | No. Resumable function execution (coroutine/semi-coroutine semantics) is not expressible via composition of existing primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `emit` could desugar to iterator protocol calls, but the state-machine transformation of the generator body requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Lazy production is a semantic guarantee, not an optimization. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for lazy sequence production. Eliminates manual iterator implementation boilerplate. Enables infinite sequences, composition without intermediate allocation, and declarative stream operations. |

**Classification per D-03:** Language. Lazy sequence semantics (`emit`) are a compiler-recognized pattern with special evaluation guarantees (lazy-by-default, per Phase 3 D-06). Not expressible via primitives alone.

**Primitive decomposition path:** Generator function → `function` + state-machine transformation (compiler-generated); `emit value` → iterator protocol `next()` call + suspension/resumption; `return sequence(value)` → iterator completion + value emission; `return value ->` → equivalent desugaring. The state-machine transformation and lazy evaluation semantics add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to produce a sequence of values without writing a stateful class with manual iterator methods. A three-line generator should replace a multi-field stateful class.

**Press release.** *Orthon makes lazy sequence production effortless. Just use `emit` — the compiler handles the state machine, suspension, and resumption. Three canonical forms cover every pattern. Infinite sequences, composition chains, and zero intermediate allocation — all from a single keyword.*

**FAQ.**
- *Is `emit` eager or lazy?* — Lazy by default (Phase 3 D-06). No value is produced until the consumer calls `next()`.
- *Can I return from a generator?* — Yes — `return sequence(value)` completes the generator with a final value. `return;` with no argument terminates the sequence.
- *Can a generator produce infinite values?* — Yes. The language imposes no artificial finite-only constraint.

**Verdict: Pass.** Every programmer who has implemented an iterator manually knows the boilerplate. `emit` is the natural solution.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Generator = function with `emit`. `emit` = produce value + suspend execution. Completion = control flow exits function body. All three canonical forms are provably equivalent (same desugared state machine).

**Test with counterexamples.**
- *What happens when `emit` is used outside a generator?* — Compile-time error. `emit` is only valid in generator functions.
- *Can a generator call another generator?* — Yes — yields are composed. The inner generator produces values into the outer sequence.
- *What happens to the generator state if `next()` is never called?* — No values are produced. The state machine never executes.

**Verdict: Pass.** All terms precisely defined. Three canonical forms are provably equivalent. Lazy-by-default rule is consistent.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "The generator model is minimal — removing any component makes lazy sequence production incomplete."

**Simplest experiment.** Remove `emit`: forces manual iterator class implementation. Remove lazy evaluation: breaks infinite sequences. Remove `Iterator[T]` conformance: forces a separate consumption protocol. Remove stackless compilation: forces heap allocation per generator instance.

**Verdict: Pass.** All components are necessary.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Generators operate at Level 1–2 boundary (Primitive Operations / Language Patterns). `emit` composes `function`, `call`, `scope` into a state-machine pattern. Generators implement `Iterator[T]` (Level 2 trait).

**Test for contradiction.** Do generators require StdLib? No — the generator mechanism is core language. Iterator conformance is specified at the trait level, not the library level.

**Verdict: Pass.** Clean layer separation.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Generators need state-machine transformation (seems strategy-dependent). Yet semantics must be strategy-independent.

**Apply separation.** The *semantic definition* — "a function that produces values on demand" — is strategy-independent. Stackless state machine (Default), stackful coroutine (High Performance), or CPS transformation (Embedded) are all valid.

**Verdict: Pass.** Lazy evaluation guarantee is identical across strategies.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"A generator is a function that produces values one at a time using `emit`, and the compiler handles all the state management."*

**Remove one thing.** Removing generators forces manual iterator implementation everywhere. Necessary.

**Verdict: Pass.** Matches Python `yield`, C# `yield return`. Evolution path: async generators, bidirectional generators.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `emit value` has a single, unambiguous meaning. No context-dependent syntax.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Generator signature `fn name() -> Iterator[T]` is expressible in the type system |
| Predictable generation (≥90%) | Pass | Pattern matches Python `yield` and Rust generators |
| No hallucination surface | Pass | `emit` has exactly one meaning |
| Strategy-aware default | Pass | Lazy evaluation works identically under all Strategies |
| Self-correctable | Pass | Missing `Iterator[T]` return type is a compile error |

**Verdict: Pass.** The `emit` pattern matches established generator syntax across languages.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The lazy sequence generator model is accepted via EDR-021.

**Gates not applied:** None.

---

## Entry: ITERATOR_PROTOCOL (EDR-022)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/ITERATOR_PROTOCOL.md`](../../what/concepts/ITERATOR_PROTOCOL.md)
**Decision recorded as:** [EDR-022](../decision_records/architecture/EDR-022-iterator-protocol.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How does a language provide lazy, composable, memory-efficient iteration over sequences without forcing the programmer to manage iterator state manually? |
| Q2 | Is this a language problem or a library problem? | **Language.** The `Iterator` trait is a protocol definition with special `for` loop desugaring. The compiler must recognize `Iterator[T]` to desugar `for` loops, enforce type constraints, and enable optimisations. Combinators (map, filter, etc.) are StdLib. |
| Q3 | Can it be solved with existing primitives? | No. `for` loop desugaring to `next()` calls requires compiler recognition of the `Iterator` trait. The `IntoIterator` trait for collection-to-iterator conversion requires type-system support. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one protocol covers all iteration: generators, collections, ranges, I/O streams), Orthogonality (Iterator is the consumption side, generators are the production side), Composition (combinators chain without intermediate allocation). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** `Iterator[T]` trait defines a consumption protocol. `for` loop desugaring is compiler-level. Combinators return lazy iterators (one-to-one mapping from source protocol). Single-pass semantics — iterator consumed on traversal. |
| Q6 | Can it be expressed through composition? | No. `for` loop desugaring requires the compiler to recognize `Iterator[T]` and generate the loop-expansion code. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partially. `for` desugars to a loop calling `next()`, but the compiler must know which traits to look for. |
| Q8 | Is this an optimisation, not semantics? | No. Iteration semantics — lazy, single-pass, composable — are semantic properties, not optimisations. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. The iterator protocol is the foundation for all sequence consumption: collections, generators, ranges, I/O streams, and combinator chains. |

**Classification per D-03:** Language. Protocol definition (`next() -> Option[T]`) is a compiler-level concept (trait with special `for` loop desugaring). Combinators should be StdLib.

**Primitive decomposition path:** `Iterator[T]` trait → trait declaration (`trait` + `function` + `identifier`) per TRAITS model; `for item in iter` → loop + `call` to `next()` + pattern match on `Option`; combintors (map, filter, etc.) → `function` implementations on `Iterator[T]` (StdLib, not core). The `for` loop desugaring and range-syntax translation add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to iterate over a sequence and transform it — filter, map, take — without writing temporary collections or manual loops. A four-operation chain should not allocate a single intermediate array.

**Press release.** *Orthon's iterator protocol makes lazy, zero-cost iteration natural. One trait — `Iterator[T]` — unifies all sequence consumption. `for` loops desugar to protocol calls. Combinators chain without allocation. Ranges compile to tight counter loops. Production and consumption are separated: generators produce, iterators consume.*

**FAQ.**
- *What's the difference between `Iterator` and `IntoIterator`?* — `Iterator` is the consumption protocol. `IntoIterator` is how collections expose iterators. `for` accepts `IntoIterator`.
- *Are combinators lazy?* — Yes. No combinator allocates intermediate collections. `.collect()` materialises.
- *Can I iterate over the same collection twice?* — Yes — each call to `.iter()` creates a fresh iterator. Once an iterator is consumed, you call `.iter()` again.

**Verdict: Pass.** Every programmer who has chained `for` loops with intermediate arrays knows the pain. The iterator protocol is the proven solution.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `Iterator[T]` = consumption protocol with `fn next(self) -> Option[T]`. `IntoIterator[T]` = convert-to-iterator. `for` = desugared `next()` loop. Combinators = lazy method implementations. Range = counter-based iterator. `@` = protocol access prefix.

**Test with counterexamples.**
- *What happens when `for` is used on a non-iterable type?* — Compile-time error. Type must implement `IntoIterator`.
- *Can an iterator be used after `next()` returns `None`?* — Behaviour is undefined by the protocol. The iterator is considered consumed.
- *What happens when `@next()` is written without `@`?* — `iterator.next()` accesses the `.next` property, not the protocol method. The `@` prefix distinguishes protocol access from attribute access.

**Verdict: Pass.** All terms precisely defined. `@` prefix provides unambiguous protocol/attribute distinction.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "The iterator protocol is minimal — removing any component makes sequence consumption incomplete."

**Simplest experiment.** Remove `Iterator[T]`: leaves no consumption protocol. Remove `IntoIterator[T]`: forces every `for` loop to call `.iter()` explicitly. Remove combinatorial laziness: every combinator eagerly materialises. Remove `@` protocol access: conflates attribute and protocol access. Remove range expressions: forces manual counter loops.

**Verdict: Pass.** All components are necessary.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** The iterator protocol operates at Level 2 (Language Patterns). It composes Level 1 primitives (`function` for `next()`, `call` for invocation, `loop` for consumption) into a higher-level iteration pattern.

**Test for contradiction.** Does the protocol require StdLib? No — `Iterator` and `IntoIterator` are core language traits. Combinators are StdLib, but the protocol itself is core.

**Verdict: Pass.** Clean layer separation.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Iterator combinators need monomorphisation for zero-cost (seems strategy-dependent). Yet semantics must be strategy-independent.

**Apply separation.** The *semantic definition* — "a value with a `next()` method producing elements until exhausted" — is strategy-independent. Monomorphised tight loop (Default), interpreted dispatch (Development), or vtable-based (Dynamic) are all valid.

**Verdict: Pass.** Semantics are identical across strategies.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"An iterator produces values one at a time via `next()`, and combinators transform those values without allocating collections."*

**Remove one thing.** Removing iterators forces manual loops for all sequence consumption — no lazy composition. Necessary.

**Verdict: Pass.** Matches Rust `Iterator` and Java `Iterator`. Evolution path: `DoubleEndedIterator`, parallel combinators.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `Iterator[T]`, `next()`, `for item in`, ranges (`0..10`), combinator chains, and `@next()` each have a single, unambiguous meaning.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `Iterator` is a trait with a single required method — fully expressible |
| Predictable generation (≥90%) | Pass | Pattern matches Rust's Iterator trait, which LLMs generate reliably |
| No hallucination surface | Pass | Each construct has exactly one meaning |
| Strategy-aware default | Pass | Lazy combinators work identically under all Strategies |
| Self-correctable | Pass | Missing `next()` implementation is a compile error |

**Verdict: Pass.** The iterator protocol matches Rust's proven design. LLMs generate it reliably.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The iterator protocol is accepted via EDR-022.

**Gates not applied:** None.

---

## Entry: ERROR_UNION (EDR-023)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/ERROR_UNION.md`](../../what/concepts/ERROR_UNION.md)
**Decision recorded as:** [EDR-023](../decision_records/architecture/EDR-023-error-union.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Error type declaration and conversion boilerplate for tag-only errors. Most errors are simple identifiers (FileNotFound, Timeout) without payload data, but `Result<T, E>` requires explicit enum declaration and `From` implementations for every error type. |
| Q2 | Is this a language problem or a library problem? | **Language.** Inferred error sets, structural widening from subset to superset, and the `anyerror` escape hatch require compiler support. The `!T` type former is a new kind of type, not expressible via composition of existing constructs. |
| Q3 | Can it be solved with existing primitives? | No. Inferred error set semantics — computing the union of error tags from every fallible call in the function body — require compiler-level call-graph analysis not present in the 9-primitive set. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one new type former replaces error-enum declaration boilerplate), Explicitness (`!T` makes fallibility visible), Intent Over Implementation (programmer writes `!T`, compiler infers the set). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** The `!T` type former is a distinct kind of type — not sugar for `Result<T, E>`. Inferred error sets are a new semantic operation: the compiler discovers, unions, and tracks error tags across the call graph. |
| Q6 | Can it be expressed through composition? | No. Error set inference is inherently compiler-level — the set is derived from the call graph, not composed from primitive constructs. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Error handling semantics (which errors can occur, how they propagate) are semantic, not optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Coexists with `Result<T, E>` from EDR-020. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates the most common source of error-handling boilerplate. Zig's Error Union model has proven production stability. |

**Classification per D-03:** Language. `!T` type former adds inferred error set semantics not decomposable to primitives. Compiler must infer and track error sets.

**Primitive decomposition path:** `!T` → not decomposable — the type former itself is new syntax; error tag literal → `literal` (unit-like tag); error set inference → compiler-level call-graph analysis beyond primitive composition; `?` propagation → `match` + early return (shared with EDR-020's `?` operator). The inference and widening semantics add compiler-level behaviour beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to write a function that can fail with tag-only errors (FileNotFound, Timeout) without declaring an error enum, writing `From` impls, or adding conversion boilerplate for every error type.

**Press release.** *Orthon eliminates error-enum boilerplate for tag-only errors. Write `!T` — the compiler infers the error set from every fallible call. Structural widening means a function returning a subset of errors can be used where a superset is expected. The `?` operator works seamlessly with both `Result<T, E>` and `!T`.*

**FAQ.**
- *What happens when a function returns `!T` and the error set is empty?* — The function never fails. The compiler reports this as an observation, not an error.
- *Can I use `!T` with payload-bearing errors?* — No — use `Result<T, E>` for that. `!T` is for tag-only errors.
- *What is `anyerror`?* — The top error type — all error tags are subtypes of `anyerror`. Used at FFI boundaries and generic error handlers.

**Verdict: Pass.** Serves VISION.md's Comfortable by Design pillar. The common case becomes frictionless.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `!T` = inferred error set type former. Error tag = unit-like value (no payload). Structural widening = subset-to-superset coercion. `anyerror` = top type for error sets.

**Test with counterexamples.**
- *What happens when a function with error set {A, B} is called from a function expecting `!T`?* — The outer set widens to include A, B. No conversion needed.
- *What happens when a function returns `!T` and never calls a fallible function?* — The error set is empty. The compiler notes this.

**Verdict: Pass.** All terms precisely defined. Distinction from `Result<T, E>` is clear.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Error Union reduces cognitive load vs. explicit error enums for tag-only errors."

**Simplest experiment.** Compare code: explicit enum + `From` impls for 5 error tags vs. `!T` with same 5 call sites. The Error Union variant is consistently shorter, and the error set is correct by construction.

**Verdict: Pass.** One new concept (structural widening) eliminates an entire class of boilerplate.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Error Union sits at the same level as `Result<T, E>`. Both are Core Language types. `?` is a Core Language propagation mechanism.

**Test for contradiction.** Does Error Union require `Result<T, E>` concepts? No — it has its own type former, but shares the `?` propagation operator.

**Verdict: Pass.** Clean architecture. Shared `?` operator is consistent.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Inferred error sets tie error handling to a specific compilation strategy (call-graph analysis).

**Apply separation.** The *semantic model* (tag set, widening, propagation) is strategy-independent. The *inference algorithm* is a Strategy choice.

**Verdict: Pass.** Semantics are identical across strategies.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"Inferred error tags that widen structurally — the compiler tracks which errors a function can produce."*

**Remove one thing.** Removing inference forces explicit error-set declarations — the primary ergonomic benefit is lost.

**Verdict: Pass.** Zig has proven this model since 2016. `anyerror` provides clear deprecation path.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `!T` has a single, unambiguous meaning. No context-dependent syntax.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `!T` is expressible in the type system |
| Predictable generation (≥90%) | Pass | Simple rule: "if function can fail with tag-only error, return `!T`" |
| No hallucination surface | Pass | `!T` has exactly one meaning |
| Strategy-aware default | Pass | Set inference works identically under all Strategies |
| Self-correctable | Pass | Empty error sets are compiler-noted |

**Verdict: Pass.** The rule is simple. LLMs can reliably produce `!T` for fallible functions.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The Error Union model is accepted via EDR-023, complementing `Result<T, E>` from EDR-020.

**Gates not applied:** None.

---

## Entry: GENERICS (EDR-024)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/GENERICS.md`](../../what/concepts/GENERICS.md)
**Decision recorded as:** [EDR-024](../decision_records/architecture/EDR-024-generics.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How to write code that works with multiple types without sacrificing type safety, performance, or readability. Generic parameters must be constrained by behavioural contracts (traits). |
| Q2 | Is this a language problem or a library problem? | **Language.** Trait bounds on generic parameters, monomorphisation, variance rules, associated type resolution — all require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Type parameterization — the ability to abstract over types themselves — is not expressible via the 9 primitive operations. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (generics are orthogonal to specific types), Minimal Core (one mechanism — trait-bounded generics — replaces manual type-specific implementations), Explicitness (trait bounds declare constraints visibly in the signature). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Parametric polymorphism adds type-level abstraction — a function parameterised over `T` has different semantics than one specialised to a concrete type. |
| Q6 | Can it be expressed through composition? | No. Type-level abstraction is not expressible via the 9 value-level primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Generics are a semantic concept — abstraction over types. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Enables type-safe collections, algorithms, and abstractions. Required for standard library design. |

**Classification per D-03:** Language. Parametric polymorphism adds new semantics (type parameterization, trait bounds, monomorphisation) not expressible via composition.

**Primitive decomposition path:** Generic function → `function` + type parameters (new abstraction); monomorphised instantiation → `function` + concrete type substitution (compiler transformation); trait bound → `identifier` (trait name) + type-system constraint. The type-level abstraction, bound resolution, and monomorphisation add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to write one function that works with many types that share a behavioural contract — without copying and pasting code for each type, and without sacrificing type safety or performance.

**Press release.** *Orthon generics combine trait-bounded parametric polymorphism with zero-cost monomorphisation. The programmer writes behavioural constraints via `where` clauses; the compiler generates specialised code for each concrete type. No boxing, no type erasure, no runtime casts — just the type safety of static dispatch with the expressiveness of polymorphism.*

**FAQ.**
- *How is this different from Java generics?* — No type erasure. Trait bounds are real constraints, not documentation. Static dispatch by default means zero overhead.
- *How is this different from C++ templates?* — Trait bounds on every type parameter. Error messages refer to unsatisfied bounds, not internal template expansions. Clear, not cryptic.
- *Can I use dynamic dispatch?* — Opt in via `dyn Trait`. Static dispatch is the default for a reason — it's faster and safer.

**Verdict: Pass.** Serves VISION.md's Comfortable by Design and Architectural Integrity pillars.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Type parameter = abstract placeholder constrained by trait bounds. Monomorphisation = per-concrete-type code generation. Variance = how subtyping propagates through generic parameters.

**Test with counterexamples.**
- *What happens when a type parameter has no trait bound?* — Not allowed. Every type parameter must be bounded.
- *Can two different trait bounds conflict?* — Yes — resolved via bound resolution rules. Explicit `where` clauses make conflicts visible.

**Verdict: Pass.** Trait-bounded generics are internally consistent. Monomorphisation produces identical semantics for every instantiation.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Trait bounds + monomorphisation is the simplest generics model providing type safety and performance."

**Compare alternatives.** Erasure (Java): loses type safety. Duck-typed templates (C++): unclear errors. HKT: overly complex for v0.1.

**Verdict: Pass.** Trait-bounded generics with `where` clauses provide the best balance.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Generics sit at the same level as traits. Trait bounds depend on the trait system. Monomorphisation is a compiler code-generation strategy.

**Test for contradiction.** Do generics require StdLib? No — generics and traits are both Core Language concepts.

**Verdict: Pass.** Clean placement in the architecture.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Generics seem tied to monomorphisation, which is an implementation strategy.

**Apply separation.** The *semantic model* (type parameterization with trait constraints) is strategy-independent. The *dispatch mechanism* (monomorphisation vs. boxing vs. dictionary passing) is a Strategy choice.

**Verdict: Pass.** Default is monomorphisation; alternatives are permitted as Strategy profiles.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"Generic functions constrained by traits, specialised at compile time."*

**Remove one thing.** Removing trait bounds would force runtime casting or type erasure — losing Orthon's safety guarantee.

**Verdict: Pass.** Monomorphisation has proven production stability in Rust, C++, and Swift.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `fn name[T: Trait](t: T)` — clear, unambiguous syntax.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Type parameters + trait bounds are fully expressible |
| Predictable generation (≥90%) | Pass | Pattern matches Rust's generics — LLMs generate reliably |
| No hallucination surface | Pass | Each keyword has exactly one meaning |
| Strategy-aware default | Pass | Monomorphisation works under all Strategies |
| Self-correctable | Pass | Missing trait bounds are compile-time errors |

**Verdict: Pass.** LLMs can reliably produce generic code with trait bounds. The compiler catches bound violations.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. The trait-bounded generics model is accepted via EDR-024.

**Gates not applied:** None.

---

## Entry: PATTERN_MATCHING (EDR-025)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/PATTERN_MATCHING.md`](../../what/concepts/PATTERN_MATCHING.md)
**Decision recorded as:** [EDR-025](../decision_records/architecture/EDR-025-pattern-matching.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Cascading `if-else if-else` chains produce poor code — they mix data structure inspection with control flow, the compiler cannot verify exhaustiveness, and missed branches cause bugs. |
| Q2 | Is this a language problem or a library problem? | **Language.** Exhaustiveness checking, destructuring semantics, match ergonomics, and guard evaluation require compiler support. The `match` keyword is a new syntactic form. |
| Q3 | Can it be solved with existing primitives? | No. Exhaustiveness checking — verifying that all variants of a sum type are covered — is a compiler-level analysis not present in the 9-primitive set. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (exhaustiveness), Explicitness (`match` makes branching visible), Minimal Core (one construct replaces if-else chains, manual destructuring, and runtime type checking). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Exhaustiveness checking — compiler verification that all cases are covered. Destructuring — compiler-level decomposition of compound types. Guards — conditional predicates evaluated after structural matching. |
| Q6 | Can it be expressed through composition? | No. Exhaustiveness checking requires the compiler to enumerate type variants — not expressible via composition of primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `match` desugars to a decision tree of `if`/`else` and equality checks, but the exhaustiveness verification and destructuring semantics require compiler support beyond simple desugaring. |
| Q8 | Is this an optimisation, not semantics? | No. Pattern matching is a semantic operation — declarative structure description. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any modern language. Replaces entire categories of bugs (unhandled cases) with compiler-enforced correctness. |

**Classification per D-03:** Language. Exhaustiveness checking, destructuring semantics, match ergonomics. Compiler must verify exhaustiveness.

**Primitive decomposition path:** `match` expression → `function` (match arms as closures) + `call` (pattern evaluation) + `scope` (arm bodies); destructuring → `pack`/`unpack` (value composition/decomposition); guard → `function` (predicate) + `call` (predicate evaluation). The exhaustiveness verification, decision tree compilation, and type-variant enumeration add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to handle every case of a sum type without worrying about missing a variant. I should never have "unhandled case" bugs at 3 AM.

**Press release.** *Orthon's pattern matching is exhaustive — the compiler verifies every variant is covered. Destructuring decomposes values by their structure. Guards handle conditional cases. Or patterns combine arms. If it compiles, every case is handled.*

**FAQ.**
- *What happens if I miss a variant?* — Compile-time error. The compiler lists the missing variants.
- *Can I use a wildcard to catch all remaining cases?* — Yes — `_` matches anything. Use it when you genuinely don't need the other variants.
- *Is `match` an expression?* — Yes — it produces a value. Every arm must return the same type.

**Verdict: Pass.** Serves VISION.md's Comfortable by Design pillar.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Exhaustiveness = every variant covered. Destructuring = pattern-driven decomposition. Guard = conditional predicate. Or pattern = multiple patterns sharing an arm.

**Test with counterexamples.**
- *What happens when two patterns could match?* — First-match precedence. Arms are checked in order; the first to match executes.
- *Can a guard overlap with another arm?* — Yes — guards are checked in order after structural matching. The first matching guard executes.

**Verdict: Pass.** Pattern matching follows from the sum type model. First-match precedence is unambiguous.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Pattern matching replaces if-else chains, manual destructuring, and runtime type checking with a single expression."

**Compare:** `if (x instanceof A) ... else if (x instanceof B) ...` vs. `match x { case A -> ...; case B -> ... }`. The match variant is shorter, unified, and compiler-verified.

**Verdict: Pass.** One expression replaces three separate mechanisms.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Pattern matching depends on the trait system (sealed trait exhaustiveness) and the data model (pack/unpack for destructuring).

**Test for contradiction.** Does matching require StdLib? No — match is a Core Language construct.

**Verdict: Pass.** Clean architecture.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Exhaustiveness checking seems tied to a specific compiler implementation.

**Apply separation.** The *semantic model* (exhaustive matching, destructuring, guards) is strategy-independent. The *decision tree compilation* is a Strategy choice.

**Verdict: Pass.** Semantics are identical across strategies.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"Match values by structure, guaranteed complete by the compiler."*

**Verdict: Pass.** Pattern matching is mature (Rust 2015+, OCaml 1996+, Haskell 1990+). Exhaustiveness prevents the most common enum bugs.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `match x { case A -> ...; case B -> ... }` — simple, unambiguous.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Match expressions are fully expressible in grammar |
| Predictable generation (≥90%) | Pass | Pattern follows established syntax (Rust, Scala) |
| No hallucination surface | Pass | Each construct has exactly one meaning |
| Strategy-aware default | Pass | Exhaustiveness works identically under all Strategies |
| Self-correctable | Pass | Missing cases are compile-time errors — immediate feedback |

**Verdict: Pass.** LLMs can reliably produce match expressions. The Schema Provider exposes variant lists. The compiler catches missing cases.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. Pattern matching is accepted via EDR-025.

**Gates not applied:** None.

---

## Entry: PATTERN_MATCHING_DISPATCH (EDR-026)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/PATTERN_MATCHING_DISPATCH.md`](../../what/concepts/PATTERN_MATCHING_DISPATCH.md)
**Decision recorded as:** [EDR-026](../decision_records/architecture/EDR-026-pattern-matching-dispatch.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | N-way dispatch on multiple arguments requires exponential nested checks. When a function's behaviour depends on the types of multiple arguments simultaneously, single-receiver trait dispatch is insufficient. |
| Q2 | Is this a language problem or a library problem? | **Language.** Definition-site dispatch declaration, specificity resolution, and exhaustiveness across multiple argument patterns require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Dispatch on argument types at the function definition site — generating a dispatch tree from declared argument patterns — is not expressible via the 9-primitive set. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (dispatch is orthogonal to specific type combinations), Explicitness (dispatch variants are visible in the declaration), Minimal Core (one construct replaces nested if-else type checking). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Multimethod dispatch — pattern matching on function arguments at declaration site, resolved at call site. Specificity resolution — deterministic selection of the most specific matching arm. |
| Q6 | Can it be expressed through composition? | No. Dispatch on multiple argument types simultaneously is not expressible via single-receiver trait dispatch. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — pattern matching dispatch desugars to nested pattern matching on each argument, but the exhaustiveness checking across argument combinations requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Dispatch semantics — which implementation runs for a given set of argument types — is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates the most egregious nested type-check boilerplate — N-way dispatch. Complements trait dispatch for multimethod scenarios. |

**Classification per D-03:** Language. Multimethod dispatch — pattern matching applied to function arguments at definition site, resolved at call site.

**Primitive decomposition path:** `match` declaration form → `function` (dispatch function) + `match` (per EDR-025) + `scope` (arm bodies); argument pattern → `identifier` (type name) + `pack`/`unpack` (destructuring); specificity resolution → compiler-level pattern comparison. The dispatch tree generation, specificity analysis, and cross-argument exhaustiveness add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to dispatch on multiple arguments without writing nested `if instanceof` chains. One declaration should generate the entire dispatch tree.

**Press release.** *Orthon extends pattern matching to function declaration. Declare `fun process(match a: Type, match b: Type) { case ... -> }` and the compiler generates the dispatch tree, verifies exhaustiveness, and resolves specificity. No more manual `instanceof` chains.*

**FAQ.**
- *When should I use this vs. traits?* — Use traits for single-receiver dispatch. Use pattern dispatch for multimethod scenarios (dispatch on multiple arguments).
- *Is dispatch deterministic?* — Yes — specificity resolution always selects the same arm for the same argument types.

**Verdict: Pass.** Eliminates nested type-check chains.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Dispatch = selecting which arm runs based on argument types. Specificity = which pattern is "more specific" (structural comparison). Exhaustiveness = all argument type combinations covered.

**Verdict: Pass.** Direct extension of value-level pattern matching (EDR-025) to function arguments.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Definition-site dispatch replaces nested type checks with a single declaration."

**Verdict: Pass.** One declaration form expresses all dispatch variants. Exhaustiveness ensures coverage.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Depends on pattern matching (EDR-025) for pattern semantics and traits (EDR-019) for type hierarchy.

**Verdict: Pass.** Sits at the same Core Language level as pattern matching.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Dispatch resolution seems tied to a specific compiler.

**Apply separation.** The *semantic model* (argument patterns, specificity, exhaustiveness) is strategy-independent. The *dispatch tree compilation* is a Strategy choice.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"Dispatch by matching arguments, verified exhaustive."*

**Verdict: Pass.** Proven in CLOS, Julia, and Scala. Definition-site declaration provides local reasoning.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Pattern matching dispatch follows the same syntax as value-level matching.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Dispatch declarations are expressible in grammar |
| Predictable generation (≥90%) | Pass | Pattern consistent with value-level matching |
| No hallucination surface | Pass | Each construct has exactly one meaning |
| Strategy-aware default | Pass | Dispatch semantics are strategy-independent |
| Self-correctable | Pass | Coverage gaps are compile-time errors |

**Verdict: Pass.** LLMs can generate dispatch functions by enumerating argument type combinations.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. Pattern matching dispatch is accepted via EDR-026.

**Gates not applied:** None.

---

## Entry: TYPE_INFERENCE (EDR-027)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/TYPE_INFERENCE.md`](../../what/concepts/TYPE_INFERENCE.md)
**Decision recorded as:** [EDR-027](../decision_records/architecture/EDR-027-type-inference.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Type annotations inside function bodies add noise without providing documentation value. The programmer must spell out types that are obvious from context. |
| Q2 | Is this a language problem or a library problem? | **Language.** Type inference is a compiler-level type-system service — it determines types from expression context. The bidirectional inference algorithm, generic type argument inference, and the annotation boundary rule all require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Type inference is the type system determining types from usage — this is a meta-level operation, not expressible via value-level primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (annotations at API boundaries), Simplicity (inference inside functions reduces noise), Intent Over Implementation (programmer describes what, compiler resolves types). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Type inference is a compiler service — the compiler determines types that are not explicitly written. Type unification depends on EQUALITY (EDR-017) semantics. |
| Q6 | Can it be expressed through composition? | No. Type inference is inherently compiler-level — the type system determines types from context. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Type inference determines the semantic type of an expression — it is a semantic operation, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ergonomic use of generics, lambda expressions, and local variable declarations. Without inference, Orthon would require type annotations on every expression — unacceptable for LLM readability and programmer productivity. |

**Classification per D-03:** Language. Compiler-level semantic service (local bidirectional inference). Type annotations required at public API boundaries. Depends on EQUALITY (EDR-017) for type unification.

**Primitive decomposition path:** Inferred type → compiler-determined, not primitive-expressible; type annotation at API boundary → `identifier` (type name) + `scope` (binding); type unification → `===` (structural equality per EDR-017) applied to type structures. The inference algorithm, constraint solving, and type unification are compiler-level services beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to write `let x = items.map(fn (i) -> i * 2)` without annotating every intermediate type. Annotations should only be needed at module boundaries — where they serve as documentation.

**Press release.** *Orthon's local bidirectional inference eliminates type annotation noise inside function bodies. The compiler infers types from expressions and context. Annotations required at public API boundaries only — where they document contracts for consumers.*

**FAQ.**
- *What happens when inference fails?* — The compiler requests an explicit annotation at the failure point. Clear error, not a cryptic type mismatch deep in the expression.
- *Can inference cross module boundaries?* — No. Inference is per-function body. Module boundaries require explicit annotations.

**Verdict: Pass.** Serves VISION.md's Comfortable by Design pillar. Less annotation noise, same safety.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Bidirectional inference = bottom-up (from expressions) + top-down (from context). Type unification = structural comparison of types. Annotation boundary = module API surface.

**Verdict: Pass.** No circularity — inference is well-founded on the expression tree.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Local bidirectional inference provides the best balance of ergonomics and explicitness."

**Compare:** Global inference (cryptic errors), full annotation (verbose), gradual (lost guarantees). Local bidirectional with explicit boundaries is the proven middle path (Rust, Kotlin).

**Verdict: Pass.**

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Type inference is a Core Language type-system service. It depends on EQUALITY (EDR-017) for type unification.

**Verdict: Pass.** Independent of the Syntax layer — semantic model is defined before concrete annotation syntax.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Bidirectional inference seems tied to a specific algorithm.

**Apply separation.** The *semantic specification* (local, bidirectional, no cross-module) is strategy-independent. The *inference algorithm* (HM, unification-based, constraint-based) is a Strategy choice.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**Explain in one sentence:** *"Infer types inside functions; require annotations at API boundaries."*

**Verdict: Pass.** Proven stability in Rust, Kotlin, and C#. Explicit boundaries prevent inference fragility.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Simple rule — "inside = inferred, boundary = annotated."

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Inference rules are expressible in type system |
| Predictable generation (≥90%) | Pass | Simple rule for LLMs: inside inferred, boundaries annotated |
| No hallucination surface | Pass | Inference has no context-dependent semantics |
| Strategy-aware default | Pass | Inference works identically under all Strategies |
| Self-correctable | Pass | Inference failures request explicit annotation |

**Verdict: Pass.** LLMs can omit local annotations; Schema Provider exposes expected types at boundaries.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. Type inference is accepted via EDR-027.

**Gates not applied:** None.

---

## Entry: TYPE_LEVEL_NULL_SAFETY (EDR-028)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/TYPE_LEVEL_NULL_SAFETY.md`](../../what/concepts/TYPE_LEVEL_NULL_SAFETY.md)
**Decision recorded as:** [EDR-028](../decision_records/architecture/EDR-028-type-level-null-safety.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | After a pattern match on `Option<T>` that establishes the value is `Some(T)`, the programmer should not need to manually unbox. Without narrowing, every pattern match requires explicit `!` unwrap. |
| Q2 | Is this a language problem or a library problem? | **Language.** Flow-sensitive type narrowing — tracking type information across control flow edges — requires compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Flow-sensitive type analysis — tracking that a variable of type `Option<T>` is known to be `T` in a specific code path — is not expressible via the 9-primitive set. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (narrowing is compiler-enforced safety), Explicitness (narrowing follows visible checks), Minimal Core (narrowing eliminates manual unwrap calls without adding new syntax). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Flow-sensitive type narrowing — the compiler tracks per-variable type information across control flow edges. |
| Q6 | Can it be expressed through composition? | No. Flow-sensitive type analysis is inherently compiler-level — the type system must track state across control flow. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Type narrowing determines what operations are legal on a value — this is a semantic guarantee, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Builds on NULL_SAFETY (EDR-018). |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ergonomic null safety. Without narrowing, every `Some` match would require an explicit `!` — eliminating the ergonomic benefit of pattern matching. |

**Classification per D-03:** Language. Null safety tracked at type level (`Option<T>` vs `T`). Compiler tracks when a value is definitely non-null after a check. Depends on NULL_SAFETY (EDR-018).

**Primitive decomposition path:** Narrowed type → compiler-determined, not primitive-expressible; `match` narrowing → pattern matching (EDR-025) + compiler type tracking; `if` check narrowing → `function` (condition) + compiler type tracking across control flow edges. The flow-sensitive type tracking across control flow edges is a compiler-level analysis beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to match `Some(x)` and use `x` directly without calling `x!` or `x.unwrap()`. The compiler should know it's safe.

**Press release.** *Orthon's type-level null safety tracks narrowings through control flow. After `if value != None`, the compiler knows `value` is non-null in the true branch. No more `!` calls after every check.*

**Verdict: Pass.** Serves Comfortable by Design. Safe code is as concise as unsafe code.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Narrowing = type refinement after pattern match. Flow-sensitive = per-variable, per-control-flow-edge type tracking.

**Verdict: Pass.** Narrowing follows from pattern matching semantics. No self-referential paradoxes.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** "Narrowing after match reduces unboxing boilerplate without sacrificing safety."

**Compare:** With narrowing: `match x { Some(v) => process(v) }`. Without: `match x { Some(v) => process(v!) }`. Narrowing wins — safer and shorter.

**Verdict: Pass.**

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Builds on NULL_SAFETY (EDR-018) by adding narrowing semantics. Core Language type-system service.

**Verdict: Pass.**

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Narrowing seems tied to a specific type-system implementation.

**Apply separation.** Semantic specification (narrowing after match, flow-sensitive) is strategy-independent. Algorithm is a Strategy choice.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *"After matching `Some(x)`, the type is narrowed — no manual unboxing needed."*

**Verdict: Pass.** Conservative narrowing by construction. Proven in Rust and Kotlin.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Narrowing rules are expressible in type system |
| Predictable generation (≥90%) | Pass | Simple rule: match Some → value is non-null |
| No hallucination surface | Pass | Narrowing has no context-dependent semantics |
| Strategy-aware default | Pass | Works identically under all Strategies |
| Self-correctable | Pass | Mis-narrowed values are compile-time errors |

**Verdict: Pass.** LLMs can rely on narrowing without explicit annotations.

---

### Overall

All seven gates Pass outright. No gate returned Flag or Fail. Type-level null safety is accepted via EDR-028.

**Gates not applied:** None.

---

## Entry: AST_MACROS (EDR-029)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/AST_MACROS.md`](../../what/concepts/AST_MACROS.md)
**Decision recorded as:** [EDR-029](../decision_records/architecture/EDR-029-ast-macros.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Metaprogramming — code that writes code — without introducing multiple special-purpose sublanguages (macro_rules!, proc macros, annotation processors). |
| Q2 | Is this a language problem or a library problem? | **Language.** AST macros operate on compiler-level AST type nodes. The `@macro` annotation, `@derive` sugar, hygienic scoping, and single-pass expansion require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. AST node manipulation (typed AST types, AST construction) is not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one macro mechanism replaces multiple sublanguages), Explicitness (`@macro` is syntactically visible), Orthogonality (macros compose freely with other constructs). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Typed AST node manipulation at compile time, hygienic scoping, single-pass expansion, `@derive` resolution. |
| Q6 | Can it be expressed through composition? | No. Compiler-level AST types and macro expansion ordering are not expressible via primitive composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `@derive` could desugar to `@macro` invocations, but the macro mechanism itself requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Macro expansion is a semantic operation (code generation), not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for metaprogramming. Eliminates manual code duplication for trait implementations. `@derive` alone justifies the mechanism. |

**Classification per D-03:** Language. Operate on parse tree at compile time, requiring compiler-level understanding. Builds on COMPILE_TIME_EXECUTION.

**Primitive decomposition path:** `@macro` function → `function` + comptime annotation (compiler-recognized); `@derive(Trait)` → compiler-resolved macro registry lookup; AST types → compiler-internal type system (not user-visible beyond macro API). The macro registry, AST type contracts, and expansion ordering add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon language designer, I need a metaprogramming mechanism that lets library authors generate boilerplate code (trait implementations, serialisation, etc.) without requiring a separate macro-definition language or compiler plugin system.

**Press release.** *Orthon macros are just functions — any `@macro`-annotated function receives typed AST nodes and returns typed AST nodes. The most common pattern (`@derive`) has dedicated declarative syntax, and macros build on the unified comptime mechanism so no separate execution engine is needed.*

**FAQ.**
- *Why not pure comptime without macros?* — Comptime alone lacks structured contracts for code generation. Macros provide explicit input/output type signatures.
- *Why typed AST nodes instead of token streams?* — Typed AST nodes enable type-level validation of macro inputs and outputs; raw token streams (Rust proc macros) require separate parsing and validation.
- *What about nested macros?* — Single-pass expansion prevents macros that generate macro invocations, a deliberate simplification that eliminates phase-ordering bugs.

**Verdict: Pass.** Macros solve a real user need with minimal mechanism.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Macro (annotated function executing at comptime), typed AST node (compiler-level type representing syntactic structure), hygienic scoping (macro-introduced identifiers scoped to expansion site), single-pass expansion (all macros expanded once before type checking).

**Test with counterexamples.** Counterexample: a macro that generates a `@macro` invocation in its output — fails because single-pass expansion prohibits generating new macros. This is a deliberate boundary, not a contradiction. Counterexample: unhygienic access without `#` prefix — fails because hygiene enforcement catches it.

**Verdict: Pass.** Hygienic scoping, single-pass expansion, and typed AST contracts form an internally consistent model.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One mechanism (`@macro` function) can replace multiple special-purpose metaprogramming sublanguages (macro_rules!, proc macros, annotation processors).

**Simplest experiment.** Implement `@derive(Show)` as a `@macro` function that reads a `TypeDef` AST node and returns an `ImplBlock` AST node. Verify no other metaprogramming mechanism is required.

**Verdict: Pass.** Cannot be expressed via composition of primitives — AST manipulation requires compiler-level AST types.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Macros operate within the compiler pipeline after parsing, before type checking. (2) Macro functions are compiled via the comptime mechanism (EDR-031). (3) The `@macro` annotation is orthogonal to existing function semantics.

**Verdict: Pass.** Macros slot into the compiler pipeline without layering violations. Building on comptime ensures no separate execution engine.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Macros must manipulate compiler-internal AST types, which are inherently implementation-specific, yet macro semantics (hygiene, expansion order) must be implementation-independent.

**Apply separation.** Separate *what* (semantic specification: hygiene rules, expansion order, input/output contracts) from *how* (AST representation, macro execution engine). The comptime execution engine IS an implementation detail — macro semantics remain invariant across strategies.

**Verdict: Pass.** Macro semantics (hygiene, expansion order, AST types) are strategy-independent. The comptime execution engine is a compiler concern, not an Implementation Strategy concern.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *"The macro system must support adding new AST types and derive targets without breaking existing macros."*

**Verdict: Pass.** Evolution path clear: new AST types can be added; `@derive` registry is extensible. Single-pass expansion avoids future phase-ordering complexity.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Macro contracts (input type, output type, hygiene rules) are explicit and schema-exposable. `@derive` is purely declarative.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | AST types are compiler-defined types with known schema. `@macro` signature (input type → output type) is schema-exposable. |
| Predictable generation (≥90%) | Pass | Common patterns (`@derive`) are declarative; macro definitions follow function structure. |
| No hallucination surface | Pass | AST type system constrains macro output to valid nodes. Compiler type-checks macro output after expansion. |
| Strategy-aware default | Pass | Default expansion strategy (single-pass, type-checked) applies across all strategies. |
| Self-correctable | Pass | Type errors in macro output produce structured diagnostics with repair hints. |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright. Macros as typed AST functions provide a safe, predictable metaprogramming mechanism that builds on comptime and requires no separate macro-definition sublanguage.

**Gates not applied:** None.

---

## Entry: COMPILER_AS_STATIC_ANALYZER (EDR-030)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/COMPILER_AS_STATIC_ANALYZER.md`](../../what/concepts/COMPILER_AS_STATIC_ANALYZER.md)
**Decision recorded as:** [EDR-030](../decision_records/architecture/EDR-030-compiler-as-static-analyzer.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | The line between compiler errors, warnings, and linter concerns must be explicitly defined. LLM-native design requires a single feedback channel for all static checks. |
| Q2 | Is this a language problem or a library problem? | **Language.** The compiler IS the analyzer — verification layers are part of the compiler pipeline, not an external tool. |
| Q3 | Can it be solved with existing primitives? | No. Verification layers (ownership, effects, exhaustiveness) require compiler-level semantic analysis not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (effects tracking requires declared effect boundaries), Declarative With Static Guarantees (compiler enforces correctness before runtime), LLM Readiness (single diagnostic channel). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Each verification layer adds compiler-enforced semantic guarantees: ownership, effect tracking, exhaustiveness, pattern-match completeness. |
| Q6 | Can it be expressed through composition? | No. Ownership analysis, effect tracking, and exhaustiveness checking are compiler-level analyses, not compositions of primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Verification establishes semantic correctness. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for a safe, LLM-native language. The compiler as single verification channel provides the fastest feedback loop. |

**Classification per D-03:** Language. Compiler provides static analysis API — the compiler IS the analyzer. Meta-concept — not a feature programmers invoke, but the architecture of how the compiler verifies correctness.

**Primitive decomposition path:** Not directly applicable — the static analyzer is the compiler itself. Verification layers are meta-operations on the compiler pipeline, not decomposable to user-visible primitives.

### Gate Validation

**Gates applied:** All 7 (a meta-concept defining how the compiler verifies correctness requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want the compiler to catch as many bugs as possible before running my program, from syntax errors to ownership violations to contract breaches — all from a single `build` or `check` command, without configuring or running separate linters.

**Press release.** *The Orthon compiler IS your static analyzer — seven progressive verification layers catch errors from syntax to contracts in a single pipeline. No separate linter invocation, no tooling configuration, no LLM feedback loop outside the compiler diagnostics.*

**FAQ.**
- *Does this mean I can't use external linters?* — External linting is possible at Layer 7 (extension analyses), but not required for any standard check.
- *What if I want fast feedback during prototyping?* — `--relaxed` mode skips deeper layers (6–7) while preserving syntax, type, and ownership guarantees.
- *How does this help LLM workflows?* — One invocation (compile) produces all diagnostics in structured format — no separate LLM-linter integration needed.

**Verdict: Pass.** Programmers get fast, unified feedback. LLM gets single-channel diagnostics.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Verification layer (a named stage in the compiler pipeline that checks a specific class of properties), guaranteed analysis (layers 1–6, always enabled), extension analysis (layer 7, opt-in), undefined behaviour (program behaviour not fully specified by the language), `--relaxed` mode (skips layers 6–7 for prototyping).

**Test with counterexamples.** Counterexample: a program that passes layers 1–5 but fails layer 6 (exhaustiveness) — correctly rejected. Counterexample: a program that passes exhaustive checks in one Implementation Strategy but not another — impossible because layers 1–6 are strategy-independent requirements.

**Verdict: Pass.** Layers are ordered by dependency; each layer builds on the previous without contradiction.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Seven progressive verification layers built into the compiler provide better correctness guarantees than a minimal compiler + external linters.

**Simplest experiment.** Implement layers 1–3 (syntax, name resolution, type checking) as the minimum viable compiler. Compare developer experience (single vs. multi-tool workflow) with and without built-in analysis.

**Verdict: Pass.** Progressive layers are a well-known pattern (Rust, Swift, Haskell). One mechanism (compiler pipeline) replaces compiler + linter + CI checks.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) The compiler defines what a valid Orthon program is. (2) Static analysis checks program validity against the specification. (3) Therefore the compiler is the natural owner of all verification.

**Verdict: Pass.** Verification layers are part of the Core Language — every Implementation Strategy must support them. No layering violation.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Verification checks must be strategy-independent (a program that passes should pass everywhere), but the *implementation* of each check necessarily depends on the compiler's internal representation.

**Apply separation.** Separate *what* (semantic specification of each layer: what properties it checks, what it rejects) from *how* (the algorithm implementing each check). The semantic specification is strategy-independent; only the algorithm differs per strategy.

**Verdict: Pass.** The semantic specification of each layer is strategy-independent; only the implementation (check algorithm) differs per strategy.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *"The verification pipeline must accommodate new checks over time without requiring changes to the existing layer structure or breaking programs that pass current checks."*

**Verdict: Pass.** Layers can be extended independently. New checks can be added at existing layers or as new layers without breaking existing code.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** The compiler IS the analyzer — LLM sees all diagnostics from one invocation. Structured error format eliminates parsing ambiguity.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Diagnostic format is structured (error code, location, repair hint) by specification. |
| Predictable generation (≥90%) | Pass | The compiler produces diagnostics deterministically — same input, same output. |
| No hallucination surface | Pass | No generative component — every diagnostic corresponds to a specific check. |
| Strategy-aware default | Pass | Default mode runs layers 1–6; this applies uniformly across all strategies. |
| Self-correctable | Pass | LLM receives structured diagnostics with repair hints — can generate corrected code. |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright. The compiler-as-static-analyzer model provides the fastest possible verification feedback loop for both human and LLM programmers, with progressive layers that prevent verification from being optional.

**Gates not applied:** None.

---

## Entry: COMPILE_TIME_EXECUTION (EDR-031)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/COMPILE_TIME_EXECUTION.md`](../../what/concepts/COMPILE_TIME_EXECUTION.md)
**Decision recorded as:** [EDR-031](../decision_records/architecture/EDR-031-compile-time-execution.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Generics, reflection, and metaprogramming should not each require their own sublanguage. A single compile-time execution mechanism replaces four separate mechanisms. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `comptime` keyword, comptime parameter semantics, and comptime evaluation model require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Compile-time evaluation of the same language semantics is a new execution phase, not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one mechanism replaces four), Same Semantics Earlier Phase (no separate sublanguage), Explicit Semantics (`comptime` keyword makes phase visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Comptime evaluation phase, comptime parameters, type as first-class comptime value, deterministic sandboxed execution, reflection operations. |
| Q6 | Can it be expressed through composition? | No. A second execution phase (compile time vs. runtime) is a fundamental semantic addition, not a composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Comptime defines a new execution phase with semantic consequences (code runs at compile time vs. runtime produces different observable behaviour). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Unified comptime is the elegant answer to generics + reflection + metaprogramming. Cross-ref with GENERICS (Plan 04-02) — comptime IS the generic mechanism. |

**Classification per D-03:** Language. Unified comptime model (Zig-inspired). Same semantics, earlier phase. Compiler-level execution mode.

**Primitive decomposition path:** Comptime parameter → `function` parameter + comptime annotation; comptime block → `scope` + comptime annotation; `@typeInfo` → comptime-evaluated `call` to compiler intrinsic; monomorphisation → compiler specialization of `function` + types. The comptime execution phase and evaluation engine add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new language execution phase requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want one mechanism for generics, reflection, and metaprogramming — writing code that runs at compile time using the same syntax and semantics as runtime code — so I don't need to learn four separate sublanguages.

**Press release.** *Orthon's unified `comptime` model replaces generics, reflection, and metaprogramming with one familiar-looking mechanism: `comptime` parameters execute at compile time using the same language as runtime code, with explicit trait bounds on public APIs for LLM discoverability.*

**FAQ.**
- *Does comptime mean I always need generics syntax?* — `comptime` replaces `<T>` syntax: `comptime T: type` is the Orthon way.
- *How do I do reflection?* — `@typeInfo(T)`, `@field(value, name)` are comptime-evaluated function calls, not a separate API.
- *Can comptime code do IO?* — No — comptime is deterministic and sandboxed from IO.

**Verdict: Pass.** Programmers get generics, reflection, and metaprogramming from one familiar-looking mechanism.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Comptime (compile-time execution phase using same semantics as runtime), comptime parameter (function parameter resolved at compile time), trait bound (explicit constraint on a comptime parameter), monomorphisation (compiler specialization of generic code), reflection (comptime-evaluated operations on type metadata).

**Test with counterexamples.** Counterexample: a private function with duck-typed comptime parameters compiles but its callers don't know the parameter constraints — this is a deliberate trade-off (internal code has lighter requirements). Counterexample: a comptime block that tries to open a file — rejected because comptime is sandboxed.

**Verdict: Pass.** "Same semantics, earlier phase" is internally consistent. Hybrid bounds regime (public required, private optional) has clear rules.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One unified comptime model can replace four separate mechanisms (declared generics, structural typing, metaobjects, reflection alternatives).

**Simplest experiment.** Implement a generic `max(T, T) -> T` function using `comptime T: type + Comparable`. Implement `@typeInfo` reflection on a struct fields. Implement a `toJSON` metaprogram using comptime loop over struct fields. Verify all three use the same syntactical and semantic rules.

**Verdict: Flag.** The hybrid bounds regime (public MUST have bounds, private MAY omit) adds mild complexity over a pure duck-typed or pure declared model. Acceptable given the LLM discoverability benefit.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Comptime is a second execution phase (compile time and runtime), not a new semantic domain. (2) The `comptime` keyword is orthogonal — it modifies existing constructs (parameters, blocks) without introducing new structural patterns. (3) Comptime operates within the Core Language layer.

**Verdict: Pass.** Comptime operates within the Core Language layer. The `comptime` keyword is orthogonal to existing constructs.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Comptime execution requires a compile-time evaluation engine, which is inherently implementation-specific, yet comptime *semantics* must be implementation-independent.

**Apply separation.** Separate *what* (comptime semantics: deterministic, sandboxed, same language, parameter bounds) from *how* (the evaluation engine: interpreter, compiled-and-run, JIT). The semantic specification refers only to inputs and outputs, not to evaluation strategy.

**Verdict: Pass.** Comptime semantics (deterministic, sandboxed, same language) are strategy-independent. Implementation Strategy only affects *how* comptime is executed (interpreter, compiled-and-run, JIT).

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *"The comptime model must accommodate new compile-time operations and potential tightening of private duck-typed bounds without breaking existing comptime code."*

**Verdict: Pass.** Evolution path clear: new comptime operations can be added. Private duck-typed regime can be tightened in future if needed.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Public API bounds requirement ensures LLM can determine contracts from signatures. Private duck-typed parameters are a known risk — Schema Provider MUST expose bounds info.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Comptime parameter bounds are explicit in function signatures — Schema Provider can expose them. |
| Predictable generation (≥90%) | Pass | Public API generics have explicit bounds; LLM generates correct constraint patterns reliably. |
| No hallucination surface | Flag | Private duck-typed parameters lack visible constraints — LLM may generate invalid internal code that passes bounds check only at comptime expansion time. Mitigated by Schema Provider exposure. |
| Strategy-aware default | Pass | Default bounds regime (public required, private optional) is consistent across strategies. |
| Self-correctable | Pass | Comptime type errors (bounds violations, sandbox violations) produce structured diagnostics. |

**Verdict: Critical —** with mitigation (Schema Provider must expose bounds info). Restricted to visible comptime constructs.

---

### Overall

Six gates Pass outright; one gate (Conceptual Simplicity) flags mild complexity from the hybrid bounds regime, and one gate (LLM Generability) is marked Critical due to private duck-typed parameter risk, mitigated by Schema Provider exposure. The unified comptime model is accepted with the understanding that the hybrid bounds regime is a deliberate trade-off for LLM discoverability.

**Gates not applied:** None.

---

## Entry: COMPOSABLE_COLLECTION_OPS (EDR-032)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/COMPOSABLE_COLLECTION_OPS.md`](../../what/concepts/COMPOSABLE_COLLECTION_OPS.md)
**Decision recorded as:** [EDR-032](../decision_records/architecture/EDR-032-composable-collection-ops.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Manual index-based loops, empty accumulator lists, and explicit search flags force the programmer to describe *how* instead of *what*. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** `.map()`, `.filter()`, `.reduce()` are compositions of `Iterator[T].next()` calls. No new language semantics required — the Iterator Protocol (EDR-022) provides everything. |
| Q3 | Can it be solved with existing primitives? | Yes. Each combinator is implementable as a method on `Iterator[T]` using existing `function`, `call`, `scope`, and `pack`/`unpack` primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (StdLib is the right home), Intent Over Implementation (declarative combinators over imperative loops). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Each combinator is a function composition. Loop fusion is an optimisation, not semantics. |
| Q6 | Can it be expressed through composition? | Yes — of `Iterator[T].next()` calls. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — combinators are function calls, fully expressible via primitive operations. |
| Q8 | Is this an optimisation, not semantics? | The operations themselves are semantic (map, filter, reduce). Loop fusion (combining multiple passes) is a pure optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Declarative collection operations are essential for any practical language. StdLib classification means zero language additions. |

**Classification per D-03:** StdLib. Combinators are compositions of ITERATOR_PROTOCOL operations. Note compiler-level optimization (loop fusion) is an Implementation Strategy concern.

### Gate Validation

**Gates applied:** All 7 (a feature classified as StdLib still requires full validation — the decision to classify as StdLib IS a language-level decision, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to transform collections without writing index-based loops, accumulator lists, or explicit search flags — I want to say *what* (map, filter, reduce) instead of *how* (for, if, accumulate).

**Press release.** *Orthon's Standard Library provides lazy composable collection operations — `.map()`, `.filter()`, `.reduce()`, and more — building on the Iterator Protocol (EDR-022). Zero language additions needed: every combinator is a method on `Iterator[T]`.*

**FAQ.**
- *Why no comprehension syntax?* — Comprehensions are deferred to v0.2+ as syntactic sugar over combinator chains. The combinator API is sufficient for v0.1.
- *Are combinators lazy or eager?* — Lazy by default (per Phase 3 D-06). Explicit `.collect()` materialises results.
- *Can I add my own combinators?* — Yes — any Iterator extension method is a valid combinator.

**Verdict: Pass.** Combinators solve the real problem of imperative loop boilerplate (see crutch analysis in research doc).

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Combinator (method on `Iterator[T]` that returns a new lazy `Iterator[T]`), lazy evaluation (computation deferred until materialisation), materialisation (explicit call to `.collect()`, `.to_list()`, etc.), loop fusion (optimisation combining multiple combinator passes into one loop).

**Test with counterexamples.** Counterexample: `.map(|x| f(x)).map(|x| g(x))` — logically two passes, semantics say two mappings occur. Loop fusion would optimise to one pass, but semantics guarantee the same result. No contradiction.

**Verdict: Pass.** All combinators are compositions of `Iterator[T].next()`. No new semantic axioms needed.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Composable collection operations can be fully implemented as Standard Library methods on `Iterator[T]` without any language-level additions.

**Simplest experiment.** Implement `.map()`, `.filter()`, and `.fold()` as methods on `Iterator[T]` using the existing `function`, `call`, and `scope` primitives. Verify no compiler changes are required.

**Verdict: Pass.** StdLib classification is the simplest possible answer — no language changes needed.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) `Iterator[T]` is a Language-layer trait (EDR-022). (2) Combinator methods on `Iterator[T]` live in the Standard Library. (3) The Standard Library builds on the Language layer, not the reverse.

**Verdict: Pass.** Combinators operate entirely within the Standard Library layer, depending on `Iterator[T]` (Language layer).

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Combinator semantics (what each operation produces) must be consistent across all implementations, but loop fusion (how multiple passes are optimised) is inherently implementation-specific.

**Apply separation.** Separate *what* (the semantic specification of each combinator — what it produces for a given input) from *how* (whether multiple combinator passes are fused into a single loop). Loop fusion is explicitly an Implementation Strategy concern, not a semantic guarantee.

**Verdict: Pass.** Combinator semantics are strategy-independent. Loop fusion (optimisation) is explicitly delegated to implementation.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *"The collection operations API must accommodate new combinators and future comprehension syntax without breaking existing combinator chains."*

**Verdict: Pass.** StdLib classification means combinators can be added, deprecated, or changed without language version changes.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Combinators have simple, well-defined signatures. The Iterator combinator pattern is well-established across languages (Rust, Swift, Kotlin).

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Each combinator has a well-defined function signature: `Iterator[T].map(U)(f: T -> U) -> Iterator[U]`. |
| Predictable generation (≥90%) | Pass | Map/filter/reduce pattern is one of the most reliably generated patterns across all languages. |
| No hallucination surface | Pass | Combinator signatures are explicit; type errors on wrong signature produce standard type-checking diagnostics. |
| Strategy-aware default | Pass | Lazy-by-default semantics are consistent across all strategies (eager strategies must still honour lazy API contract). |
| Self-correctable | Pass | Type errors in combinator chains produce standard diagnostics with expected type hints. |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright. Composable collection operations as StdLib is the simplest, most maintainable answer — the Iterator Protocol (EDR-022) provides everything needed, and zero language additions are required.

**Gates not applied:** None.

---

## Entry: CONCURRENCY_MODEL (EDR-033)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/CONCURRENCY_MODEL.md`](../../what/concepts/CONCURRENCY_MODEL.md)
**Decision recorded as:** [EDR-033](../decision_records/architecture/EDR-033-concurrency-model.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Concurrent execution without shared mutable state, data races, or deadlocks. How does Orthon extend its "no shared mutable state" safety guarantee to parallel execution? |
| Q2 | Is this a language problem or a library problem? | **Language.** The `act` modifier, `delegate` keyword, `<-` message operator, and ownership transfer rules require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. The `act` modifier changes type semantics (isolated state, message-passing interface). The `<-` operator introduces message-queue semantics. |
| Q4 | Does it violate any Design Principle? | No. Aligns with "no shared mutable state" (core principle), Explicit Semantics (`act`, `delegate`, `<-` are syntactically visible), Orthogonality (concurrency model composes with traits, error handling, ownership). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Delegate isolation, message-passing execution, single-threaded per-delegate processing, automatic parallelism from independence, ownership transfer across boundaries, error propagation across delegates. |
| Q6 | Can it be expressed through composition? | No. Delegate isolation and message-passing semantics require compiler-level support — the compiler must enforce that no two delegates share mutable memory. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Concurrency semantics define how programs execute in parallel. Scheduling (work-stealing vs. pinned-to-thread) is an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language targeting multi-core processors. The delegate model provides data-race freedom by construction. |

**Classification per D-03:** Language. Core semantic dimension (how concurrent execution is defined). Compiler-level guarantees (data-race freedom, isolation). Delegate-based model, `act` modifier, no shared-state threads.

**Primitive decomposition path:** `act` modifier → type declaration modifier (compiler-enforced isolation semantics); `delegate` → `reference` + isolated `scope` + message queue; `<-` operator → compiler-recognized message-send syntax; ownership transfer → existing `reference` + ownership semantics across boundaries. The isolation guarantee, message ordering, and single-threaded processing per delegate add compiler-level semantics beyond primitive composition.

### Gate Validation

**Gates applied:** All 7 (a new Core Language semantic — concurrent execution model — requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer targeting multi-core hardware, I want safe concurrent execution without data races, deadlocks, or shared-memory complexity — I want to declare isolation boundaries and send messages, not manage threads and locks.

**Press release.** *Orthon's delegate-based concurrency model eliminates entire categories of parallel bugs. The `act` modifier declares concurrent types; `delegate` creates isolated execution contexts; `<-` sends messages between them. No shared-state threads, no mutexes, no data races by construction — and the model is implementation-independent, supporting any runtime from work-stealing to bare-metal.*

**FAQ.**
- *No threads at all?* — No shared-state threads. Delegates map to OS threads, fibers, or WASM workers depending on Implementation Strategy — the programmer never manages threads directly.
- *How do I share data between delegates?* — Explicit ownership transfer (`$` operator). Data moves, it is never shared.
- *Can a delegate return a value?* — Yes — value-returning messages return a future that resolves when the delegate processes the message.

**Verdict: Pass.** Concurrency without data races or deadlocks is a direct user need. Delegate model is familiar from actor frameworks.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Delegate (isolated execution context with its own state), message passing (data sent between delegates via `<-` operator), ownership transfer (data moves, not copies, across isolation boundaries), `act` modifier (declares a type as concurrent), single-threaded processing (at most one message processed at a time per delegate).

**Test with counterexamples.** Counterexample: two delegates both hold a `$`-transferred reference to the same mutable data — impossible because `$` transfers ownership, removing access from the sender. Counterexample: a delegate waits for a message from another delegate that is also waiting — potential deadlock at application level (message ordering) but no lock-level deadlock.

**Verdict: Pass.** Isolation guarantee follows from ownership rules. No shared mutable state is already a core principle — this extends it to concurrent execution.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One model (delegate + message passing) can replace threads, locks, condition variables, atomics, and async/await.

**Simplest experiment.** Implement a producer-consumer pattern: one delegate produces values, another consumes them via `<-` message sends. No mutexes, no channels, no thread spawning. Verify data-race freedom by construction.

**Verdict: Pass.** One model (delegate + message passing) replaces threads, locks, condition variables, atomics, and async/await.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Concurrency semantics (`act`/`delegate`/`<-`) live in the Core Language layer. (2) Ownership transfer rules (existing) extend naturally to cross-delegate boundaries. (3) StdLib concurrency utilities (Plan 04-06) build on these primitives without defining new semantics.

**Verdict: Pass.** `act`/`delegate`/`<-` operate in the Core Language layer. StdLib concurrency (Plan 04-06) builds on top.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Concurrency must be implementable across radically different runtimes (work-stealing thread pool, pinned-to-thread, single-threaded cooperative, WASM), yet the *semantics* of concurrent execution must be invariant.

**Apply separation.** Separate *what* (semantics: isolation, message ordering, ownership transfer) from *how* (the runtime realising these semantics). The model defines concurrency purely in terms of isolation and message passing — threads, fibers, and async runtimes are implementation concerns.

**Verdict: Critical — Pass.** The model is defined purely in terms of isolation, message passing, and ownership transfer. No reference to threads, fibers, async runtimes, or scheduling strategies. Every proposed Implementation Strategy (work-stealing, pinned-to-thread, single-threaded) can realise the semantics.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *"The concurrency model must accommodate future parallelism patterns and runtime innovations without requiring changes to user code that uses `act`/`delegate`/`<-`."*

**Verdict: Pass.** Evolution path: Plan 04-06 adds StdLib utilities. No shared-state threads means no future compatibility burden from thread-safety guarantees.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `act` modifier, `delegate` keyword, and `<-` operator are explicit and locally visible. The model follows a well-known pattern (actor model) that LLMs generate reliably.

**Apply the five-criterion table:**

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `act` modifier on types, `delegate` keyword, and `<-` operator are syntactically explicit. Message types are trait-bound and schema-exposable. |
| Predictable generation (≥90%) | Pass | Actor-model message passing is one of the most reliably generated concurrent patterns across LLMs. |
| No hallucination surface | Pass | No implicit shared state — all cross-delegate data uses explicit `$` ownership transfer. |
| Strategy-aware default | Pass | Default strategy (work-stealing thread pool) realises the semantics without user-visible scheduling annotation. |
| Self-correctable | Pass | Ownership transfer violations (`$` missing) produce standard compiler diagnostics with repair hints. |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright — one gate (Implementation Independence) is marked Critical but Passes, confirming the model's independence from runtime implementation. The delegate-based concurrency model provides data-race freedom by construction while remaining implementation-independent.

**Gates not applied:** None.

---

### Essential — Policy Level

The following concepts run through the Decision Pipeline but are classified as **Policy** (not Language) per D-04. They do not add new semantics expressible via primitives — they are implementation choices about HOW primitives are realised.

---

## Entry: ALLOCATION (EDR-034)

**Date:** 2026-07-27
**Artifact validated:** [`how/strategies/ALLOCATION.md`](../../how/strategies/ALLOCATION.md)
**Decision recorded as:** [EDR-034](../decision_records/architecture/EDR-034-allocation.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How and when is memory allocated for program data? Allocation affects performance predictability, memory safety, and overall program design. |
| Q2 | Is this a language problem or a library problem? | **Policy.** Allocation is how language semantics are realised in memory — an implementation choice, not a language feature. |
| Q3 | Can it be solved with existing primitives? | Yes — allocation is the materialisation of existing primitives (`literal`, `pack`, `identifier`, `reference`). The programmer declares data structures; allocation is implicit. |
| Q4 | Does it violate any Design Principle? | No. Allocation as Policy aligns with Minimal Core (allocation mechanism is an implementation detail), Intent Over Implementation (programmer declares data; compiler decides allocation), Implementation Independence (allocation strategy is interchangeable). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Allocation is the realisation of existing primitives — it does not add new semantic operations. |
| Q6 | Can it be expressed through composition? | N/A — Policy is about how primitives are realised, not composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | N/A — Policy classification. |
| Q8 | Is this an optimisation, not semantics? | Yes — allocation is purely an implementation/optimisation concern. Language semantics are allocation-agnostic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical implementation. Strategy system must define allocation choices. |

**Classification per D-04:** Policy. Allocation is an Implementation Policy — how data is materialised in memory, not what data means.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Skipped | Policy concept — user value is indirect through strategy selection. Not a direct language feature. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Policy classification is consistent with D-04: no new semantics, independent of language core. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Five values cover known allocation strategies. Each is well-understood. No overlap. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Fits cleanly in Strategy system — one policy type with orthogonal values. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Allocation is fully independent of any specific implementation. Core semantics work under all values. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Adding new allocation strategies (e.g., NUMA-aware) simply adds a new policy value. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Skipped | Policy concept — LLM never writes allocation code. Strategy selection is a build-time concern. |

**Overall:** Pass with reduced gate set. USER_VALUE_GATE and LLM_GENERABILITY_GATE are skipped per D-04's Policy pipeline — Policy concepts provide indirect user value through strategy selection, and LLMs never write allocation strategy code. All remaining gates (LOGICAL_CONSISTENCY through LONG_TERM_MAINTAINABILITY) pass with clear reasoning.

---

## Entry: REGION_BASED_MEMORY_MANAGEMENT (EDR-035)

**Date:** 2026-07-27
**Artifact validated:** [`how/strategies/REGION_BASED_MEMORY_MANAGEMENT.md`](../../how/strategies/REGION_BASED_MEMORY_MANAGEMENT.md)
**Decision recorded as:** [EDR-035](../decision_records/architecture/EDR-035-region-based-memory-management.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Predictable, efficient memory deallocation without garbage collection. How to manage memory in bulk by scoped regions (arenas). |
| Q2 | Is this a language problem or a library problem? | **Policy.** Region-based allocation is a sub-policy within the Allocation Policy — a specific implementation choice for how Arena allocation works. |
| Q3 | Can it be solved with existing primitives? | Yes — region allocation is a materialisation strategy for the `pack` and `reference` primitives. The programmer never writes arena management code. |
| Q4 | Does it violate any Design Principle? | No. Region allocation as Policy aligns with Intent Over Implementation (arenas are an invisible optimisation), Minimal Core (arena mechanism is implementation detail). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Region allocation refines how Arena policy materialises memory — it does not add new language semantics. |
| Q6 | Can it be expressed through composition? | N/A — Policy classification. |
| Q7 | Can it be syntactic sugar over existing primitives? | N/A — Policy classification. |
| Q8 | Is this an optimisation, not semantics? | Yes — region allocation is an implementation optimisation (bulk deallocation, bump allocation). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Provides the default allocation strategy (Arena) with three scoping modes: ScopeRegion, ExplicitRegion, NoRegion. |

**Classification per D-04:** Policy. Sub-policy of Allocation Policy (EDR-034). Refines Arena allocation with lifetime-scoping strategies.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Skipped | Sub-policy — indirect value through allocation performance. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Sub-policy refinement is consistent with Policy architecture. Arena allocation naturally decomposes into lifetime-scoping strategies. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Three values cover the natural spectrum: automatic, explicit, disabled. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Sub-policy pattern extends the Strategy system without adding new architectural concepts. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Region semantics (scope-bound lifetimes, bulk deallocation) are implementation-independent. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Adding inference strategies is a new sub-policy value, not a system change. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Skipped | Policy concept — LLM never writes arena management code. |

**Overall:** Pass with reduced gate set. USER_VALUE_GATE and LLM_GENERABILITY_GATE are skipped per D-04's Policy pipeline — Policy concepts provide indirect user value through strategy selection, and LLMs never write arena management code. All remaining gates (LOGICAL_CONSISTENCY through LONG_TERM_MAINTAINABILITY) pass with clear reasoning.

---

## Entry: EXECUTION_PROGRAM (EDR-036)

**Date:** 2026-07-27
**Artifact validated:** [`how/strategies/EXECUTION_PROGRAM.md`](../../how/strategies/EXECUTION_PROGRAM.md)
**Decision recorded as:** [EDR-036](../decision_records/architecture/EDR-036-execution-program.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | The fragmented toolchain problem — compilation, packaging, and deployment are separate stages with incompatible formats. The same information is described multiple times. |
| Q2 | Is this a language problem or a library problem? | **Policy.** Execution model is an implementation choice — how a program is run, not what the program means. Introduces a new Policy type (Execution Model Policy). |
| Q3 | Can it be solved with existing primitives? | Yes — execution model does not add new language semantics. Core concepts (equality, ownership, pattern matching) are unchanged regardless of execution strategy. |
| Q4 | Does it violate any Design Principle? | No. Execution Program aligns with Intent Over Implementation (programmer declares what; infrastructure decides how), SOLID, Minimal Core (execution is infrastructure, not language). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Execution model is about packaging and running — it does not change what programs mean. |
| Q6 | Can it be expressed through composition? | N/A — Policy classification. |
| Q7 | Can it be syntactic sugar over existing primitives? | N/A — Policy classification. |
| Q8 | Is this an optimisation, not semantics? | Yes — execution strategy is a build-time/deployment concern, not language semantics. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** This is Orthon's core innovation — decoupling semantics from execution strategy. The Execution Program model eliminates toolchain fragmentation. |

**Classification per D-04:** Policy. New Execution Model Policy type in the Strategy system. Execution is infrastructure, not language.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Skipped | Execution model Policy — indirect user value through simplified deployment. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Decoupling semantics from execution is internally consistent with Orthon's Intent Over Implementation principle. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Five values cover the full execution spectrum. Each is a well-understood technology. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | New Policy type integrates cleanly into the existing Strategy system as a peer of Allocation, Algorithm, etc. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Execution model is fully independent of any specific implementation. All values are realisable by different runtime implementations. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Adding new execution engines (MicroVM, FPGA, etc.) adds a new Policy value without changing the system. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Skipped | Execution model is a build-time/deployment concern — LLM never writes execution strategy code. |

**Overall:** Pass with reduced gate set. USER_VALUE_GATE and LLM_GENERABILITY_GATE are skipped per D-04's Policy pipeline — Policy concepts provide indirect user value through strategy selection, and execution model selection is a build-time/deployment concern. All remaining gates (LOGICAL_CONSISTENCY through LONG_TERM_MAINTAINABILITY) pass with clear reasoning.

---

### Essential — Derived Features (Wave 3)

The following borderline concepts were evaluated per D-03 classification rules. Both resolved as corrections to existing documents rather than standalone Language features.

---

## Entry: CONTEXT_PARAMETERS (EDR-037)

**Date:** 2026-07-27
**Artifact validated:** [`what/SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md) (correction)
**Decision recorded as:** [EDR-037](../decision_records/architecture/EDR-037-context-parameters.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Plumbing execution environment objects (logger, database connection, configuration) through call chains without explicit threading at every intermediate call site. |
| Q2 | Is this a language problem or a library problem? | **Language.** Context resolution requires compiler support for type-directed `given` resolution and lexical scoping rules. However, the mechanism is a cross-cutting concern of Evaluation and Visibility, not a standalone feature. |
| Q3 | Can it be solved with existing primitives? | No — implicit parameter threading is not expressible via the 9-primitive set. It requires compiler-level parameter resolution. |
| Q4 | Does it violate any Design Principle? | No. Context parameters align with Explicitness (`using` block is visible in signature), Intent Over Implementation (programmer declares context need; compiler resolves it). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | Yes — context parameters add implicit-passing semantics with static resolution. However, these are a correction to the Evaluation dimension (context supply timing) and Visibility dimension (`given` scope resolution), not a standalone feature. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No — parameter passing is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes, but deferred.** Context parameters solve a real ergonomic problem. However, for v0.1, a SEMANTIC_MODEL correction acknowledging context flow as a cross-cutting concern is sufficient. Full specification deferred beyond v0.1. |

**Classification per D-03:** SEMANTIC_MODEL correction. Cross-cutting concern of Evaluation and Visibility dimensions. Not a standalone Language feature.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Flag | Context parameters provide significant ergonomic value (eliminate manual plumbing), but correction-level treatment defers full value. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Context flow as a cross-cutting concern of Evaluation and Visibility is logically consistent with the existing semantic dimensions. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | A brief note in SEMANTIC_MODEL.md is the simplest correct treatment — adds awareness without commitment. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Cross-cutting concern pattern fits the Semantic Model's existing design (each dimension already has cross-cutting references). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Context resolution semantics (compile-time, scope-directed) are implementation-independent. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Preserving the design space as a SEMANTIC_MODEL note avoids future contradictions. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Context parameters have LLM Generability concerns (implicit resolution can produce non-obvious results), but correction-level treatment defers resolution. |

**Overall:** Pass with reduced gate set appropriate for a SEMANTIC_MODEL correction. USER_VALUE_GATE is flagged (value recognised, but full specification deferred beyond v0.1). All remaining gates pass — the correction is logically consistent, conceptually simple, architecturally sound, implementation-independent, maintainable, and LLM-compatible at the correction level.

---

## Entry: REPRESENTATION_MODIFIERS (EDR-038)

**Date:** 2026-07-27
**Artifact validated:** [`what/PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) (correction)
**Decision recorded as:** [EDR-038](../decision_records/architecture/EDR-038-representation-modifiers.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How to control how values of a type are stored in memory (inline, boxed, packed, FFI-compatible) without changing the type's semantic identity. |
| Q2 | Is this a language problem or a library problem? | **Primitive-level.** Representation modifiers are annotations on existing primitives (`pack` for inline/struct, `reference` for indirection/boxed) — they are not new operations requiring separate language status. |
| Q3 | Can it be solved with existing primitives? | Yes — representation modifiers are orthogonal annotations on the `pack` primitive (struct, packed) and the `reference` primitive (boxed, shared, atomic). |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (modifier syntax is visible), Orthogonality (modifiers compose with any type), Minimal Core (modifiers are annotations, not new primitives). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | The modifiers add annotation semantics (storage strategy selection), but these are constraints on how existing primitives are realised — not new primitive operations. |
| Q6 | Can it be expressed through composition? | Yes — `struct(T)` = `pack` without runtime metadata; `boxed(T)` = `reference` to heap allocation. The modifiers are composition annotations. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — each modifier desugars to a constraint on how `pack` or `reference` is realised. |
| Q8 | Is this an optimisation, not semantics? | Partially — storage selection is an implementation concern (how to lay out data). The modifier syntax itself is language-level, but the semantics are unchanged. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes, but deferred.** Representation modifiers solve a genuine problem. For v0.1, a PRIMITIVE_BLOCKS correction noting that representation annotations are orthogonal modifiers on primitives is sufficient. Full specification deferred beyond v0.1. |

**Classification per D-03:** PRIMITIVE_BLOCKS correction. Representation modifiers are orthogonal annotations on existing primitives (`pack` and `reference`), not new primitives or a standalone Language feature.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Flag | Representation modifiers provide significant value (storage control without type duplication), but correction-level treatment defers full specification. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Modifiers as annotations on primitives is logically consistent: they add constraints to existing operations without creating new operations. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Annotation model is simpler than adding 6 new primitives. The correction correctly identifies the minimal solution. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Annotation pattern fits Orthon's existing modifier system (e.g., `fun`/`proc`/`new` are declaration kind modifiers on the `function` primitive). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Representation modifiers describe storage intent — implementation is independent of how the compiler realizes that intent (inline, boxed, arena, etc.). |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Keeping representation modifiers as annotations rather than primitives ensures the primitive set remains stable as new modifiers are added. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs can add `struct`/`boxed` annotations — they are syntactic modifiers with clear semantics. |

**Overall:** Pass with reduced gate set appropriate for a PRIMITIVE_BLOCKS correction. USER_VALUE_GATE is flagged (value recognised, but full specification deferred beyond v0.1). All remaining gates pass — the annotation model is logically consistent, conceptually simpler than adding new primitives, architecturally aligned with Orthon's modifier system, implementation-independent, maintainable, and LLM-generable.

---

### Important Tier — Wave 4

The following important-tier concepts were processed through the Decision Pipeline. See individual EDRs for Gate Validation and detailed reasoning.

---

## Entry: ALGEBRAIC_DATA_TYPES (EDR-039)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/ALGEBRAIC_DATA_TYPES.md`](../../what/concepts/ALGEBRAIC_DATA_TYPES.md)
**Decision recorded as:** [EDR-039](../decision_records/architecture/EDR-039-algebraic-data-types.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Data that takes one of several forms needs language-level support. Enum alternatives (named constants, iota) are subsumed by ADTs. |
| Q2 | Language/StdLib/Policy? | **Language.** Combined variant+field declaration, automatic discriminant, recursive type checking, sealed exhaustiveness — all require compiler support. |
| Q3 | Existing primitives? | No. Combined sum+product in one form is not expressible via manual sealed trait + variant types. |
| Q4 | Violates principle? | No. Minimal Core (one sum-type mechanism replaces two), Orthogonality, Explicitness. |
| Q5 | New semantics? | **New semantics.** Combined variant+field declaration, automatic discriminant, sealed exhaustiveness, recursive type termination. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | Partial — ADT could desugar to sealed trait + variant types, but automatic discriminant and recursive checking need compiler support. |
| Q8 | Optimisation? | No. Sum types are semantic — what values a type can hold. |
| Q9 | Backward compat? | N/A — pre-v1.0. Builds on TRAITS (EDR-019) + PATTERN_MATCHING (EDR-025). |
| Q10 | Worth adding? | **Yes.** Essential. ADTs are the proven mechanism for type-safe sum types. Subsumes enum construct. |

**Classification per D-03:** Language. New semantics beyond TRAITS + PATTERN_MATCHING. Combined with ENUM_ALTERNATIVES (no separate EDR-040).

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | ADTs solve a fundamental data-modelling problem. Every non-trivial program needs sum types. Enum use case is covered with zero additional syntax. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | One sum-type mechanism, one declaration form. No contradictions with existing TRAITS or PATTERN_MATCHING semantics. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Combined ADT + enum in a single mechanism is simpler than two separate mechanisms. The `type ... = ... | ...` form is intuitive. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | ADTs build on existing sealed trait (EDR-019) and pattern matching (EDR-025) foundations. No new architectural layer required. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | ADT semantics (sealed variants, exhaustiveness, recursive types) are defined independently of any memory layout strategy. Tagged union is the default layout but alternatives (niche optimisation, flat layout) are permitted. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Single sum-type mechanism is more maintainable than two overlapping ones. ADT pattern is proven across Rust, Haskell, OCaml, Swift, Kotlin. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | ADT declaration syntax is simple and regular. Pattern matching on ADTs follows the same rules as pattern matching on sealed traits. LLMs already demonstrate reliable ADT generation in similar languages (Rust enums, TypeScript discriminated unions). |

**Overall:** Pass — all 7 gates pass. ADTs are a fundamental, well-understood language feature that builds on existing Orthon foundations (TRAITS + PATTERN_MATCHING) without introducing architectural complexity. The single-mechanism approach satisfies "One concept — one syntax" while subsuming the enum use case.

---

## Entry: COLLECTION_LITERAL_SYNTAX (EDR-041)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/COLLECTION_LITERAL_SYNTAX.md`](../../what/concepts/COLLECTION_LITERAL_SYNTAX.md)
**Decision recorded as:** [EDR-041](../decision_records/architecture/EDR-041-collection-literal-syntax.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Creating a collection manually is verbose and encourages mutation. `[1, 2, 3]` is universally preferred. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Collection literal desugars to constructor call. No new compiler-level semantics. |
| Q3 | Existing primitives? | Yes — each element is `literal`/`identifier`, the literal as a whole is a `call` to a collection constructor. |
| Q4 | Violates principle? | No. Minimal Core, Intent Over Implementation. |
| Q5 | New semantics? | **No new semantics.** Syntactic sugar for constructor calls. |
| Q6 | Composition? | Yes — of `literal` elements + `call` to collection constructor. |
| Q7 | Sugar over primitives? | Yes — `[1, 2, 3]` desugars to `List(1, 2, 3)`. |
| Q8 | Optimisation? | Syntax, not optimisation. Large-literal desugaring is optimisation. |
| Q9 | Backward compat? | N/A — pre-v1.0. Syntax deferred to Phase 5. |
| Q10 | Worth adding? | **Yes.** Universal ergonomic expectation. Zero language additions. Syntax deferred. |

**Classification per D-03:** StdLib. Syntactic sugar for collection constructors. Syntax deferred to Phase 5.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Collection literals solve a universal ergonomic pain point — no programmer wants to write `list.add(1); list.add(2)` when `[1, 2]` suffices. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Desugaring to constructor calls is internally consistent — no special cases, no new type rules. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | StdLib classification keeps the language core minimal. Literal syntax is syntactic sugar, not new semantics. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | StdLib classification places collection literals in the correct architectural layer (syntax desugaring to StdLib, not core language). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Collection semantics (immutable by default, desugaring to constructor calls) are independent of any specific collection implementation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Collection literals are a universal, stable language feature. The desugaring approach minimizes long-term maintenance burden. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Collection literals are among the most LLM-generable constructs — `[1, 2, 3]` is unambiguous and universally understood. |

**Overall:** Pass — all 7 gates pass. Collection literals as StdLib syntactic sugar is the correct classification: they solve a universal ergonomic need without adding compiler-level complexity. Syntax deferral to Phase 5 preserves the design space without premature commitment.

---

## Entry: DATACLASSES (EDR-042)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/DATACLASSES.md`](../../what/concepts/DATACLASSES.md)
**Decision recorded as:** [EDR-042](../decision_records/architecture/EDR-042-dataclasses.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Boilerplate for passive data carriers — manual constructors, accessors, equality, hashing, string representation. |
| Q2 | Language/StdLib/Policy? | **StdLib.** `@derive` mechanism (EDR-029) already provides compile-time code generation. No new language semantics. |
| Q3 | Existing primitives? | Yes — `@derive(init, eq, repr, hash)` reuses existing macro mechanism. |
| Q4 | Violates principle? | No. Minimal Core, Explicitness (`@derive` annotations are visible). |
| Q5 | New semantics? | **No new semantics** for derives. `with` expression adds limited compiler intrinsic. |
| Q6 | Composition? | Yes — each derive target maps to a registered macro function. |
| Q7 | Sugar over primitives? | Yes — `@derive(init)` desugars to `@macro` invocation. |
| Q8 | Optimisation? | No — code generation is semantic. Uses existing mechanisms. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Eliminates the most common boilerplate. No new keywords. |

**Classification per D-03:** StdLib. Dataclass pattern via existing `@derive` mechanism (EDR-029). No dedicated keyword.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Dataclasses eliminate a major source of boilerplate — programmers write `@derive(init, eq, repr, hash)` instead of dozens of lines of mechanical code. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Dataclass derives are a straightforward application of the existing `@derive` mechanism. No new type rules or semantic contradictions. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | The `@derive` pattern is simpler than a dedicated keyword — one mechanism (derive) covers all boilerplate generation needs. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | StdLib classification places dataclass derives in the correct layer. The `with` expression is a limited compiler intrinsic (copy-with-modify), not a general syntactic extension. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Dataclass semantics (structural equality, copy-with-modify) are independent of any specific memory layout or allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | The derive-based approach is more maintainable than a dedicated keyword — new derive targets can be added without language changes. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `@derive(init, eq, repr, hash)` is a simple, regular annotation that LLMs can reliably generate. The pattern is familiar from Rust's `#[derive(...)]`. |

**Overall:** Pass — all 7 gates pass. Dataclasses as a derive-based pattern (not a dedicated keyword) is the correct approach. It reuses the existing `@derive` mechanism, keeps the language core minimal, and provides composable boilerplate elimination.

---

## Entry: LITERAL_TYPES (EDR-043)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/LITERAL_TYPES.md`](../../what/concepts/LITERAL_TYPES.md)
**Decision recorded as:** [EDR-043](../decision_records/architecture/EDR-043-literal-types.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Modelling closed sets of string/number values without ADT declaration ceremony. `"GET"` is its own type, composing via `\|`. |
| Q2 | Language/StdLib/Policy? | **Language.** Compiler must track literal types as singleton types, apply widening rules, support narrowing in pattern matching. |
| Q3 | Existing primitives? | No. Singleton type tracking is a type-system operation not in the 9-primitive set. |
| Q4 | Violates principle? | No. Explicit Semantics (one widening rule), Minimal Core (one rule covers all cases). |
| Q5 | New semantics? | **New semantics.** Singleton types for literal values, widening rule, narrowing in pattern matching. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | No — see Q3. |
| Q8 | Optimisation? | No. Type tracking is semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for API boundaries and protocol constants. Composes with union types (EDR-045). |

**Classification per D-03:** Language. Values as types require compiler-level literal type tracking, widening, and narrowing.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Literal types solve a real ergonomic need — modelling closed sets of string/number values without ADT declaration ceremony. Essential for API boundaries and protocol constants. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | One explicit widening rule eliminates context-dependent ambiguity. No contradictions with existing type system (ADTs, pattern matching). |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | The concept is simple: a value is its own type. The widening rule is a single, always-applicable rule. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on the existing type system — no new architectural layer. Literal types are a natural extension of the type inference mechanism (EDR-027). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Literal type semantics (singleton types, widening, narrowing) are independent of any memory layout or allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Literal types are a well-understood feature (TypeScript, Scala 3, Python typing). The restricted scalar-only scope limits long-term complexity. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The single widening rule (`let` preserves, `var` widens) is LLM-generable. Literal type union syntax (`"GET" | "POST"`) is unambiguous. |

**Overall:** Pass — all 7 gates pass. Literal types provide essential ergonomic value for API boundaries and protocol constants. The single explicit widening rule avoids TypeScript-style context-dependent complexity while preserving LLM generability. Scope is appropriately restricted to scalar primitives for v0.1.

---

## Entry: STRUCTURAL_TYPING (EDR-044)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/STRUCTURAL_TYPING.md`](../../what/concepts/STRUCTURAL_TYPING.md)
**Decision recorded as:** [EDR-044](../decision_records/architecture/EDR-044-structural-typing.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Polymorphism without explicit declaration ceremony. Marker traits require mechanical `impl` blocks. |
| Q2 | Language/StdLib/Policy? | **Language.** Structural trait resolution — checking method signatures at compile time — requires compiler support. |
| Q3 | Existing primitives? | No. Structural method signature matching across types requires the type system to compare method shapes without an explicit `impl`. |
| Q4 | Violates principle? | No. Intent Over Implementation, Explicitness (`structural` keyword makes mode visible). |
| Q5 | New semantics? | **New semantics.** Structural matching — compiler checks method signatures without explicit `impl`. Priority rules, ambiguity detection. |
| Q6 | Composition? | No — structural signature matching is a type-system operation. |
| Q7 | Sugar over primitives? | No — see Q6. |
| Q8 | Optimisation? | No. Trait satisfaction determines legal operations — semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. Builds on TRAITS (EDR-019). |
| Q10 | Worth adding? | **Yes.** Eliminates ceremony for marker traits. Nominal-by-default maintains explicitness. |

**Classification per D-03:** Language. Structural trait resolution adds compiler-level semantics beyond nominal TRAITS (EDR-019).

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Structural typing eliminates ceremony for marker traits. The `structural` keyword makes the trade-off explicit. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Clear precedence: explicit `impl` > structural match. Ambiguity detection prevents silent conflicts. No contradictions with nominal TRAITS (EDR-019). |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Two modes (nominal, structural) with clear, simple rules. The `structural` keyword is a single syntactically visible modifier. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on the existing TRAITS architecture (EDR-019). Structural matching is an extension of trait resolution — no new architectural layer. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Structural typing semantics (method signature matching, priority rules, conflict resolution) are independent of any specific memory layout or dispatch strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Nominal-by-default with opt-in structural is a proven pattern (Scala, OCaml). The two-mode approach avoids the coherence challenges of fully structural systems. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The `structural` keyword makes the mode explicit. LLMs can reliably determine when a type structurally satisfies a trait by checking method signatures. |

**Overall:** Pass — all 7 gates pass. Structural typing as opt-in (via the `structural` keyword) with nominal-by-default is the correct balance of flexibility and explicitness. The clear precedence rules and ambiguity detection preserve Orthon's explicitness commitment while eliminating ceremony for marker traits.

---

## Entry: UNION_INTERSECTION_TYPES (EDR-045)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/UNION_INTERSECTION_TYPES.md`](../../what/concepts/UNION_INTERSECTION_TYPES.md)
**Decision recorded as:** [EDR-045](../decision_records/architecture/EDR-045-union-intersection-types.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Ad-hoc composition of alternative types at point of use — `String \| Int` for an ID — without ADT declaration ceremony. |
| Q2 | Language/StdLib/Policy? | **Language.** The `\|` combinator introduces a new type former — anonymous, untagged union of types. |
| Q3 | Existing primitives? | No. Anonymous union type concept is not expressible via the 9-primitive set. |
| Q4 | Violates principle? | No. Orthogonality, Minimal Core (one combinator for all union needs). |
| Q5 | New semantics? | **New semantics.** Anonymous union type formation, narrowing via match/is, structural assignment compatibility. No exhaustiveness. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | No — see Q3. |
| Q8 | Optimisation? | No. Type formation is semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for ad-hoc composition. Intersection types NOT accepted for v0.1. |

**Classification per D-03:** Language. Union types add new type-system combinator (`A | B`) with narrowing semantics. Intersection types NOT accepted for v0.1.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Union types solve a real ergonomic problem — ad-hoc composition of alternative types without declaration ceremony. Essential for JSON-like APIs, configuration, and heterogeneous collections. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Clear rules: named type members only, no tag at runtime, narrowing follows EDR-028 rules. No contradictions with ADTs (EDR-039) — ADTs are for declared variants with exhaustiveness; unions are for ad-hoc composition. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | The `A | B` combinator is the simplest possible union form. Named-type-only restriction limits complexity. Intersection types excluded reduces surface area. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on the existing type system infrastructure — narrowing reuses EDR-028 rules, literal types provide the members (EDR-043). No new architectural layer. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Union type semantics (structural, untagged, narrowing via match/is) are independent of any memory layout or allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Named-type-only restriction limits long-term complexity. Union types are a well-understood, stable feature (TypeScript, Scala 3). |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The `A | B` syntax is unambiguous and widely understood. Named-type union members are easy for LLMs to reason about. The no-exhaustiveness rule is simple and clear. |

**Overall:** Pass — all 7 gates pass. Union types complement ADTs for ad-hoc composition without declaration ceremony. The named-type-only restriction and intersection-types-excluded decision keep the feature scope well-bounded for v0.1.

---

## Entry: TYPE_LEVEL_COMPUTATION (EDR-046)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/TYPE_LEVEL_COMPUTATION.md`](../../what/concepts/TYPE_LEVEL_COMPUTATION.md)
**Decision recorded as:** [EDR-046](../decision_records/architecture/EDR-046-type-level-computation.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Deriving new types from existing types — DTO types omitting sensitive fields, optional fields, property name extraction — without manual parallel type declarations. |
| Q2 | Language/StdLib/Policy? | **Language (restricted intrinsic set).** Compiler provides built-in type manipulation operations (`KeyOf`, `Pick`, `Omit`, `Partial`). |
| Q3 | Existing primitives? | No. Type-level operations transforming type shapes are not expressible via value-level primitives. |
| Q4 | Violates principle? | No, with restricted set. Full type-level language would violate Minimal Core and LLM Generability. Closed set of 8 documented intrinsics aligns with Explicit Semantics. |
| Q5 | New semantics? | **New semantics (intrinsics).** Each intrinsic defines a type-level transformation. Closed and non-recursive. |
| Q6 | Composition? | No — type-level transformations not expressible via value-level primitives. |
| Q7 | Sugar over primitives? | No — see Q6. |
| Q8 | Optimisation? | No. Type transformation is semantic — changes what types exist. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes, restricted set.** Closed set covers essential DTO-shaping use cases. No Turing-complete type-level language. |

**Classification per D-03:** Language (restricted closed intrinsic set). Non-recursive compiler intrinsics for type transformation. NO user-extensible type-level programming language.

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Type-level intrinsics solve a real ergonomic problem — deriving DTO types from domain types without manual parallel type declarations. The closed set covers the essential use cases. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Non-recursive, closed-set intrinsics have no self-referential paradoxes. Each intrinsic has a clear, fixed semantic definition. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | A closed set of 8 documented intrinsics is the simplest possible type-level computation model. No conditional logic, no recursion, no user-extensible syntax. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Intrinsics are a natural extension of the type system — no new architectural layer. Coexists with COMPILE_TIME_EXECUTION (EDR-031) for general metaprogramming and AST_MACROS (EDR-029) for custom derives. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Type-level computation semantics are independent of any memory layout, allocation strategy, or execution model. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Closed intrinsic set is maximally maintainable — no type-level code to version, deprecate, or debug. New intrinsics require explicit design review. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Fixed set of documented intrinsics is the most LLM-generable model. No recursive type reasoning, no conditional type evaluation — the LLM simply looks up the intrinsic's semantics. |

**Overall:** Pass — all 7 gates pass. The closed-set, non-recursive intrinsic model is the right balance for Orthon: it covers essential DTO-shaping use cases while avoiding the Turing-completeness failure mode of TypeScript's type-level language. The derive/macro mechanism (EDR-029) provides an escape hatch for custom type-level operations.

---

## Entry: ASYNC_AWAIT (EDR-047)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/ASYNC_AWAIT.md`](../../what/concepts/ASYNC_AWAIT.md)
**Decision recorded as:** [EDR-047](../decision_records/architecture/EDR-047-async-await.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | How to perform blocking I/O without blocking the OS thread. Async/await transforms imperative code into state machines. |
| Q2 | Language/StdLib/Policy? | **Language.** Async modifies function/proc/new semantics. Compiler transforms async functions into state machines. |
| Q3 | Existing primitives? | No. State machine transformation (coroutine compilation) is not expressible via the primitive set. |
| Q4 | Violates principle? | No. Explicit Semantics, Minimal Core (modifier on existing kinds), Orthogonality (async composes with `proc`/`fun`/`new`). |
| Q5 | New semantics? | **New semantics.** Stackless coroutine state machine, `await` suspension points, `Future` as first-class type, `spawn`/`scope`. |
| Q6 | Composition? | No — coroutine transformation is compiler-level. |
| Q7 | Sugar over primitives? | No — see Q6. |
| Q8 | Optimisation? | No. Async defines new execution semantics (suspension, resumption). |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for any practical language doing I/O. Proven model (Rust, JS, Python, C#). |

**Classification per D-03:** Language. Async as orthogonal modifier on `proc`/`fun`/`new`. Colourless model with Future as first-class value.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to perform I/O without blocking the OS thread, using straight-line code that reads naturally.

**Press release.** *Async/await is an orthogonal modifier on any `proc`/`fun`/`new` — colourless futures eliminate the async/sync function colouring problem, and `scope` provides structured concurrency with automatic lifecycle management.*

**FAQ.** What does `async` alone mean? Suspension, not parallelism — `spawn` is required for concurrent execution. Can I call an async function without awaiting it? Yes — `Future` is a first-class value.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `async` = coroutine modifier (suspension-only), `await` = suspension point, `Future<T>` = first-class async result value, `spawn` = parallel execution, `exclusive` = access serialisation, `scope` = structured concurrency lifecycle. Each term is precisely defined and orthogonal to the others.

**Test with counterexamples.** Does `async` without `await` compile? Yes — produces a `Future<T>`. Does `async new` make sense? Yes — a constructor that performs async setup. Does `exclusive` conflict with `async`? No — they address separate dimensions: suspension vs. access control.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** The async modifier model is minimal — removing any component creates a gap.

**Verdict: Pass.** Removing `async` forces callback-based async. Removing colourless futures forces strict colouring. Removing `spawn` conflates async with parallelism. Removing `scope` leaves lifecycle management to libraries. Each component justifies its existence.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** (1) Core Language has no StdLib or Implementation Strategy dependencies. (2) `async` operates within Core Language — it modifies execution of existing declaration kinds.

**Deduce the consequences.** The state machine transformation is a compiler-level operation at the Core → Syntax boundary. No layer violations — async does not depend on any specific runtime.

**Verdict: Pass.**

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Async requires runtime support (event loop), yet must be implementation-independent.

**Apply separation.** The *semantic definition* is strategy-independent: "a function that may suspend at `await` points." The event loop/runtime is an Implementation Strategy choice (single-threaded event loop default, multi-threaded work-stealing, or blocking I/O for embedded targets). The colourless model (Future as value) is strategy-independent.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Async marks a function that may suspend; await marks where it suspends; the compiler handles the rest.*

**Verdict: Pass.** The model matches proven patterns (Python, JS, Rust, C#) and addresses known pain points (colour, lifecycle, parallelism) with orthogonal solutions. Evolution path: async streams, iterators, and generators can be added without changing the core model.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `async` modifier and `await` expression have single, unambiguous meanings.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `async fun name() -> Future[T]` is fully expressible in the type system |
| Predictable generation (≥90%) | Pass | Async/await is one of the most LLM-generable patterns across Python, JS, Rust |
| No hallucination surface | Pass | Single unambiguous meaning for each keyword |
| Strategy-aware default | Pass | Stackless coroutines are the universal default |
| Self-correctable | Pass | Missing `await` where `Future` value used as `T` is a compile error; `await` outside async function is a compile error |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: CONCURRENCY (EDR-049)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/CONCURRENCY.md`](../../what/concepts/CONCURRENCY.md)
**Decision recorded as:** [EDR-049](../decision_records/architecture/EDR-049-concurrency.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Concrete concurrent execution patterns — typed channels, select, supervision, timers, fan-out/fan-in. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Channels wrap delegate mailboxes. Select is a macro/function. All implementable using existing constructs (EDR-033). |
| Q3 | Existing primitives? | Yes — channels = delegate + mailbox; select = polling multiple channels; supervision = delegate lifecycle management. |
| Q4 | Violates principle? | No. StdLib preserves Minimal Core. |
| Q5 | New semantics? | **No new semantics.** All utilities are function/method implementations. |
| Q6 | Composition? | Yes — of `delegate`, `<-`, `$`, and existing primitives. |
| Q7 | Sugar over primitives? | Yes — all are ordinary function calls. |
| Q8 | Optimisation? | Utilities are not optimisations. Scheduling is optimisation. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for coordinating work across delegates. Zero language additions. |

**Classification per D-03:** StdLib. Concrete async/concurrent patterns building on CONCURRENCY_MODEL (EDR-033).

### Gate Validation

**Gates applied:** `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` (StdLib minimum per `DECISION_VALIDATION.md` § Gate Selection). `LOGICAL_CONSISTENCY_GATE` and `ARCHITECTURAL_INTEGRITY_GATE` also verified per EDR-049.

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer using the delegate model, I need to coordinate work across multiple concurrent delegates — passing messages, waiting on multiple sources, supervising lifecycle.

**Press release.** *Channels, select, supervision, timers, and async I/O wrappers are StdLib utilities built on the delegate model — no new language constructs needed, all implementable as ordinary Orthon code.*

**FAQ.** How is a channel different from a delegate mailbox? A channel is a delegate that holds a buffer — it wraps the mailbox for multi-producer/multi-consumer use. Can I skip channels and use delegates directly? Yes — channels are optional utilities.

**Verdict: Pass.**

---

#### 2. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** All concurrency utilities are expressible as StdLib on top of the delegate model — no new language semantics required.

**Verdict: Pass.** Channels wrap delegate mailboxes. Select polls multiple channels. Supervision is a delegate that monitors other delegates. Timers are delegates on a scheduler. All are ordinary Orthon code.

---

#### 3. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Channels let delegates communicate; select lets them wait on multiple sources.*

**Verdict: Pass.** StdLib classification means channels can evolve independently of the language. No conceptual debt — channels, select, and supervision are well-understood patterns with decades of production use (Go, Erlang, Elixir).

---

#### 4. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** A channel is a delegate holding a buffer. `select` is consistent with pattern matching semantics — the first ready channel triggers the corresponding branch. All terms are precisely defined and consistent with the delegate model.

**Verdict: Pass.**

---

#### 5. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** All utilities live in the Standard Library layer, depending on the delegate model from the Core Language layer.

**Verdict: Pass.** No layer violations.

---

### Overall

All applied gates Pass outright.

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — StdLib additions do not require implementation independence (they are implementations). `LLM_GENERABILITY_GATE` — StdLib functions have standard LLM generability properties.

---

## Entry: GENERATORS (EDR-050)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/GENERATORS.md`](../../what/concepts/GENERATORS.md)
**Decision recorded as:** [EDR-050](../decision_records/architecture/EDR-050-generators.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Producers may need bidirectional communication with consumers. Simple lazy sequences need concise inline syntax (generator expressions). |
| Q2 | Language/StdLib/Policy? | **Language.** `yield` is a bidirectional variant of `emit`, requiring compiler-level state machine modification. |
| Q3 | Existing primitives? | No. Two-way communication via `yield` requires state machine to accept consumer values — new semantic dimension beyond EDR-021. |
| Q4 | Violates principle? | No. Builds on LAZY_SEQUENCE_GENERATORS (EDR-021). |
| Q5 | New semantics? | **New semantics.** Bidirectional yield, generator expressions, `yield from` delegation. |
| Q6 | Composition? | No — bidirectional state machine communication not expressible via primitives. |
| Q7 | Sugar over primitives? | Generator expressions are sugar over generator functions. Bidirectional yield needs state machine changes. |
| Q8 | Optimisation? | No. Two-way communication is a semantic extension. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Bidirectional yield enables interactive coroutine patterns. Generator expressions provide concise inline syntax. |

**Classification per D-03:** Language. Bidirectional yield adds consumer-to-producer communication semantics beyond EDR-021.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to produce lazy sequences and occasionally communicate back to the producer — without adding channels or callbacks.

**Press release.** *Bidirectional `yield` enables coroutine-style two-way communication; generator expressions provide concise inline syntax for simple lazy sequences; `yield from` enables natural generator composition.*

**FAQ.** How is `yield` different from `emit`? `yield` without expression is equivalent to `emit`. `yield expr` adds consumer-to-producer communication. When would I use bidirectional yield? Interactive generators, streaming protocols, collaborative computations.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `yield` without expression ≡ `emit` (defined equivalence). `yield expr` adds consumer-to-producer communication with defined semantics. Generator expressions desugar to generator functions deterministically. `yield from` delegates to a sub-generator's iterator protocol.

**Test with counterexamples.** Can `yield` and `emit` be mixed in the same function? Yes — they share the same state machine. Does `yield from` compose with bidirectional yield? Yes — the sub-generator's consumer communication is transparently forwarded.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Bidirectional yield adds one new capability (consumer-to-producer communication) not expressible via EDR-021's one-way `emit`.

**Verdict: Pass.** Two-way communication requires either channels, callbacks, or bidirectional yield. Channels add more complexity; callbacks invert control. Bidirectional yield is the simplest model.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Generator expressions and yield extend LAZY_SEQUENCE_GENERATORS (EDR-021) within the same Core Language layer.

**Verdict: Pass.** No layer violations.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Bidirectional yield requires the state machine to accept values from the consumer at suspension points, which seems to couple producer and consumer scheduling.

**Apply separation.** The state machine includes a storage slot for the consumer's sent value — this is a static structural addition, independent of scheduling or allocation strategy. The slot exists regardless of whether the consumer ever sends a value.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *A generator can optionally receive values from its consumer using `yield`.*

**Verdict: Pass.** Removing bidirectional yield loses two-way communication capability — channels are a heavier alternative. The model matches Python generators, a well-proven pattern.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Generator expressions match Python comprehensions. `yield` patterns match Python generators. `yield from` matches Python's `yield from`.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Generator expressions desugar to function calls — fully type-representable |
| Predictable generation (≥90%) | Pass | Python comprehensions and generators are among the most LLM-generable constructs |
| No hallucination surface | Pass | Single unambiguous meaning per keyword |
| Strategy-aware default | Pass | Stackless state machine is the universal default |
| Self-correctable | Pass | Type mismatches in bidirectional yield produce compile errors |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: PUSH_STREAMS (EDR-051)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/PUSH_STREAMS.md`](../../what/concepts/PUSH_STREAMS.md)
**Decision recorded as:** [EDR-051](../decision_records/architecture/EDR-051-push-streams.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Push-based reactive streams — dual of pull-based generators. Producer determines arrival. Needed for events, I/O, GUI, sensors. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Push streams are implementable using delegate model (EDR-033) and channels (EDR-049). |
| Q3 | Existing primitives? | Yes — stream = delegate + channel; subscription = callback; backpressure = channel capacity. |
| Q4 | Violates principle? | No. StdLib preserves Minimal Core. |
| Q5 | New semantics? | **No new semantics.** Streams are ordinary Orthon types with method implementations. |
| Q6 | Composition? | Yes — of delegates, channels, closures, and function calls. |
| Q7 | Sugar over primitives? | Yes — all stream operations are method calls. |
| Q8 | Optimisation? | No. Stream semantics are StdLib patterns, not optimisations. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Universal need (event-driven programming). Zero language additions. |

**Classification per D-03:** StdLib. Push-based reactive streams built on delegate model. No new language constructs.

### Gate Validation

**Gates applied:** `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` (StdLib minimum per `DECISION_VALIDATION.md` § Gate Selection). `LOGICAL_CONSISTENCY_GATE` and `ARCHITECTURAL_INTEGRITY_GATE` also verified per EDR-051.

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer building event-driven systems, I need to react to values as they arrive — network data, UI events, sensor readings — without polling or blocking.

**Press release.** *`Stream<T>` provides a unified push-based reactive model built on delegates and channels — no new language constructs, full combinator support, configurable backpressure.*

**FAQ.** How is a stream different from a generator? A generator is pull-based (consumer asks for the next value); a stream is push-based (producer decides when values arrive). Can I convert a generator to a stream? Yes — a pull→push bridge consumes the iterator and pushes values to subscribers.

**Verdict: Pass.**

---

#### 2. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Push streams are fully expressible as StdLib on the delegate model.

**Verdict: Pass.** `Stream<T>` wraps a delegate + channel. Combinators are function compositions. Subscription is callback registration. Backpressure is channel capacity management. All are ordinary Orthon code.

---

#### 3. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *A stream pushes values to subscribers — like a generator, but the producer decides when values arrive.*

**Verdict: Pass.** StdLib classification means streams can evolve independently. No conceptual debt — the reactive stream pattern is production-proven across RxJS, Kotlin Flow, and Java Stream.

---

#### 4. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `stream.emit(v)` pushes a value. `stream.subscribe(fn)` registers a consumer. `stream.complete()` signals the end. `stream.error(e)` signals an error. Each operation has precisely defined semantics consistent with the delegate and channel models.

**Verdict: Pass.**

---

#### 5. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Streams live in the Standard Library layer, depending on delegates (Core Language) and channels (StdLib).

**Verdict: Pass.** No layer violations.

---

### Overall

All applied gates Pass outright.

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — StdLib addition, not a language construct. `LLM_GENERABILITY_GATE` — StdLib functions have standard LLM generability properties.

---

## Entry: EMIT_AS_INTERMEDIATE_RESULT (EDR-052)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`](../../what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md)
**Decision recorded as:** [EDR-052](../decision_records/architecture/EDR-052-emit-as-intermediate-result.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | A function should publish intermediate results during long-running computation while maintaining a final return value. |
| Q2 | Language/StdLib/Policy? | **Language (semantic refinement).** Builds on LAZY_SEQUENCE_GENERATORS (EDR-021). Clarifies `emit` serves both lazy sequence and intermediate-result publication. |
| Q3 | Existing primitives? | Partially — `emit` mechanism (EDR-021) already supports the pattern. Refinement adds `.final()` accessor. |
| Q4 | Violates principle? | No. Consistent with EDR-021. Adds clarity without changing core semantics. |
| Q5 | New semantics? | **Semantic refinement.** No new runtime semantics. `.final()` is a minor extension. |
| Q6 | Composition? | Yes — `emit` + `return` pattern already supported by EDR-021. |
| Q7 | Sugar over primitives? | Yes — specification refinement of existing mechanism. |
| Q8 | Optimisation? | No. Refinement clarifies existing semantics. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Clarifies dual use of `emit`. Improves specification clarity. |

**Classification per D-03:** Language (semantic refinement). Builds on LAZY_SEQUENCE_GENERATORS (EDR-021).

### Gate Validation

**Gates applied:** `LOGICAL_CONSISTENCY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE` (semantic refinement minimum per `DECISION_VALIDATION.md` § Gate Selection). `CONCEPTUAL_SIMPLICITY_GATE` and `USER_VALUE_GATE` also verified per EDR-052.

#### 1. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** All terms are precisely defined and consistent with EDR-021. The intermediate result model is a conceptual refinement, not a semantic change. `emit` + `return` pattern is already supported by EDR-021; `.final()` accessor is a minor extension to the existing state machine.

**Test with counterexamples.** Does a function with `emit` but no `return` still return `Iterator[T]`? Yes — per EDR-021. Does `.final()` work on a generator that never terminates? It blocks (the function must complete to produce a final value).

**Verdict: Pass.**

---

#### 2. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** The refinement operates entirely within EDR-021's existing model. `emit` remains a Core Language construct. `.final()` is an accessor on the existing `Iterator[T]` type.

**Verdict: Pass.** No layer violations.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Adding the `.final()` accessor and documentation does not increase language complexity.

**Verdict: Pass.** The model is already present in EDR-021 — this EDR makes it explicit. The conceptual unification of lazy sequences and incremental computation under a single `emit` mechanism reduces overall complexity.

---

#### 4. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer processing large datasets, I want to publish intermediate results incrementally during a long computation and return a final summary when done.

**Press release.** *`emit` now serves both lazy sequence production and intermediate result publication — the same keyword unifies both patterns; `.final()` gives access to the computation's final result.*

**Verdict: Pass.**

---

### Overall

All applied gates Pass outright.

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — The refinement does not change implementation requirements. `LONG_TERM_MAINTAINABILITY_GATE` — The `.final()` accessor is a minor extension of the existing state machine pattern. `LLM_GENERABILITY_GATE` — The pattern is intuitive; LLMs familiar with generators understand `emit` + `return` + `.final()`.

---

## Entry: ITERATION_LOOP (EDR-053)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/ITERATION_LOOP.md`](../../what/concepts/ITERATION_LOOP.md)
**Decision recorded as:** [EDR-053](../decision_records/architecture/EDR-053-iteration-loop.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | How to express repeated execution over a sequence of values. `for`/`while` loops are fundamental. |
| Q2 | Language/StdLib/Policy? | **Language.** `for`, `while`, `loop` keywords require compiler-level syntax, desugaring, and control-flow (break, continue). |
| Q3 | Existing primitives? | No — loop constructs require compiler-level syntax and control-flow. |
| Q4 | Violates principle? | No. Minimal Core (one iteration construct + `while`), Explicit Semantics (`break`/`continue` are visible). |
| Q5 | New semantics? | **New semantics.** Loop control flow (break, continue), `for` desugaring to iterator protocol, `while` condition evaluation. |
| Q6 | Composition? | No — loop desugaring requires compiler support. |
| Q7 | Sugar over primitives? | `for` desugars to `IntoIterator` + `loop` + `match` + `next()`. Desugaring requires compiler support. |
| Q8 | Optimisation? | No. Loop semantics are fundamental control flow. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for any practical language. Simplest, safest iteration model. No C-style `for (;;)`. |

**Classification per D-03:** Language. `for`/`while`/`loop` constructs require compiler-level syntax and desugaring.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I need to iterate over sequences of values and evaluate conditions for looping — `for ... in` and `while` are fundamental control flow constructs.

**Press release.** *One iteration construct (`for ... in`), one condition construct (`while`), and one infinite-loop construct (`loop`) — the simplest, safest iteration model. No C-style `for (;;)`, no off-by-one errors.*

**FAQ.** How do I iterate with an index? Use range syntax: `for i in 0..len(array)`. Is `break value` supported? Yes — in `loop` constructs (Rust-style expression-oriented infinite loops).

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `for ... in` desugars to iterator protocol (`IntoIterator::iter()` + `loop` + `next()`). `while` operates on a boolean condition. `break` and `continue` have defined semantics in both `for` and `while`. `loop { ... }` executes forever unless `break` is called.

**Test with counterexamples.** Does `for` work with non-iterable types? Compile error. Does `while` require an iterable? No — it operates on a boolean condition. Can `break` take a value outside `loop`? No — `break value` is only valid in `loop` constructs.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** The loop model is minimal — one iteration construct, one condition construct, one infinite construct.

**Verdict: Pass.** Removing `for` would force `while` + manual iterator management. Removing `while` would force `loop` + conditional `break`. Each construct serves a distinct purpose.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** `for` desugars to the iterator protocol (EDR-022) within the same Core Language layer. `while` is a primitive control flow construct.

**Verdict: Pass.** No layer violations.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Loop semantics seem to require specific compiler transformations (desugaring, control-flow graph construction), yet must be implementation-independent.

**Apply separation.** Loop semantics are strategy-independent — iteration follows the `next()` protocol regardless of allocation or evaluation strategy. `for` desugaring is a syntactic transformation applicable in any implementation.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *`for item in sequence` iterates over the sequence; `while condition` loops until false.*

**Verdict: Pass.** The model matches Python, Rust, and Swift — proven across decades. `loop` with `break value` for expression-oriented infinite looping follows Rust's proven pattern.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `for ... in` and `while` are among the most LLM-generable constructs.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Loop syntax is fully expressible in the grammar |
| Predictable generation (≥90%) | Pass | `for ... in` and `while` are universal across languages |
| No hallucination surface | Pass | Single unambiguous meaning per keyword |
| Strategy-aware default | Pass | Loop desugaring is a fixed compiler transformation |
| Self-correctable | Pass | Non-iterable source in `for` is a compile error; non-boolean condition in `while` is a type error |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: OBJECT_INITIALIZATION (EDR-054)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/OBJECT_INITIALIZATION.md`](../../what/concepts/OBJECT_INITIALIZATION.md)
**Decision recorded as:** [EDR-054](../decision_records/architecture/EDR-054-object-initialization.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Creating objects with many optional fields without telescoping constructors or boilerplate builder patterns. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Named parameters and defaults are part of the general function call model. Copy-and-update is sugar over `new` + field assignment. |
| Q3 | Existing primitives? | Yes — named parameters already in function call model. Copy-and-update desugars to `new` + field assignment. |
| Q4 | Violates principle? | No. Minimal Core, Explicit Semantics. |
| Q5 | New semantics? | **No new semantics.** All patterns are sugar over existing mechanisms. |
| Q6 | Composition? | Yes — of existing function call, type declaration, and macro mechanisms. |
| Q7 | Sugar over primitives? | Yes — copy-and-update desugars to `new` + field assignments. |
| Q8 | Optimisation? | No — StdLib pattern, not optimisation. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Solves genuine ergonomic problem. Zero language cost. |

**Classification per D-03:** StdLib. Constructor patterns, builder patterns, copy-and-update syntax — all StdLib / macro features.

### Gate Validation

**Gates applied:** `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` (StdLib minimum per `DECISION_VALIDATION.md` § Gate Selection). `LOGICAL_CONSISTENCY_GATE` and `ARCHITECTURAL_INTEGRITY_GATE` also verified per EDR-054.

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to create objects with many optional fields without writing telescoping constructors or separate builder classes.

**Press release.** *Named parameters with defaults let you create objects with the fields you need — copy-and-update syntax provides immutable modifications; `@builder` macro generates fluent builder APIs on demand.*

**FAQ.** What happens if I omit a required field? Compile-time error — all required fields are enforced. Is copy-and-update deep or shallow? It follows the value-semantics model — logical copy with specified field replacements.

**Verdict: Pass.**

---

#### 2. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** All features use existing language mechanisms — no new semantics required.

**Verdict: Pass.** Named parameters are already part of the function call model. Copy-and-update is sugar over `new` + field assignment. Builder is a macro pattern (EDR-029).

---

#### 3. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Create objects with named parameters; omit optional ones.*

**Verdict: Pass.** The model matches Kotlin, Swift, and TypeScript — proven patterns. Zero language additions means zero long-term surface commitment.

---

#### 4. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Named parameters have consistent semantics with positional ones. Default values follow the same rules as any parameter default. Copy-and-update is symmetric with named-parameter construction.

**Verdict: Pass.**

---

#### 5. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** All patterns use existing Core Language mechanisms — function calls, type declarations, macros.

**Verdict: Pass.** No layer violations.

---

### Overall

All applied gates Pass outright.

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — StdLib addition; implementation concerns covered by existing mechanisms. `LLM_GENERABILITY_GATE` — Named parameters are standard; LLMs generate them reliably.

---

## Entry: UNPACKING (EDR-055)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/UNPACKING.md`](../../what/concepts/UNPACKING.md)
**Decision recorded as:** [EDR-055](../decision_records/architecture/EDR-055-unpacking.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Concise extraction of values from compound data structures without verbose indexing. |
| Q2 | Language/StdLib/Policy? | **Language.** Destructuring patterns require compiler-level parsing and desugaring to `pack`/`unpack` primitives. |
| Q3 | Existing primitives? | Semantically yes — `unpack(point)` exists as primitive. But destructuring syntax needs compiler-level pattern recognition. |
| Q4 | Violates principle? | No. Representation Symmetry (construction/destruction share syntax), Minimal Core (desugars to existing primitives). |
| Q5 | New semantics? | **Syntactic sugar.** All forms desugar to `pack`/`unpack` primitives. No new runtime semantics. |
| Q6 | Composition? | Yes — `let (x, y) = point` desugars to `let x, y = unpack(point)`. |
| Q7 | Sugar over primitives? | **Yes** — destructuring is syntax over `pack`/`unpack`. |
| Q8 | Optimisation? | No — syntactic convenience. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for ergonomic data decomposition. Symmetric with `pack` construction. |

**Classification per D-03:** Language. Destructuring syntax matching pack/unpack symmetry. All forms desugar to `pack`/`unpack` — no new runtime semantics.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to extract values from compound data structures — tuples, records, arrays — without verbose indexing or manual unpacking.

**Press release.** *Destructuring assignment mirrors construction syntax: `let (x, y) = point` uses the same shape as `let point = (x, y)`. Tuple, record, nested, and function-parameter destructuring — all desugar to `pack`/`unpack` primitives.*

**FAQ.** How does ownership work in destructuring? Destructuring from a value moves; destructuring from a reference borrows (per PATTERN_MATCHING semantics). Can I ignore fields? Yes — `_` for single positions, `..` for rest.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Destructuring syntax mirrors pattern-matching syntax (EDR-025) — consistent across the language. The `pack`/`unpack` symmetry is maintained: `(x, y)` creates a tuple, `let (x, y) = ...` decomposes one.

**Test with counterexamples.** Does `let {name, age} = person` work on any record type? Yes — field names must match. Does nested destructuring compose? Yes — `let {address: {city}} = user` decomposes recursively.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Destructuring is syntactic sugar over `pack`/`unpack` — removing it loses ergonomics, not expressive power.

**Verdict: Pass.** Every destructuring form desugars to existing primitives. The ergonomic gain (concise extraction) justifies the syntax.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Destructuring composes Level 1 primitives (`pack`/`unpack`, `identifier`, `assignment`) into a Level 2 Language Pattern.

**Verdict: Pass.** No layer violations.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Destructuring requires the compiler to recognise pattern syntax and generate field-by-field extraction code, yet must work across all implementation strategies.

**Apply separation.** Destructuring is a syntactic transformation — strategy-independent. Field-by-field extraction follows the same semantics regardless of allocation or evaluation strategy.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Destructuring lets you unpack compound values using the same syntax used to create them.*

**Verdict: Pass.** Removing destructuring would force manual indexing everywhere. The pattern is proven across Rust, Python, JS, and Swift.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Destructuring is highly LLM-generable — the pattern matches Python, Rust, and JS destructuring.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Pattern syntax is fully expressible in the grammar |
| Predictable generation (≥90%) | Pass | `let (x, y) = tuple` and `let {name, age} = record` are intuitive |
| No hallucination surface | Pass | Single unambiguous meaning per pattern form |
| Strategy-aware default | Pass | Desugaring is a fixed compiler transformation |
| Self-correctable | Pass | Incorrect destructuring patterns (missing field, type mismatch) produce compile errors |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: CONTRACTS (EDR-056)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/CONTRACTS.md`](../../what/concepts/CONTRACTS.md)
**Decision recorded as:** [EDR-056](../decision_records/architecture/EDR-056-contracts.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Verifiable guarantees about function behaviour — typed signatures give structure but not intent. Contracts reduce LLM hallucination of nonsensical calls. |
| Q2 | Language/StdLib/Policy? | **Language.** `requires`, `ensures`, `invariant` keywords require syntactic integration into function signatures and compiler-level enforcement. |
| Q3 | Existing primitives? | No. Contract semantics (caller-visible constraints, `result` postcondition variable) are not expressible via primitive composition. |
| Q4 | Violates principle? | No. Declarative With Static Guarantees, Explicitness, Minimal Core. |
| Q5 | New semantics? | **New semantics.** Pre/postcondition constraints, `result`/`old` implicit variables, invariant bounds, Liskov inheritance rules. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | No — see Q3. |
| Q8 | Optimisation? | No. Contract verification is a semantic guarantee. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for LLM code generation accuracy. Proven model from Eiffel and Ada 2012. |

**Classification per D-03:** Language. Contract keywords require syntactic integration and compiler-level enforcement.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer — and as an LLM generating Orthon code — I need verifiable guarantees about function behaviour beyond the type signature. Types give structure; contracts give intent.

**Press release.** *`requires`, `ensures`, and `invariant` are first-class function signature keywords — compiler-verified where possible, runtime-asserted otherwise. LLMs generate more accurate code when contracts document intent at the declaration site.*

**FAQ.** Are contracts checked at compile time? Where possible, yes — static conditions are verified by the compiler. Dynamic conditions degrade to runtime assertions. Are contracts evaluated in release builds? Only if `--enforce-contracts` is passed.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** `requires` = precondition (must hold before call). `ensures` = postcondition (must hold after call). `invariant` = class/type invariant (must hold before and after each public operation). `result` = implicit variable for postcondition value. `old` = pre-call value of an expression.

**Test with counterexamples.** Can a contract refer to the function's result? Yes — via the implicit `result` variable. Can a contract have side effects? No — contract expressions must pass purity analysis.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Three keywords (`requires`, `ensures`, `invariant`) cover the essential contract types — no fourth is needed.

**Verdict: Pass.** Preconditions, postconditions, and invariants are the three canonical contract forms (Eiffel, Ada 2012). Higher-order contracts (contracts on function arguments) are deferred to v0.2.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Contracts are part of the function signature — they compose with the existing declaration syntax. Pure analysis is a compiler check.

**Verdict: Pass.** Contracts operate within the existing signature model. Contract inheritance follows Liskov substitution rules.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Contracts must be verifiable (requiring compiler analysis) but strategy-independent.

**Apply separation.** Contract *semantics* are independent of any specific checker implementation. Static verification depth varies by strategy — the Default Strategy may check more conditions than the Embedded Strategy — but the semantic model (`requires`/`ensures`/`invariant`) is universal.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Contracts document intent at the declaration site so the compiler — and LLMs — know what a function guarantees.*

**Verdict: Pass.** Contracts improve maintainability by making implicit assumptions explicit. The proven model (Eiffel, Ada 2012) has decades of production use.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Contracts reduce LLM error rates by providing semantic intent alongside type structure.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Contract syntax is part of the function signature grammar |
| Predictable generation (≥90%) | Pass | `requires`/`ensures`/`invariant` follow obvious patterns |
| No hallucination surface | Pass | Each keyword has exactly one semantic meaning |
| Strategy-aware default | Pass | Static verification is the default; dynamic fallback is universal |
| Self-correctable | Pass | Impure contract expressions are compile errors; postcondition referring to undefined variables is a compile error |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: DELEGATION (EDR-057)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/DELEGATION.md`](../../what/concepts/DELEGATION.md)
**Decision recorded as:** [EDR-057](../decision_records/architecture/EDR-057-delegation.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Boilerplate forwarding when a type reuses behaviour from another type without inheritance (class delegation, property delegation). |
| Q2 | Language/StdLib/Policy? | **StdLib.** `@delegate` macro desugars to explicit `impl` blocks using existing macro system (EDR-029). |
| Q3 | Existing primitives? | Yes — delegation is expressible via trait implementation + manual method forwarding. Macro automates the pattern. |
| Q4 | Violates principle? | No. Composition Over Inheritance, Minimal Core (reuses macro system), Explicitness. |
| Q5 | New semantics? | **No new semantics.** `@delegate` desugars to explicit `impl` blocks. |
| Q6 | Composition? | Yes — of trait `impl` blocks + method forwarding. |
| Q7 | Sugar over primitives? | Yes — `@delegate` is sugar over macro-generated `impl` blocks. |
| Q8 | Optimisation? | No. Delegation is a code organisation pattern. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Eliminates boilerplate forwarding. Zero language additions. |

**Classification per D-03:** StdLib. Delegation expressible via composition. Macro system automates the pattern.

### Gate Validation

**Gates applied:** All 7 (per EDR-057 — StdLib addition with broader cross-cutting implications verified against the full catalogue).

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to reuse behaviour from another type without writing boilerplate forwarding methods or using inheritance.

**Press release.** *`@delegate` auto-generates forwarding `impl` blocks — class delegation and property delegation without boilerplate. Explicit override: your methods take precedence over delegated ones.*

**FAQ.** What's the difference between `@delegate` and `delegate` (the execution model)? Complete — the execution `delegate` creates concurrent execution contexts (EDR-036). `@delegate` composes types by forwarding method calls. Orthogonal concepts, similar naming.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Class delegation: type T delegates interface I to field F — all methods of I on T forward to F. Property delegation: getter/setter delegated to a helper object following `Get`/`Set` protocol. Forwarding semantics are well-defined — explicit methods always win over delegated ones.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One macro keyword (`@delegate`) covers both class delegation and property delegation.

**Verdict: Pass.** No additional keywords needed. The macro system (EDR-029) provides the implementation mechanism.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Delegation builds on the existing macro system (EDR-029) and trait `impl` mechanism.

**Verdict: Pass.** No new compiler-level constructs. The macro generates standard forwarding code.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Macro-generated forwarding code depends on compile-time macro execution (EDR-031), yet delegation semantics must be strategy-independent.

**Apply separation.** Delegation *semantics* (method A forwards to method B on the delegate) are independent of macro implementation. The macro is a code-generation convenience; the forwarding code it produces is ordinary Orthon.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *One annotation replaces boilerplate forwarding methods.*

**Verdict: Pass.** StdLib-based means no long-term language surface commitment. The pattern is well-understood from Kotlin's `by` keyword and Go's embedding.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** One annotation with clear semantics.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `@delegate(Trait) to field` is a simple annotation grammar |
| Predictable generation (≥90%) | Pass | One annotation — easy for LLMs to generate correctly |
| No hallucination surface | Pass | Single clear purpose: method forwarding |
| Strategy-aware default | Pass | Macro expansion is compile-time only |
| Self-correctable | Pass | Missing trait implementation on delegate field is a compile error |

**Verdict: Pass.**

---

### Overall

All applied gates Pass outright.

**Gates not applied:** None.

---

## Entry: EXTENSION_FUNCTIONS (EDR-058)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/EXTENSION_FUNCTIONS.md`](../../what/concepts/EXTENSION_FUNCTIONS.md)
**Decision recorded as:** [EDR-058](../decision_records/architecture/EDR-058-extension-functions.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Adding behaviour to existing types without modifying source, inheritance, or adapter wrapping. Method syntax is more natural. |
| Q2 | Language/StdLib/Policy? | **Language.** Extension functions require compiler to recognize receiver-call syntax on types from other modules and resolve dispatch. |
| Q3 | Existing primitives? | No. Receiver-call syntax on external types requires name resolution rules and static dispatch. |
| Q4 | Violates principle? | No. Explicit Semantics (extension import required, member precedence documented), Intent Over Implementation. |
| Q5 | New semantics? | **New semantics.** Extension function resolution — compiler finds correct definition based on receiver type, import scope, precedence. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | No — requires compiler-level name resolution and dispatch. |
| Q8 | Optimisation? | No. Dispatch determines which function runs — semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Proven in Kotlin, C#, Swift — natural extension mechanism. |

**Classification per D-03:** Language. Extension functions require compiler-level name resolution and static dispatch.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to add methods to types I don't own — StdLib types, third-party types, even primitives — using natural receiver-call syntax.

**Press release.** *`fun Type.method() = expr` defines a function on `Type` from anywhere — import it explicitly, call it with receiver syntax, and know that member functions always take precedence.*

**FAQ.** Do extension functions have access to private members? No — they respect encapsulation. Is dispatch static or dynamic? Static — based on the receiver's static type. Can two imports define the same extension? Yes — ambiguity is a compile error.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Extension function: a function defined outside the receiver type, called with receiver syntax. Static dispatch: resolution based on the static type of the receiver. Member precedence: a member function always shadows an extension function of the same signature. Explicit import: extension functions from other packages require an import statement.

**Test with counterexamples.** Can extension functions satisfy trait bounds? No — they do not participate in trait/interface satisfaction. Does this limit their usefulness? Yes — trait-based extension (Rust-style) is more powerful but requires more ceremony.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One concept — function defined outside type, called with receiver syntax — covers all extension use cases.

**Verdict: Pass.** Extension properties are supported (computed only, no backing fields) following the same resolution rules.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Extension functions compose with the name resolution and import system. They do not modify the target type.

**Verdict: Pass.** No layer violations. Extension functions live at the language level, building on existing resolution infrastructure.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Extension function resolution seems to require specific name resolution algorithms (scope-based lookup), yet must be implementation-independent.

**Apply separation.** The *semantic rule* (receiver type → import scope → member precedence) is strategy-independent. Any concrete resolution algorithm that satisfies this rule is valid.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *A function defined outside a type can be called with receiver syntax on values of that type.*

**Verdict: Pass.** Well-understood from Kotlin, C#, and Swift — a proven pattern. Import control prevents namespace pollution.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Obvious syntax — `fun Type.method()` — LLMs generate correctly from natural language descriptions.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Extension function syntax is part of the function declaration grammar |
| Predictable generation (≥90%) | Pass | `fun Type.method()` is intuitive and matches Kotlin/C# patterns |
| No hallucination surface | Pass | Clear precedence rules (member > extension) |
| Strategy-aware default | Pass | Static dispatch is the universal default |
| Self-correctable | Pass | Ambiguous extension resolution is a compile error |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: GRADUAL_TYPING (EDR-059)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/GRADUAL_TYPING.md`](../../what/concepts/GRADUAL_TYPING.md)
**Decision recorded as:** [EDR-059](../decision_records/architecture/EDR-059-gradual-typing.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | The same programmer needs dynamic speed while prototyping and static safety while shipping. |
| Q2 | Language/StdLib/Policy? | **Language.** Selective type checking, boundary checks at typed/untyped interfaces, consistency passes require compiler infrastructure. |
| Q3 | Existing primitives? | No. Selectively enabled/disabled type checking at module boundaries is not expressible via primitives. |
| Q4 | Violates principle? | No. Minimal Core (one concept covers spectrum), Intent Over Implementation (start dynamic, add types as code matures). |
| Q5 | New semantics? | **New semantics.** Boundary checking, type inference on unannotated code, global consistency pass as optional lint. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | No — see Q3. |
| Q8 | Optimisation? | No. Type checking determines program correctness — semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Critical for LLM adoption — starts with minimal annotations; compiler catches structural errors. Proven in TypeScript. |

**Classification per D-03:** Language. Optional type annotations with selectively enabled checking. Boundary checks, inference, consistency passes.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to start prototyping fast — no type annotations, no ceremony — and add types incrementally as my codebase matures, without switching languages or tools.

**Press release.** *Same language from prototype to production — start fully dynamic, add type annotations where they provide the most value, and let the compiler verify boundaries between typed and untyped code. No separate declaration files.*

**FAQ.** Is untyped code completely unchecked? No — type inference still provides type information even for unannotated code. How does the boundary check work? At typed/untyped function call interfaces, the compiler inserts runtime checks for type consistency.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Gradual typing: optional type annotations with selectively enabled checking. Boundary check: compiler-verified consistency between typed and untyped code at function call boundaries. Global consistency pass: optional lint that catches type errors across the entire codebase.

**Test with counterexamples.** Can untyped code call a typed function? Yes — the typed function's contract is enforced. Can typed code call untyped code? Yes — the return type is inferred. Does gradual typing affect runtime performance of typed code? No — boundary checks only apply at the interface.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One concept — optional annotations — covers the entire spectrum from dynamic to fully static typing.

**Verdict: Pass.** No separate modes, no pragma directives, no configuration files. The type annotation is the control: present = checked, absent = inferred.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Gradual typing composes with type inference and the module system. The boundary check model requires no changes to the Core Language semantic model.

**Verdict: Pass.** The compiler handles gradual typing as a mode within the existing type system, not as a separate type checker.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Gradual typing seems to require a specific inference algorithm (local type inference for unannotated code, global consistency pass for fully-typed code), yet must be implementation-independent.

**Apply separation.** The *semantic model* (optional annotations, boundary checks) is strategy-independent. The specific inference algorithm is an implementation choice — local inference for rapid prototyping, global analysis for production verification.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Add types where they help; skip them where they don't.*

**Verdict: Flag.** Gradual typing adds compiler complexity — the type system must support both fully-statically-typed and gradually-typed compilation modes. Well-understood from the TypeScript ecosystem, but the complexity is real and must be managed.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Critical for LLM adoption — the LLM can generate minimal-annotation code and let the compiler catch structural errors.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Optional type annotations are part of the general declaration grammar |
| Predictable generation (≥90%) | Pass | LLMs generate good TypeScript — same pattern applies |
| No hallucination surface | Pass | Unannotated code is valid; annotations only add constraints |
| Strategy-aware default | Pass | Start dynamic (no annotations) is the default |
| Self-correctable | Pass | Boundary checks catch mismatches; missing annotations never cause errors |

**Verdict: Pass.**

---

### Overall

Six gates Pass outright. One gate (`LONG_TERM_MAINTAINABILITY_GATE`) returns a Flag — compiler complexity is real, but well-understood from TypeScript's ecosystem-scale validation.

**Gates not applied:** None.

---

## Entry: SMART_CAST (EDR-060)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/SMART_CAST.md`](../../what/concepts/SMART_CAST.md)
**Decision recorded as:** [EDR-060](../decision_records/architecture/EDR-060-smart-cast.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | After checking that a value is a specific type, the programmer should not need to cast again. Compiler tracks type-narrowing through control flow. |
| Q2 | Language/StdLib/Policy? | **Language.** Flow-sensitive type analysis across control flow edges is a compiler-level operation. |
| Q3 | Existing primitives? | No. Flow-sensitive type tracking is not expressible via primitive set. |
| Q4 | Violates principle? | No. Declarative With Static Guarantees, Explicitness (narrowing follows visible checks). |
| Q5 | New semantics? | **New semantics.** Flow-sensitive type narrowing — per-variable type info across control flow edges. |
| Q6 | Composition? | No — see Q3. |
| Q7 | Sugar over primitives? | No — see Q3. |
| Q8 | Optimisation? | No. Type narrowing determines legal operations — semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. Builds on NULL_SAFETY (EDR-018) + PATTERN_MATCHING (EDR-025). |
| Q10 | Worth adding? | **Yes.** Eliminates redundant casts after type checks. Partially subsumed by PATTERN_MATCHING. |

**Classification per D-03:** Language. Flow-sensitive type narrowing across control flow. Partially subsumed by PATTERN_MATCHING.

### Gate Validation

**Gates applied:** All 7 (per `DECISION_VALIDATION.md` § Gate Selection)

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, after checking `if value is Type`, I expect the compiler to know that `value` is now `Type` — no redundant explicit cast needed.

**Press release.** *After `if value is Type`, the compiler narrows `value` to `Type` in the true branch. After `value isnt None`, `Option[T]` unwraps to `T`. Smart cast applies to immutable variables only — conservative by default, never unsound.*

**FAQ.** Does smart cast work across function calls? No — narrowing does not propagate through function boundaries. Does it work on mutable variables? No — only effectively-immutable variables (val, or local vars provably unmodified).

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Smart cast: flow-sensitive type narrowing. Narrowing scope: the compiler determines which control flow paths narrow the type and which reset it. Conservative: if the compiler cannot prove safety, it stays at the wider type.

**Test with counterexamples.** What happens if a narrowed variable is reassigned? Narrowing resets. What happens after `if` with `&&` — does the narrowing compose? Yes — `if value is A && value is B` narrows to the intersection. What about `||`? Each branch narrows independently.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One rule — after type check, type narrows in the checked scope — covers all smart cast scenarios.

**Verdict: Pass.** The rule composes across `if`, `when`, `&&`, `||`, `return`, and `throw`. The explicit cast `as Type` provides an escape hatch.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Smart cast composes with pattern matching (EDR-025) — smart cast handles non-pattern narrowing scenarios that pattern matching doesn't cover.

**Verdict: Pass.** Flow-sensitive analysis is a natural extension of the type checker, not a separate analysis pass.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Flow-sensitive type analysis seems to require a specific control-flow graph representation, yet must be implementation-independent.

**Apply separation.** The *semantic rule* (narrowing follows visible control flow) is strategy-independent. The concrete CFG representation and analysis algorithm are implementation choices.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *After checking a type, the compiler remembers.*

**Verdict: Pass.** Well-understood from Kotlin and TypeScript — proven pattern. Conservative by default means no unsound narrowing surprises.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** LLMs correctly generate type checks expecting narrowing — smart cast makes generated code work without explicit casts.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Flow-sensitive narrowing is a compiler analysis, not a syntactic construct |
| Predictable generation (≥90%) | Pass | LLMs naturally write `if value is Type` — smart cast eliminates the cast they'd forget |
| No hallucination surface | Pass | Narrowing is automatic; the programmer never writes smart cast syntax |
| Strategy-aware default | Pass | Conservative narrowing is the universal default |
| Self-correctable | Pass | If LLM generates a redundant cast, it compiles but is unnecessary — never wrong |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright.

**Gates not applied:** None.

---

## Entry: COPY_ON_WRITE (EDR-061)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/COPY_ON_WRITE.md`](../../what/concepts/COPY_ON_WRITE.md)
**Decision recorded as:** [EDR-061](../decision_records/architecture/EDR-061-copy-on-write.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Memory safety without borrow checker learning curve. Value semantics by default with efficient implementation. |
| Q2 | Language/StdLib/Policy? | **StdLib / Implementation Strategy.** CoW is an optimisation technique for value-semantics collections. |
| Q3 | Existing primitives? | Yes — CoW is an implementation choice for existing value-semantics primitives. Invisible to programmer. |
| Q4 | Violates principle? | No. Intent Over Implementation, Minimal Core. |
| Q5 | New semantics? | **No new semantics.** CoW changes performance, not observable behaviour. |
| Q6 | Composition? | Yes — of assignment, mutation, reference counting. |
| Q7 | Sugar over primitives? | Yes — programmer writes value-semantics code; CoW is invisible. |
| Q8 | Optimisation? | **Yes.** Pure implementation optimisation. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Performance of sharing with safety of copying. Avoids borrow checker complexity. |

**Classification per D-03:** StdLib / Implementation Strategy. Implementation technique for value-semantics collections.

### Gate Validation

**Gates applied:** All 7 (per EDR-061 — implementation strategy with cross-cutting implications verified against the full catalogue).

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want value semantics by default — assignment copies logically — without worrying about performance. CoW makes value semantics fast without a borrow checker.

**Press release.** *Write value-semantics code; CoW handles the performance transparently — sharing is invisible, cloning happens only when mutation would break sharing. No borrow checker learning curve.*

**FAQ.** When does CoW trigger a clone? On the first mutation of a shared reference-counted value. Is the clone deep or shallow? Shallow — only the immediate container is cloned; children remain shared until they are mutated in turn.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** CoW: copy-on-write — a memory optimisation where assignment shares data and mutation triggers a clone only when data is shared. Value semantics: assignment is a logical copy; mutation does not affect other aliases. `shared`: explicit reference semantics with RC-based sharing.

**Test with counterexamples.** Does CoW change observable behaviour? No — it is transparent. Does CoW interact correctly with `shared`? Yes — `shared` uses RC-based sharing instead of CoW, making identity sharing syntactically visible.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One optimisation technique covers the default collection strategy.

**Verdict: Pass.** No separate mechanisms for different collection types. CoW is the DEFAULT_STRATEGY's mechanism for implementing value semantics on standard collections.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** CoW is an implementation choice within the Strategy system — it is not exposed in the language surface.

**Verdict: Pass.** The programmer never writes CoW-specific code. The SEMANTIC_MODEL specifies value semantics by default; CoW is how the runtime achieves this efficiently.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** CoW appears to be a specific implementation technique (reference counting + copy-on-mutation), yet value semantics must be implementable across multiple strategies (Default, Embedded, High-Performance).

**Apply separation.** Value *semantics* are strategy-independent: `let b = a` means `b` is a logical copy. CoW is one *implementation* of that semantic. The Embedded Strategy might use eager copying; the High-Performance Strategy might use region-based allocation. The observable behaviour is identical.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Assignment creates a logical copy; CoW makes it fast.*

**Verdict: Pass.** CoW is a well-understood technique with decades of production use (Swift, Clojure, many systems). No conceptual debt — it is pure optimisation.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** CoW is transparent — LLMs never write CoW-specific code.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | CoW is invisible — the programmer writes standard value-semantics code |
| Predictable generation (≥90%) | Pass | LLMs generate value-semantics code naturally; CoW is an invisible optimisation |
| No hallucination surface | Pass | CoW introduces no syntax or API that an LLM could misuse |
| Strategy-aware default | Pass | CoW is the DEFAULT_STRATEGY; other strategies use different mechanisms transparently |
| Self-correctable | Pass | The programmer never writes CoW-specific code — nothing to get wrong |

**Verdict: Pass.**

---

### Overall

All applied gates Pass outright.

**Gates not applied:** None.

---

## Entry: PROPERTIES (EDR-062)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/PROPERTIES.md`](../../what/concepts/PROPERTIES.md)
**Decision recorded as:** [EDR-062](../decision_records/architecture/EDR-062-properties.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Exposing data without coupling consumers to internal representation. Stored and computed values should use same call syntax. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Properties desugar to field access + function calls. |
| Q3 | Existing primitives? | Yes — attribute access + function calls already provide the primitive operations. |
| Q4 | Violates principle? | No. Minimal Core, Uniform Access. |
| Q5 | New semantics? | **No new semantics.** Implicit getter is sugar over attribute access + function. |
| Q6 | Composition? | Yes — of attribute access + function calls. |
| Q7 | Sugar over primitives? | **Yes** — properties are sugar over field access + getter/setter calls. |
| Q8 | Optimisation? | No. Code organisation pattern. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Proven in C#, Kotlin, Swift — eliminates refactoring friction. |

**Classification per D-03:** StdLib. Properties desugar to field access + function calls. No new compiler-level semantics.

### Gate Validation

**Gates applied:** All 7 (per EDR-062 — StdLib addition with broader cross-cutting implications verified against the full catalogue).

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer, I want to expose data without coupling consumers to internal representation — stored fields and computed values should use the same `.name` call syntax.

**Press release.** *Every field is implicitly a property with a getter and optional setter. Stored and computed properties are indistinguishable at the call site — changing a field to a computation never changes the call site. No refactoring friction.*

**FAQ.** Can a property have a getter without a setter? Yes — read-only properties are the default (consistent with `val` semantics). Can a property have a setter without a getter? No — every property must be readable.

**Verdict: Pass.**

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** Property: a named value with an optional getter/setter. Implicit getter: auto-generated getter that returns the backing field. Computed property: property with an explicit getter body. Uniform access: `obj.name` syntax regardless of stored vs. computed.

**Test with counterexamples.** Can a computed property reference other fields? Yes — arbitrary expressions are valid in getters. Does changing a stored property to a computed one break callers? No — uniform access guarantees backward compatibility.

**Verdict: Pass.**

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** One concept — named value with optional computed getter/setter — covers all property use cases.

**Verdict: Pass.** Properties desugar to existing primitives (field access + function calls). No new compiler-level runtime behaviour.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.** Properties desugar to existing primitives — they are a syntactic convenience, not an architectural addition.

**Verdict: Pass.** No architectural impact. Properties operate entirely within the existing field access and function call model.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** The implicit getter generation seems to require compiler-level desugaring, yet the concept is classified as StdLib.

**Apply separation.** Property *syntax* (implicit getter) is sugar over existing primitives — the desugaring is a fixed transformation independent of any specific runtime. The StdLib classification refers to the property *pattern*, not the desugaring mechanism.

**Verdict: Pass.**

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *A field with optional getter/setter — same call syntax either way.*

**Verdict: Pass.** Proven pattern from C#, Kotlin, and Swift — lowest-risk addition. Uniform access eliminates the most common refactoring friction point (changing a field to a computation).

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** Obvious syntax — `field: Type { get: ... set: ... }`.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | Property syntax is part of the type declaration grammar |
| Predictable generation (≥90%) | Pass | `field: Type { get: expr }` is intuitive and matches Kotlin/C# |
| No hallucination surface | Pass | Properties are a direct superset of field semantics |
| Strategy-aware default | Pass | Implicit getter is the universal default |
| Self-correctable | Pass | Missing getter (if computed) or type mismatch is a compile error |

**Verdict: Pass.**

---

### Overall

All applied gates Pass outright.

**Gates not applied:** None.

---

## Entry: SLOTS (EDR-063)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/SLOTS.md`](../../what/concepts/SLOTS.md)
**Decision recorded as:** [EDR-063](../decision_records/architecture/EDR-063-slots.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Fixed-field storage for types — preventing accidental attribute creation, ensuring ABI-stable layout, eliminating per-instance dictionary overhead. |
| Q2 | Language/StdLib/Policy? | **Language.** Fixed-field verification requires compiler-level type checking. |
| Q3 | Existing primitives? | No. Compile-time field-set restriction not expressible via 9-primitive set. |
| Q4 | Violates principle? | No. Fixed fields as default aligns with Explicitness, Safety. |
| Q5 | New semantics? | **No new runtime semantics** — slots are compile-time verification. `dynamic` modifier is annotation on existing type declaration. |
| Q6 | Composition? | Yes — fixed fields = type declaration with compile-time field verification. |
| Q7 | Sugar over primitives? | Yes — slot restriction is compile-time gate over `attribute access` primitive. |
| Q8 | Optimisation? | Slot layout is ABI optimisation. Field verification is safety gate. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Fixed fields as default is simplest, safest model. |

**Classification per D-03:** Language. Fixed-field verification is compiler-level type checking. Dynamic modifier for opt-out.

### Gate Validation

**Gates applied:** All 7 (a new Language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Predictable memory layout without ceremony. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Every declared property implicitly reserves a slot — consistent. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Slots are a consequence of the property model, not a separate feature. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with existing property and type system. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Slot semantics independent of storage implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Minimal surface — slots are implicit in property declarations. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs declare properties; slots follow automatically. |

**Overall:** Pass. All 7 gates clear. Slots are a natural consequence of Orthon's property model with no new conceptual burden.

---

## Entry: SPAN (EDR-064)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/SPAN.md`](../../what/concepts/SPAN.md)
**Decision recorded as:** [EDR-064](../decision_records/architecture/EDR-064-span.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Safe, non-owning access to contiguous memory — preventing dangling pointers, bounds violations, implicit copies. |
| Q2 | Language/StdLib/Policy? | **Language.** Lifetime tracking, bounds checking, slice syntax require compiler support. |
| Q3 | Existing primitives? | No. Lifetime tracking to prevent dangling requires compiler-level borrow checking. |
| Q4 | Violates principle? | No. Safety, Explicitness, Minimal Core. |
| Q5 | New semantics? | **New semantics.** Lifetime-tracked borrowed view, bounds-checked access guarantee, sub-span slicing. |
| Q6 | Composition? | No — lifetime tracking through borrow checking is compiler-level. |
| Q7 | Sugar over primitives? | No — see Q6. |
| Q8 | Optimisation? | No. Safe memory access is a semantic guarantee. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for safe, efficient memory view operations. |

**Classification per D-03:** Language. Lifetime-tracked, bounds-checked memory view. Compiler-level borrow-checking integration.

### Gate Validation

**Gates applied:** All 7 (a new Language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes slicing, gets safety without copies. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Lifetime rules are a direct application of ownership model. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept: non-owning view with lifetime. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Span composes with existing collection and slice patterns. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Span semantics independent of lifetime tracking mechanism. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood from Rust `&[T]` and C++ `std::span`. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Obvious semantics — LLMs correctly generate slicing expecting a view. |

**Overall:** Pass. All 7 gates clear. Span provides safe memory access with well-understood semantics from existing languages.

---

## Entry: NAMED_AND_OPTIONAL_PARAMETERS (EDR-065)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md`](../../what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md)
**Decision recorded as:** [EDR-065](../decision_records/architecture/EDR-065-named-and-optional-parameters.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Call-site readability for functions with many parameters — boilerplate overload explosion for optional parameters. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Named arguments desugar to positional calls. Defaults are ordinary expressions. |
| Q3 | Existing primitives? | Yes — named args = `call` with argument-name matching (desugars to positional `call`). |
| Q4 | Violates principle? | No. Minimal Core, Explicitness. |
| Q5 | New semantics? | **No new semantics.** Sugar over existing function call mechanism. |
| Q6 | Composition? | Yes — of `call` primitive + `identifier` + desugaring. |
| Q7 | Sugar over primitives? | Yes — named args desugar to positional `call`. |
| Q8 | Optimisation? | No — call syntax is syntactic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for ergonomic APIs with many parameters. |

**Classification per D-03:** StdLib. Sugar over existing function call mechanism. Macro-based desugaring.

### Gate Validation

**Gates applied:** All 7 (StdLib features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Self-documenting call sites — direct readability improvement. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Named resolution has well-defined rules — no ambiguity. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept — named binding — covers both features. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Desugars to positional parameters — no architectural impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Semantics independent of argument-passing implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Proven from Python, C#, Kotlin, Swift — lowest-risk feature. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Named arguments are the most LLM-friendly call syntax — explicit mapping from name to value. |

**Overall:** Pass. All 7 gates clear. Named and optional parameters are proven sugar with zero architectural risk.

---

## Entry: SORTING (EDR-066)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/SORTING.md`](../../what/concepts/SORTING.md)
**Decision recorded as:** [EDR-066](../decision_records/architecture/EDR-066-sorting.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Sorting stability — multi-key sort pipelines require predictable relative ordering of equal elements. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Sort algorithms are method implementations on collection types. |
| Q3 | Existing primitives? | Yes — sort = `function` on collection + `call` to `Ord` comparison. |
| Q4 | Violates principle? | No. Minimal Core, Deterministic Behavior. |
| Q5 | New semantics? | **No new semantics.** Function composition. Stability is specification property. |
| Q6 | Composition? | Yes — of comparison operations (`==`, `<` per EDR-017) + collection reordering. |
| Q7 | Sugar over primitives? | Yes — sort methods are function calls. |
| Q8 | Optimisation? | Algorithm selection is optimisation. Sort operation is semantics. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential — sorting is universal. |

**Classification per D-03:** StdLib. Sort algorithm as method implementation. Algorithm selection is Implementation Policy.

### Gate Validation

**Gates applied:** All 7 (StdLib features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Stable sort by default — least surprise. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Stability composes predictably. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One default algorithm, one opt-in variant. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | StdLib method on collection types — no architectural impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Sort policy can change without affecting language semantics. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Timsort is proven across Python, Java, and Android. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `list.sort()` is the most LLM-obvious API. |

**Overall:** Pass. All 7 gates clear. Stable sort by default is the simplest, most proven approach.

---

## Entry: DECLARATIVE_MULTI_KEY_SORT (EDR-067)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/DECLARATIVE_MULTI_KEY_SORT.md`](../../what/concepts/DECLARATIVE_MULTI_KEY_SORT.md)
**Decision recorded as:** [EDR-067](../decision_records/architecture/EDR-067-declarative-multi-key-sort.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Manual comparator chain boilerplate for multi-field sorting with mixed directions. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Sugar over SORTING + EQUALITY. Tuple-as-key lexicographic comparison uses existing `Ord` trait. |
| Q3 | Existing primitives? | Yes — `sorted(by: [.a, .b])` desugars to lexicographic comparison using `Ord` + tuple comparison. |
| Q4 | Violates principle? | No. Intent Over Implementation (specify sort keys, not comparators). |
| Q5 | New semantics? | **No new semantics.** Sugar over existing `Ord` trait comparisons. |
| Q6 | Composition? | Yes — of `Ord` comparisons on tuples. |
| Q7 | Sugar over primitives? | Yes — declarative key paths desugar to `Ord` comparator construction. |
| Q8 | Optimisation? | No — declarative API is a convenience. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Eliminates comparator chain boilerplate. |

**Classification per D-03:** StdLib. Sugar over SORTING + EQUALITY. No new language semantics.

### Gate Validation

**Gates applied:** All 7 (StdLib features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | `sorted(by: .field)` is the natural reading. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Lexicographic comparison of key paths is deterministic. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One API with variadic key paths — no overload explosion. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Pure StdLib — builds on SORTING. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Comparator construction independent of sort algorithm. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Low surface area — one API method. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `sorted(by: .field)` is the most LLM-obvious declarative API shape. |

**Overall:** Pass. All 7 gates clear. Declarative multi-key sort eliminates comparator boilerplate with zero new semantics.

---

## Entry: IMMUTABLE_DATE_TIME (EDR-068)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/IMMUTABLE_DATE_TIME.md`](../../what/concepts/IMMUTABLE_DATE_TIME.md)
**Decision recorded as:** [EDR-068](../decision_records/architecture/EDR-068-immutable-date-time.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Mutable date/time bugs — thread-unsafe formatters, surprising mutation semantics, inconsistent API design. |
| Q2 | Language/StdLib/Policy? | **StdLib.** Date/time types are library types. Immutability enforced at type level. |
| Q3 | Existing primitives? | Yes — each type = `pack` (fields) + `function` (arithmetic, formatting). Immutability = no mutating methods. |
| Q4 | Violates principle? | No. Data-First model, Safety. |
| Q5 | New semantics? | **No new semantics.** Arithmetic is function composition. Parsing returns `Result<T>`. |
| Q6 | Composition? | Yes — of `pack`, `function`, `call`, `===`. |
| Q7 | Sugar over primitives? | Yes — all methods are function calls. |
| Q8 | Optimisation? | No — temporal arithmetic is semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Universal need. Immutable-by-default eliminates common date/time bugs. |

**Classification per D-03:** StdLib. Value-semantics date/time types. Immutable by construction.

### Gate Validation

**Gates applied:** All 7 (StdLib features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Predictable date arithmetic — no surprises. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Value semantics for time — `yesterday = today - 1`. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One set of immutable types covers all date/time needs. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Pure library — no language impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Date/time implementation independent of any runtime choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood domain — proven from Java `java.time` and Noda Time. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs correctly generate immutable date/time operations. |

**Overall:** Pass. All 7 gates clear. Immutable date/time types follow the proven `java.time`/Noda Time model with zero language impact.

---

## Entry: PERSISTENT_DATA_STRUCTURES (EDR-069)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/PERSISTENT_DATA_STRUCTURES.md`](../../what/concepts/PERSISTENT_DATA_STRUCTURES.md)
**Decision recorded as:** [EDR-069](../decision_records/architecture/EDR-069-persistent-data-structures.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Hash-key safe, concurrent-safe, fully immutable collections with structural sharing across versions. |
| Q2 | Language/StdLib/Policy? | **StdLib (deferred to v0.2).** `Immutable` marker trait for compiler optimisation hooks. Collection types are library implementations. |
| Q3 | Existing primitives? | Partially — `Immutable` marker trait is compile-time guarantee. Algorithms are library. |
| Q4 | Violates principle? | No. Minimal Core, Safety. |
| Q5 | New semantics? | **Interface contract only in v0.1.** `Immutable` trait is a guarantee. Full implementations deferred to v0.2. |
| Q6 | Composition? | Yes — `Immutable` trait = trait with no methods. |
| Q7 | Sugar over primitives? | Yes — all collection operations are method calls. |
| Q8 | Optimisation? | Structural sharing is implementation technique for value semantics. |
| Q9 | Backward compat? | N/A — v0.1 uses Tuple + CoW. `Immutable` trait is forward-compatible. |
| Q10 | Worth adding? | **Yes, but deferred.** v0.1 uses Tuple + CoW. `Immutable` trait accepted as forward contract. |

**Classification per D-03:** StdLib (deferred to v0.2). `Immutable` marker trait + persistent collection types. v0.1 uses Tuple + CoW.

### Gate Validation

**Gates applied:** All 7 (StdLib features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Safe concurrent access without locks — direct programmer need. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Value semantics for collections — consistent with rest of the model. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One `Immutable` marker + library types covers the gap. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | `Immutable` marker uses existing trait system (EDR-019). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Structural sharing is an implementation detail — interface is value semantics. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood from Clojure, Scala, and functional programming. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs correctly use immutable collections for thread-safe and key-safe scenarios. |

**Overall:** Pass, deferred to v0.2. All 7 gates clear. `Immutable` marker trait accepted as forward contract; full implementations deferred. v0.1 uses Tuple + CoW.

---

## Entry: DERIVE_SERIALIZATION (EDR-070)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/DERIVE_SERIALIZATION.md`](../../what/concepts/DERIVE_SERIALIZATION.md)
**Decision recorded as:** [EDR-070](../decision_records/architecture/EDR-070-derive-serialization.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Manual recursive serialization code — error-prone, type-unsafe, maintenance burden. |
| Q2 | Language/StdLib/Policy? | **StdLib / Macro.** `@derive(Serialize, Deserialize)` uses existing macro system (EDR-029). |
| Q3 | Existing primitives? | Yes — `Serialize`/`Deserialize` traits + `@derive` macro generation. |
| Q4 | Violates principle? | No. Minimal Core, Explicitness. |
| Q5 | New semantics? | **No new semantics.** Serialization is trait implementation generation. |
| Q6 | Composition? | Yes — of existing primitives + `@derive` macro. |
| Q7 | Sugar over primitives? | Yes — `@derive(Serialize)` desugars to macro-generated `impl` blocks. |
| Q8 | Optimisation? | No — serialization format is semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Eliminates most common boilerplate + bug surface. |

**Classification per D-03:** StdLib / Macro. `@derive(Serialize, Deserialize)` via existing macro system (EDR-029). Format-agnostic.

### Gate Validation

**Gates applied:** All 7 (StdLib / Macro features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | `@derive(Serialize)` — one annotation eliminates all serialization boilerplate. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Serialization follows structural type shape — deterministic mapping. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One annotation — `@derive` — covers all format backends. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on existing `@derive` macro system (EDR-029). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Serialization trait is independent of format backend implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Derive-based serialization is proven in Rust (serde) and Swift (Codable). |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `@derive(Serialize)` is the most LLM-obvious annotation — one declarative line eliminates an entire category of generation errors. |

**Overall:** Pass. All 7 gates clear. Derive-based serialization builds on existing macro infrastructure with proven ergonomics from Rust and Swift.

---

## Entry: COMMAND_PATTERN_VIA_DELEGATE (EDR-071)

**Date:** 2026-07-27
**Artifact validated:** N/A — existing concept coverage confirmation
**Decision recorded as:** [EDR-071](../decision_records/architecture/EDR-071-command-pattern-via-delegate.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | GoF Command pattern requires class-per-command in languages without first-class functions. |
| Q2 | Language/StdLib/Policy? | **Not a new feature.** Existing delegate model (EDR-033) and first-class functions subsume all Command pattern use cases. |
| Q3 | Existing primitives? | Yes — `() -> void` = Command, `() -> V` = Callable. All via `function` + `call` + `scope` (closure capture). |
| Q4 | Violates principle? | No. Not adding a new concept. |
| Q5 | New semantics? | **No new semantics.** Documentation-only — pattern disappears into existing delegates. |
| Q6 | Composition? | Yes — of existing `delegate`, `function`, `scope`. |
| Q7 | Sugar over primitives? | Yes — all Command patterns already expressed via existing primitives. |
| Q8 | Optimisation? | N/A — no new feature. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Not as a new feature.** Existing delegate model covers it. |

**Classification per D-03:** Existing concept. Not a new feature — delegate model subsumes Command pattern.

### Gate Validation

**Gates applied:** All 7 (documentation-only confirmation; gates evaluate existing-concept coverage rather than new-feature design)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Developer writes a lambda, not a class — direct ergonomic gain. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Delegate model subsumes all command arities. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Fewer concepts — one primitive (delegate) replaces multiple patterns. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | No new architecture — the concept documents existing capabilities. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Documentation-only — no implementation dependency. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | No surface area — no maintenance burden. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs naturally generate lambdas/closures — no need to teach a Command abstraction. |

**Overall:** Pass (existing-concept confirmation). All 7 gates clear. The delegate model already subsumes all GoF Command pattern use cases — no new feature needed.

---

## Entry: CONTEXT_LIMITED_MODULES (EDR-072)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/CONTEXT_LIMITED_MODULES.md`](../../what/concepts/CONTEXT_LIMITED_MODULES.md)
**Decision recorded as:** [EDR-072](../decision_records/architecture/EDR-072-context-limited-modules.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | LLM attention window limit — understanding a module requires loading its entire transitive closure. Bounded context for human readers. |
| Q2 | Language/StdLib/Policy? | **Language.** Module system requires compiler support for scoping, visibility, dependency declaration, effect isolation. |
| Q3 | Existing primitives? | No. Module scoping, visibility control, effect verification require compiler-level semantics. |
| Q4 | Violates principle? | No. Explicitness, Minimal Core (one module system). |
| Q5 | New semantics? | **New semantics.** Module-level scoping, effect isolation at module boundary, explicit dependency declaration, context budget diagnostic. |
| Q6 | Composition? | No — module system is fundamental language organisation construct. |
| Q7 | Sugar over primitives? | No — see Q6. |
| Q8 | Optimisation? | No. Module organisation determines what code is visible — semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Essential for LLM-native design — bounded context window. |

**Classification per D-03:** Language. Module system with explicit public API, declared dependencies, and effect isolation.

### Gate Validation

**Gates applied:** All 7 (a new Language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Understand a module from its header — direct LLM and human benefit. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Explicit dependency graph — no hidden access. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One module header format covers API, dependencies, and effects. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Module system is the foundation of compilation units — architectural centrality justified. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Module semantics independent of compilation model. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Explicit module boundaries improve long-term maintainability. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLM can see a module's entire surface in its header — critical for context window management. |

**Overall:** Pass. All 7 gates clear. Context-limited modules are essential for LLM-native design with bounded attention windows.

---

## Entry: DECLARATIVE_CONSTRUCTS (EDR-073)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/DECLARATIVE_CONSTRUCTS.md`](../../what/concepts/DECLARATIVE_CONSTRUCTS.md)
**Decision recorded as:** [EDR-073](../decision_records/architecture/EDR-073-declarative-constructs.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Declarative constructs reduce LLM generation error rates — specifying intent (what) is easier than implementation steps (how). |
| Q2 | Language/StdLib/Policy? | **StdLib.** All declarative constructs are method implementations on collection/resource types. |
| Q3 | Existing primitives? | Yes — each has documented desugaring to imperative primitives (collection ops = Iterator protocol, resource mgmt = scope). |
| Q4 | Violates principle? | No. Intent Over Implementation, Minimal Core. |
| Q5 | New semantics? | **No new semantics.** All constructs have defined desugaring paths. |
| Q6 | Composition? | Yes — of existing primitives. |
| Q7 | Sugar over primitives? | Yes — all declarative constructs are sugar over imperative primitives. |
| Q8 | Optimisation? | No — declarative constructs express programmer intent. Loop fusion is optimisation. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** Improves LLM generation accuracy and human readability. Query expressions deferred to v0.2. |

**Classification per D-03:** StdLib. Declarative constructs for common transformations. All have documented desugaring. Query expressions deferred.

### Gate Validation

**Gates applied:** All 7 (StdLib features still require validation per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Documentation of best practices — direct programmer value. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | All constructs are existing concepts — no inconsistency risk. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One document cataloguing existing patterns. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | No new architecture — pure documentation. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Documentation-only — no implementation dependency. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Must be kept in sync with evolving concepts — lightweight maintenance. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Documentation tells LLMs the preferred declarative form for each task. |

**Overall:** Pass. All 7 gates clear. Declarative constructs document preferred idiomatic patterns with documented desugaring paths.

---

## Entry: DECLARATION_BY_ASSIGNMENT (EDR-074)

**Date:** 2026-07-27
**Artifact validated:** [`what/concepts/DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)
**Decision recorded as:** [EDR-074](../decision_records/architecture/EDR-074-declaration-by-assignment.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Variable declaration ceremony — explicit `let`/`var`/`Type name` before first use adds noise. But no accidental creation via typos. |
| Q2 | Language/StdLib/Policy? | **Language.** Definite assignment analysis, read-before-write detection, `let` for shadowing all require compiler support. |
| Q3 | Existing primitives? | No. Definite assignment analysis — verifying variable is assigned on all paths before read — is compiler-level flow analysis. |
| Q4 | Violates principle? | No. Explicitness (`let` for shadowing, `mut` for mutation), Concision (first assignment creates). |
| Q5 | New semantics? | **New semantics.** Definite assignment analysis (read-before-write is compile-time error), `let` shadowing semantics, no implicit globals. |
| Q6 | Composition? | No — flow analysis across code paths is compiler-level. |
| Q7 | Sugar over primitives? | Partial — `let` for shadowing is syntactic. Definite assignment requires compiler support. |
| Q8 | Optimisation? | No. Variable existence and initialization tracking is semantic. |
| Q9 | Backward compat? | N/A — pre-v1.0. Concrete syntax deferred to Phase 5. |
| Q10 | Worth adding? | **Yes.** Concise + safe. Deferred to Phase 5 for concrete syntax finalization. |

**Classification per D-03:** Language. Definite assignment analysis, read-before-write detection, explicit `let` for shadowing. Borderline with Phase 5 Syntax.

### Gate Validation

**Gates applied:** All 7 (a new Language construct requires the full catalogue, per `DECISION_VALIDATION.md` § Gate Selection)

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Write `x = 1` and it works — minimum ceremony. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Definite assignment analysis is well-defined — no ambiguity. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One rule — first assignment declares — covers all cases. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with type inference (EDR-027) and immutability-by-default. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Declaration semantics independent of concrete syntax. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Proven from Python and Julia — well-understood semantics. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs already generate `x = 1` patterns — no new syntax to learn. |

**Overall:** Pass. All 7 gates clear. Declaration by assignment provides minimal ceremony with compiler-enforced safety. Concrete syntax deferred to Phase 5.

---

## Entry: CONSTRAINED_TYPES (EDR-080)

**Date:** 2026-07-30
**Artifact validated:** [`what/concepts/CONSTRAINED_TYPES.md`](../../what/concepts/CONSTRAINED_TYPES.md)
**Decision recorded as:** [EDR-080](../decision_records/architecture/EDR-080-constrained-types.md)
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Type-level constraints vs. function-level contracts. No ergonomic way to say "this type only holds values from a subset" at declaration level. |
| Q2 | Language/StdLib/Policy? | **Language Pattern (Level 2).** Nominal type with type-level constraint predicate, decomposed via `struct` + `Callable` trait. Runtime validation — not compile-time proof. |
| Q3 | Existing primitives? | Yes — full decomposition to `struct` + `pack` + `Callable` impl. No new primitives needed. |
| Q4 | Violates principle? | No. Composition over addition. Minimal Core satisfied. |
| Q5 | New semantics? | **No new semantics.** Pure syntactic sugar. Constraint follows Contract Enforcement Policy. |
| Q6 | Composition? | Yes — `struct` + contract compose to produce constrained types. |
| Q7 | Sugar over primitives? | **Yes** — `type Age = Int requires v >= 0 && v <= 150` desugars to `struct` + `Callable` impl with boundary check. |
| Q8 | Optimisation? | No. Constraint checking follows contract enforcement rules — same mechanism. |
| Q9 | Backward compat? | N/A — pre-v1.0. |
| Q10 | Worth adding? | **Yes.** LLM generability benefit (schema-visible constraints) + boilerplate elimination for domain primitives. |

**Classification per D-03:** Language Pattern (Level 2). Full decomposition to struct + contract. No compiler-level changes required beyond desugaring.

### Gate Validation

**Gates applied:** All 7 per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

#### 1. `USER_VALUE_GATE` — [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer modelling domain primitives, I want to say `type Email = String requires matches(v, pattern)` instead of writing a wrapper struct with manual validation. As an LLM generating code, I want to see `Email{base: String, constraint: matches}` in the schema so I know what values are valid.

**Press release.** *Orthon introduces constrained types — a one-line declaration that turns `Int` into `Age`, `String` into `Email`, and any primitive into a self-validating domain type. The constraint is declared once on the type and enforced at every boundary where a raw value enters. Consuming functions carry no duplicate `requires`. LLMs see the constraint in the schema and generate correct values.*

**FAQ.**
- *How is this different from writing a struct?* — The constraint is declared once on the type, not duplicated on every consuming function.
- *How is it constructed?* — Via the `Callable` trait — `Age(42)` works like any other call. Not via `new`/`make`.
- *Is this checked at compile time?* — Literals are. Runtime values are checked at the type boundary, following the same rules as contracts.
- *What about mutation?* — The backing field is immutable. Once constructed, the value is always valid.

**Verdict: Pass.** The problem is concrete (boilerplate + LLM schema gap), the solution is minimal (syntax sugar), and the benefit is clear.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md)

**Define all terms.**
- **Constrained type:** a nominal type whose singleton field wraps a base type with a predicate
- **Constraint expression:** a pure expression, checked at construction, following contract expression rules
- **Backing field:** the single immutable field holding the constrained value

**Test with counterexamples.**
- *What happens when you mutate the backing field?* — You cannot. The field is immutable (struct with immutable field). Constraint cannot be bypassed.
- *What about `Age` and `Score` both wrapping `Int`?* — They are distinct nominal types. No subtyping. `fn(x: Age)` does not accept `Score`.
- *What if the constraint is a complex expression?* — Same rules as contract expressions. Purity is enforced by the compiler.

**Follow the contradiction.** Apparent tension: "constraint lives at the type level" vs. "checked at runtime, not compile time." Resolved: the type *declares* the constraint; enforcement follows Contract Enforcement Policy. The constraint is part of the type *documentation* (visible in Schema) and *runtime behaviour*, not the type *proof system*.

**Verdict: Pass.** No internal contradictions. One Flag: mutation guard must be explicitly specified in the concept draft.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** Constrained types are fully expressible as composition of existing primitives (struct + contract).

**Observations.**
```
type Age = Int(0..150)
    → struct Age { value: Int }
    → new(v) requires v >= 0 && v <= 150
    → ensures result.value == v
```

**Prediction.** Any constrained type can be mechanically desugared to a struct with a single immutable field and a contract-bound constructor — no special compiler logic.

**Test.** Try compound constraints: `Int(v > 0 && v % 2 == 0)` → desugars to `requires v > 0 && v % 2 == 0`. Try pattern: `String(matches: p)` → desugars to `requires matches(v, p)`. All pass.

**Alternative hypothesis (falsified).** Constrained types require a new primitive (type-level predicate). Falsified: the predicate lives in the contract expression, not in the type checker.

**Verdict: Pass.** Complete decomposition to Level 0–1 primitives. Zero new semantics.

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.**
1. Constrained types desugar to Level 2 (Language Pattern) constructs
2. The constraint uses the existing contract expression language
3. The backing field follows struct semantics

**Deductions.**
- No change to Data Model (Level 0) — `Int(0..150)` is not a new representation
- No change to Primitive Operations (Level 1) — no new atomic operation
- No change to Implementation Strategies — constraint checking follows existing Contract Enforcement Policy
- Composes freely with generics: `Option<Age>`, `[Email]`, `fn(x: Age)` all work
- No privileged position: any primitive type can be the base

**Verdict: Pass.** Fits cleanly in Level 2. No layer violations.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](../../gates/methods/TRIZ_METHOD.md)

**Apparent contradiction.** Constraint checking must happen somewhere (seems to require runtime support), but must be strategy-independent (any strategy must support it).

**Apply separation.** The *enforcement mechanism* is strategy-dependent (elision in release is a policy choice). The *semantic rule* — "value is validated at construction" — is strategy-independent. All strategies can implement construction validation; they differ only in whether they check it.

**Verdict: Pass.** Constraint semantics are strategy-agnostic. Enforcement follows Contract Enforcement Policy which already handles strategy separation.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md)

**One sentence:** *A constrained type wraps a primitive with a validation rule — like a struct with a contract, but you write it in one line.*

**Evolution path:**
- Future SMT integration would strengthen enforcement without changing syntax
- Constraint forms can be extended without breaking existing types
- Deprecation path: keep the struct + contract form, desugar manually

**Conceptual debt:** None. The entire semantic weight is carried by already-accepted Level 0–1 primitives.

**Verdict: Pass.** Reversible, extensible, zero debt.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `type Name = Base(constraint)` is a simple, single-syntax form.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | `{base: Int, constraint: {range: [0, 150]}}` — fully expressible |
| Predictable generation (≥90%) | Pass | Pattern matches Python type alias + Pydantic — LLMs know it |
| No hallucination surface | Pass | Single form per constraint type; no ambiguity |
| Strategy-aware default | Pass | Follows Contract Enforcement Policy — implicit, not annotation-required |
| Self-correctable | Pass | Literal out of range → compile error; type mismatch → type error |

**Verdict: Pass.**

---

### Overall

All seven gates Pass outright. One Flag (mutation guard) resolved in concept draft.

**Gates not applied:** None.

---

### Post-Acceptance Amendment (Type B — 2026-07-30)

**Trigger:** Design review uncovered three refinements to the accepted model.

**Gap type:** Type B (unresolved detail) per `CONCEPT_PIPELINE.md` § 10.

**Changes applied (in-place edit of EDR-080, CONSTRAINED_TYPES.md, CORE_CONCEPTS.md, GLOSSARY.md):**

| Before | After | Rationale |
|--------|-------|-----------|
| `type Age = Int(0..150)` | `type Age = Int requires v >= 0 && v <= 150` | `()` is call syntax (Semantic Purity). `Int(0..150)` reads as call, not type constraint. `requires` reuses existing contract keyword — consistent with function-level contracts. |
| Desugar to `struct` + `new` + `requires`/`ensures` on constructor | Desugar to `struct` + `Callable(Int) -> Age` impl + boundary check | `new` is reserved for transforming constructors. `Callable` trait is consistent with uniform call syntax: `Age(42)` is a call, not a factory. Constraint lives on the type, not on a constructor function. |
| Constraint duplicated on constructor and consuming functions | Constraint **only on the type** — checked at every `Int → Age` boundary | Eliminates redundancy. `fn greet(age: Age)` needs no `requires` — `Age` already guarantees validity. Simpler compiler: one check point (boundary), not N (constructor + every consumer). |

**Gate impact assessment:** All 7 validation gates were re-evaluated against the amended model. No verdict changed:

| Gate | Original | Amended | Why unchanged |
|------|----------|---------|---------------|
| `USER_VALUE_GATE` | Pass | Pass | Problem (domain primitives with schema-visible constraints) is identical. Solution is cleaner (no duplication). |
| `LOGICAL_CONSISTENCY_GATE` | Pass | Pass | `Callable` trait + boundary enforcement removes the `new`/`make` ambiguity. No new contradictions. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Pass | Pass | Same decomposition to existing primitives. `Callable` replaces `new` — fewer keywords, not more. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Pass | Pass | `Callable` is already an accepted trait. No layer violations. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Pass | Pass | Boundary checks follow Contract Enforcement Policy — same mechanism, same strategy independence. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Pass | Pass | Model is strictly simpler. Less conceptual debt, same evolution path. |
| `LLM_GENERABILITY_GATE` | Pass | Pass | `Int requires v >= 0` is less ambiguous for LLMs than `Int(0..150)` (no confusion with call syntax). |

**Procedure:** Per `CONCEPT_PIPELINE.md` § 10 (Type B), the decision pipeline, concept design review, and validation gates were NOT re-run. Only the affected gates were re-assessed against the delta. Rationale: the core decision (Level 2 Language Pattern, runtime enforcement, nominal identity) did not change — only the syntactic form and decomposition mechanism were refined.

---

## Entry: 1-Based Indexing (INDEXING_ONE_BASED)

**Date:** 2026-08-05
**Artifact validated:** [`how/concepts/research/important/INDEXING_ONE_BASED.md`](../../how/concepts/research/important/INDEXING_ONE_BASED.md)
**Decision recorded as:** Not yet — pipeline run completed, **NOT CONVERGED**. EDR (EDR-082) pending resolution of blockers B1–B4.
**Pipeline applied:** Full 10-question Decision Pipeline per `DECISION_PIPELINE.md`, then Concept Design Review, then all 7 Decision Validation gates.

### Pipeline Q&A

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem? | Cognitive gap between human counting (1, 2, 3, …) and machine addressing (offset 0, 1, …) → off-by-one errors and domain-expert translation tax in index-based code. |
| Q2 | Language/StdLib/Policy? | **Language (Core).** The index base is a semantic commitment on the collection access model — a library cannot change the meaning of `a[i]`. |
| Q3 | Existing primitives? | Partially. The *base* is a semantic parameter, not expressible by composing primitives. However, the *mechanism* `a[i]` is not covered by the existing `attribute access` primitive (named-member, `.` syntax) — see Decomposition Check flag. |
| Q4 | Violates principle? | No. Aligns with Data First, Intent Over Implementation, POLA, Minimal Core (rejects configurable base). |
| Q5 | New semantics? | **Yes.** A semantic commitment on how indexed access maps to elements — not syntactic sugar. |
| Q6 | Composition? | No. The base is a semantic default, not a composed behaviour. |
| Q7 | Sugar over primitives? | The *mechanism* `a[i]` may be sugar for `nth(i)`/`get(i)` (Level 2 pattern) — the open decomposition question. The *base* is not sugar. |
| Q8 | Optimisation? | No. Index translation is compile-time constant folding. |
| Q9 | Backward compat? | Pre-v1.0 — no released-code break. But broad spec impact: 12 documents plus accepted concepts (ITERATION_LOOP EDR-053, ITERATOR_PROTOCOL EDR-022) assume 0-based. |
| Q10 | Worth adding? | **Yes.** The decision is unavoidable — even abstract indexing (Alternative C) must choose a base for `nth(i)`. The question is which base, not whether. |

**Classification per D-03:** Core Language (Level 1/2 boundary — see Decomposition Check). **Language** category per `LIBRARY_BOUNDARY.md`.

### Primitive Decomposition Check

**Finding (Flag):** The concept doc claims `items[i]` "is a form of attribute access". This is **inaccurate** against `PRIMITIVE_BLOCKS.md` § 3.2.4: `attribute access` is defined as *named* member access via `.` syntax. The primitive set has **no positional/subscript access**. Resolution options:
- **(a)** new Level 1 primitive (positional access), or
- **(b)** Level 2 pattern: `a[i]` desugars to `a@get(i)` (`function` + `call`), with the index base as a semantic parameter of the `@get` protocol method.

Option (b) is consistent with Minimal Core and matches the existing Metadata Protocol pattern (`obj@fields`, `list@len()`, `collection@iter()`). The composition formula must be shown. **Must be corrected before EDR.**

**Resolution (2026-08-05) — RESOLVED, option (b) adopted:**
- `a[i]` ≡ `a@get(i)` — Level 2 sugar over the `@get` protocol method (`@`-prefix per Metadata Protocol), decomposing to `function` + `call`. Not `.nth(i)`: `.` is reserved for user-defined member access, and indexing must remain language-recognized.
- Composition formula: `a[i]` → `a@get(i)` → `call(function(@get), a, i)`.
- The 1-based base is a semantic parameter of the `@get` contract: first element at `@get(1)`, last at `@get(len(a))`.
- Positional access is the one-element form of `unpack` and therefore applies to every random-access `pack` composite (tuples, strings, Span, ranges), not just collections. Capability is declared via an `Indexable`-like trait, so `Sequence`/`Set` (non-random-access) reject `a[i]` at compile time.
- Bounds behaviour (Option vs Result) deferred to the `@get` contract, aligned with EDR-018 / EDR-020.

### Gate Validation

**Gates applied:** All 7 per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

#### 1. `USER_VALUE_GATE` — [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md)

**User story.** As an Orthon programmer — and every code-generating LLM — I want `items[1]` to return the first element, so that index-based code reads the way I count and the way my domain notation writes it, and so that the off-by-one class (`range(len(items))` + `items[i+1]`) disappears at the language level.

**Press release.** *Orthon counts like people do. The first element is at index 1, the last at index `len`. Mathematical formulas and business notation copy into code without index translation. Off-by-one bugs lose their primary breeding ground.*

**FAQ.**
- *How is this different from C-family?* — C-family inherits the pointer-arithmetic base 0; Orthon's Data model has no raw memory, so the hardware default has no claim here.
- *When would I use 0-based?* — Only at the FFI boundary, via an explicit translation layer.
- *What do I lose?* — C-ecosystem familiarity and zero-cost FFI indexing; both accepted as documented trade-offs.

**Requirements derived.** A Core semantic commitment (index base = 1); a range-literal convention; an FFI index-translation boundary; an `enumerate` start rule.

**Verdict: Pass.** Problem is user-stated (cognitive gap, off-by-one), justified by the comfort-by-construction Vision pillar, with a concrete code example.

---

#### 2. `LOGICAL_CONSISTENCY_GATE` — [Socratic Method](methods/SOCRATIC_METHOD.md)

**Define all terms.** "1-based indexing"; "last index == len"; "inclusive-inclusive range `1..N`"; "half-open FFI form `0..<N`"; "`enumerate` from 1".

**Test with counterexamples.**
- *Two range forms (`1..N` inclusive + `0..<N` half-open) — a special case that patches an inconsistency?* — The half-open form is justified by interop, but it *is* a second range semantic. It must be scoped to the FFI boundary only, or the language carries two range conventions.
- *Does `enumerate` starting at 0 while collections are 1-based create an index/value mismatch?* — Yes if undecided. The doc proposes enumerate from 1 (consistent), but Open Question 2 leaves it open. Must be resolved.
- *What about the accepted ITERATION_LOOP (`for i in 0..len(array)`)?* — Currently 0-based. Adopting 1-based requires deciding the canonical index-range form (`1..=N` inclusive vs `0..<N` half-open). Cross-concept interaction not ignored but unresolved.
- *What about SPAN?* — If Span (EDR-064, primarily an FFI/interop view) uses 0-based while collections use 1-based, the language has two bases — undermining the single-natural-counting story. Must be resolved (single-base rule vs. explicit two-base).

**Follow the contradiction.** Apparent tension: "one natural counting convention" vs. "half-open form exists for FFI". Resolved only if the half-open form is a *boundary-only* escape hatch, never the default.

**Verdict: Flag.** No paradox, but three undecided cross-concept interactions (range convention, `enumerate` default, SPAN base) plus a boundary special-case must be resolved before the concept is internally closed.

---

#### 3. `CONCEPTUAL_SIMPLICITY_GATE` — [Scientific Method](methods/SCIENTIFIC_METHOD.md)

**Hypothesis.** 1-based indexing is a single, learnable semantic commitment — "Orthon counts like you do" — with no new keywords and no configurable base.

**Observations.**
- One decision (base = 1); no new primitive under option (b); range/`enumerate`/FFI are separate concepts.
- Alternative B (configurable base) is correctly rejected: violates Minimal Core and Orthogonality — two collections with different bases cannot be indexed uniformly.
- Alternative C (abstract indexing) is correctly rejected: `nth(i)` still must pick a base.

**Prediction.** A learner needs one sentence to absorb it.

**Alternative hypothesis (weakened).** Indexing is a new Level 1 primitive (option a). Option (b) — Level 2 pattern over `nth(i)` — is simpler and consistent with the existing protocol pattern. Scope note: the concept doc bundles range semantics + `enumerate` + FFI translation into one proposal; those belong to RANGE, ITERATOR_PROTOCOL, and FFI respectively.

**Verdict: Pass** (with scope note — keep the minimal concept to the index base; move adjacent decisions to their owning concepts).

---

#### 4. `ARCHITECTURAL_INTEGRITY_GATE` — [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md)

**State the premises.**
1. The index base is a Core Language semantic commitment.
2. `a[i]` decomposes to a Level 2 pattern over `nth(i)` (option b) or a Level 1 primitive (option a).
3. Range, `enumerate`, and FFI translation are downstream of the base.

**Deductions.**
- Fits within the layered architecture; no StdLib/compiler-internals coupling.
- Composes orthogonally with existing constructs (with the three interactions named in the Consistency gate).
- **Retroactive modification:** ITERATION_LOOP (EDR-053) and ITERATOR_PROTOCOL (EDR-022) were accepted with 0-based ranges/enumerate. Adopting 1-based *modifies accepted concepts* — engaging the gate's fail condition "an existing concept must be modified to accommodate the new one". Because the base is more foundational than those concepts, the dependency direction is legitimate, but it must be handled as a documented cross-concept amendment, not silent drift.

**Verdict: Flag.** Architecture fit is clean, but the retroactive amendment of two accepted EDRs must be explicit and recorded.

---

#### 5. `IMPLEMENTATION_INDEPENDENCE_GATE` — [TRIZ](methods/TRIZ_METHOD.md)

**Apparent contradiction.** Indexing seems tied to a specific layout (0-based offset arithmetic), yet must be strategy-independent.

**Apply separation.** The *semantic rule* ("the first element is at index 1") is strategy-independent — all strategies (Default, Embedded, High-Performance) implement the same mapping. Index translation to C/0-based is a compile-time constant fold and a boundary concern, not a strategy concern. No strategy produces different observable behaviour.

**Verdict: Pass.** 1-based indexing is fully strategy-agnostic.

---

#### 6. `LONG_TERM_MAINTAINABILITY_GATE` — [Einstein's Method](methods/EINSTEIN_METHOD.md)

**One sentence:** *Orthon counts from 1, so `items[1]` is the first element and `items[len(items)]` is the last — and the cost of this is an explicit translation layer at every C interop boundary.*

**Evolution path.**
- Pre-1.0: the decision is cheap to make now (a design-time commitment) and effectively **irreversible after freeze** — 50+ derived concepts, GLOSSARY, and doc examples already assume 0-based and would need a mass retrofit to change later.
- FFI translation tax is permanent and paid by every FFI consumer; mitigated by a structured, auditable boundary.
- Reversal risk: shipping 1-based while the surrounding ecosystem is 0-based is a permanent ergonomic tax at interop; the choice must be made with confidence now.

**Verdict: Flag.** The decision ages well *if* it is right; it does not age gracefully *if wrong* — the reversal window is now, before freeze.

---

#### 7. `LLM_GENERABILITY_GATE` — [Empirical Analysis](methods/EMPIRICAL_ANALYSIS_METHOD.md)

**Structural analysis:** `a[i]` with 1-based indexing is single-syntax and unambiguous.

| Criterion | Verdict | Basis |
|---|---|---|
| Schema-serializable | Pass | The base is a schema-visible convention (grammar + stdlib `nth` contract) |
| Predictable generation (≥90%) | Pass | LLMs are trained on both bases (Lua, Julia, MATLAB, R); 1-based has fewer off-by-one patterns to track |
| No hallucination surface | Pass | One base, no configurable option, no ambiguity |
| Strategy-aware default | Pass | Base is strategy-independent; generated code is valid under all strategies |
| Self-correctable | Pass | Out-of-range index → Static Analyser diagnostic; boundary translation verifiable |

**Note (Flag):** LLMs are *predominantly* trained on 0-based code; the 1-based convention must be surfaced explicitly in the LLM schema/strategy (e.g., in the `nth` contract) so generation defaults to 1-based, not 0-based.

**Verdict: Pass** (with the schema-surface note).

---

### Convergence Check (pre-EDR gate)

| Check | Status |
|-------|--------|
| Syntax reviewed | ⚠️ Range syntax deferred to Phase 5; `a[i]` has no conflict with `.`/`@` |
| Edge cases probed | ✅ SPAN interaction resolved (B2) — single-base 1-based |
| Desugaring verified | ✅ Resolved (B1) — `a[i]` ≡ `a@get(i)` |
| User/stakeholder agrees | ✅ All open questions resolved (B1–B4) |
| No remaining ambiguity | ✅ No blockers remain |

**Convergence re-check (2026-08-05): PASS** — B1 (decomposition), B2 (SPAN), B3 (enumerate), and B4 (range convention) all resolved. Ready to file EDR-082.

---

### B4 Resolution (2026-08-05)

**Range norm locked — inclusive-inclusive `1..N` everywhere, incl. slices:**
- `1..N` is the **only** range semantic: index access, slices (`items[1..k]` = first k elements), and iteration all use it.
- The language owns the `+1` length arithmetic: `len(slice)` returns the element count directly; the programmer never writes `j - i + 1` (Intent Over Implementation).
- Empty slice is a value with `end < start` (e.g., `items[1..0]`), not a syntax error; exact representation is a RANGE (Type A) question.
- `0..<N` is an FFI-boundary interop utility only; visibility per FFI translation policy (automatic vs explicit — Open Q4, FFI concept, M8); never the default, never in application code.
- Cross-concept amendment: ITERATION_LOOP (EDR-053) / ITERATOR_PROTOCOL (EDR-022) canonical index iteration becomes `for i in 1..=len(array)`; recorded as Type C conflict C-001 in `CONFLICT_REGISTRY.md`; applied at EDR-082 acceptance.
- **Conflict surfaced:** `RANGE_SLICE.md` (parallel hypothesis, 2026-08-05) proposes exclusive `a..b`; this contradicts the GLOSSARY norm (`1..10` inclusive). Reconcile in the RANGE concept design (Type A).

---

### B3 Resolution (2026-08-05)

**`enumerate` defaults to 1, matching the collection base:**
- `items.enumerate()` yields `(1, first), (2, second), …` — the index is pure
  ordinal numbering, matching the `1` of the inclusive range norm; it is always
  a valid `@get(i)` index on the same collection (no index/value desync).
  Failing to pin this would fail `LOGICAL_CONSISTENCY_GATE`: one term
  ("index") would carry two bases depending on whether `enumerate` produced it
  or `@get(i)` consumed it — recreating the off-by-one class the concept
  eliminates.
- **Not a keyword, not a primitive:** `enumerate(items) ≡ zip(1..=len(items), items)` —
  a pure Level 2 composition over the inclusive range norm (B4) + `zip`.
  Both `enumerate` and `zip` are plain Standard Library methods on
  `Iterator[T]` (EDR-022/EDR-032) — no compiler special-casing, no new syntax.
  Strengthens `CONCEPTUAL_SIMPLICITY` (one counting convention for index
  production and consumption) and `LLM_GENERABILITY` (single, teachable
  formula).
- **No start parameter.** An offset is expressed by an explicit preliminary
  range, e.g. `zip(offset..len(items), items)` (range spelling per Phase
  5/RANGE). `enumerate` is exactly one thing: pair each element with its
  1-based ordinal.
- **Rejected:** Python-style default 0 (recreates the off-by-one class);
  mandatory `enumerate(from: 1)` (ceremony without information, fails POLA and
  Intent Over Implementation); a `from:`/start parameter (non-orthogonal — the
  offset is a range concern, not an `enumerate` concern).
- **Cross-concept amendment:** ITERATOR_PROTOCOL (EDR-022) `.enumerate()`
  ("Pair each element with its index", `Iterator[(Int, T)]`) — base pinned to
  1; applied at EDR-082 acceptance (recorded under C-001 in
  `CONFLICT_REGISTRY.md`). `enumerate` remains a Standard Library combinator
  (LIBRARY_BOUNDARY); only its *base* is a Core commitment of the 1-based
  decision, consistent with `len`/`@get`.

---

### B2 Resolution (2026-08-05)

**Single-base rule — `Span` is 1-based like every collection:**
- `span[1]` = first element, `span[len(span)]` = last; `enumerate(span)`
  starts at 1 (no Span-specific exception). Span is a **Language** type
  (EDR-064) and a random-access `pack` composite under B1's
  `Indexable`-like trait — so the `@get(1)` contract applies to it
  identically. A second base would fail `LOGICAL_CONSISTENCY_GATE` (one
  term "index", two bases) and `CONCEPTUAL_SIMPLICITY_GATE` (the LLM must
  remember which type uses which base).
- **FFI role does not create a second base.** Span's interop usefulness
  (wrapping raw C buffers) is served by the existing FFI index-translation
  layer (B4: `0..<N`), which already handles memory layout, calling
  convention, and type mapping at the boundary. Raw C memory enters through
  that translation path — never through 0-based Span indexing in application
  code. The exact C-facing constructor surface (`from_c`-style constructors,
  `as_c_view`-style adapters; 0-based C-native vs 1-based arguments) is a
  third-party library support question — **deferred**: requires its own
  hypothesis, belongs to the FFI concept (M8), not pinned here.
- **Rejected:** two-base language (Span 0-based, collections 1-based) —
  breaks B1's trait contract, B3's enumerate, and the single-natural-counting
  story; hybrid per-type base marker — violates Minimal Core (configurable
  bases rejected in the concept's Alternative B).
- **Cross-concept amendment:** SPAN (`what/concepts/SPAN.md`, EDR-064) —
  base committed to 1 and examples rewritten (`span[0]` → `span[1]`;
  `arr[1..3]` re-read under the inclusive norm as "first 3 elements");
  applied at EDR-082 acceptance. Recorded under C-001 in
  `CONFLICT_REGISTRY.md`.

---

### B5 Resolution (advisory, 2026-08-05)

Advisory items made concrete (none blocks EDR-082):
- **B5-1 — Collection Indexing Policy added** to
  `IMPLEMENTATION_POLICIES.md` (Status: *Pending concept acceptance
  (EDR-082)*; single value `OneBased`; no configurable base in v0.1).
  FFI Boundary Policy and Range Semantics Policy are deferred to their
  concepts (FFI M8, RANGE Type A).
- **B5-2 — LLM Toolchain requirement recorded.** The Standard Library
  Schema (Schema Provider, `LLM_NATIVE_TOOLCHAIN.md`, deferrable) must
  encode the 1-based index base (`@get` contract, range literals) so LLM
  generation defaults to 1-based, never 0-based. Honoured when the LLM
  Toolchain concept is developed.
- **B5-3 — GLOSSARY / examples audit planned.** Concrete 0-based examples
  to fix (applied at EDR-082 as part of C-001):
  - `what/GLOSSARY.md` § For Loop: `for i in 0..len(array)` →
    `for i in 1..=len(array)`.
  - `what/concepts/SPAN.md`: `span[0]`, `data[0] = 42`, `arr[1..3]`
    (re-read under the inclusive norm).
  - `ITERATION_LOOP.md` (EDR-053) and `ITERATOR_PROTOCOL.md` (EDR-022)
    examples.
  - Any other `0..`/`[0]` usage surfaced during the audit.

---

### Overall

**Verdict: CONVERGED (2026-08-05).** Pre-filter (Pipeline Q&A) → **ACCEPT as a Language decision**. Validation gates: 3 Pass, 4 Flag, 0 Fail. All four blockers (B1–B4) resolved; convergence re-check PASS. Ready to draft EDR-082 (acceptance), which will also apply the cross-concept amendments (C-001).

**Blockers before EDR-082:**
- **B1 (decomposition):** ✅ **RESOLVED (2026-08-05)** — `a[i]` is a Level 2 pattern over `a@get(i)` (Metadata Protocol, `@`-prefix); no new primitive; `INDEXING_ONE_BASED.md` § Impact on Primitive Blocks corrected. See the resolution note under Primitive Decomposition Check.
- **B2 (SPAN):** ✅ **RESOLVED (2026-08-05)** — single-base rule: Span is 1-based like every collection; FFI raw buffers translate at the boundary (`0..<N`), never a second base. Cross-concept amendment to SPAN/EDR-064 pinned at EDR-082. See the B2 Resolution note above.
- **B3 (enumerate):** ✅ **RESOLVED (2026-08-05)** — `enumerate` defaults to 1 (matching the collection base); composition `enumerate(items) ≡ zip(1..=len(items), items)`; `enumerate`/`zip` are StdLib methods on `Iterator[T]` (EDR-022/EDR-032), not keywords; no start parameter — offsets use an explicit preliminary range; Python-style default 0 rejected (index/`@get` desync). Cross-concept amendment to ITERATOR_PROTOCOL EDR-022 pinned at EDR-082. See the B3 Resolution note below.
- **B4 (retroactive amendment):** ✅ **RESOLVED (2026-08-05)** — range norm locked: inclusive-inclusive `1..N` everywhere (incl. slices); language owns `+1` (`len(slice)`); empty slice = `end < start`; `0..<N` is FFI-boundary-only. Cross-concept amendment to ITERATION_LOOP EDR-053 / ITERATOR_PROTOCOL EDR-022 recorded as Type C (C-001) in `CONFLICT_REGISTRY.md`, applied at EDR-082. See the B4 Resolution note above.

**Advisory (not blocking) — resolved 2026-08-05:**
- **B5-1:** Collection Indexing Policy added to `IMPLEMENTATION_POLICIES.md` (pending acceptance). FFI Boundary / Range Semantics Policies deferred to their concepts (FFI M8, RANGE Type A).
- **B5-2:** LLM Toolchain requirement recorded (schema encodes 1-based base) — honoured when the LLM Toolchain concept is developed.
- **B5-3:** GLOSSARY/examples audit planned — fixes applied at EDR-082 as part of C-001.
