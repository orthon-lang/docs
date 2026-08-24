# EDR-090: Data-Oriented Programming — Data Model Lineage and Two Core Philosophy Principles

**Status:** Accepted

**Date:** 2026-08-24

**Category:** Architecture

**Scope:** Project (Core Language — Data Model & Design Principles)

---

### Context

Orthon's data model — `Data First` ([`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)),
the Data / Data Modifier dichotomy, the seven Representations, immutability
by default, and gradual typing — is, in substance, an instance of the
**Data-Oriented Programming (DOP)** paradigm codified by Yehonathan Sharvit
(*Data-Oriented Programming*, Manning, 2022). DOP rests on four principles:

1. **Separate code from data** — behaviour lives in functions; data is a
   plain value with no behaviour.
2. **Represent data with generic data structures** — data is expressed in a
   small set of generic structures, not bespoke object types.
3. **Data is immutable** — data is never mutated; transformation produces a
   new version.
4. **Separate data schema from data representation** — the schema (types,
   validation) is optional and applied at boundaries, not embedded in the
   representation.

Orthon already realizes principles 1–3 explicitly and principle 4
implicitly, but **the paradigm is never named anywhere in the repository.**
A repository-wide search finds `expression-oriented`, `object-oriented`, and
`LLM-oriented`, but no reference to DOP or Sharvit. This is a Transparency
gap ([`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) § Transparency):
the provenance of the project's central idea is invisible, so a reviewer
cannot distinguish an original coinage from an instance of a known paradigm,
and cannot evaluate Orthon against DOP's documented trade-offs.

Two of the four principles lack a constitutional statement in
[`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md):

- **Data is immutable by default** is decided and enforced — Tier 1 example
  in [`DECISION_PROCESS.md`](../../process/DECISION_PROCESS.md),
  [`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) § Mutation,
  EDR-041 (immutable collection literals), EDR-061 (copy-on-write),
  EDR-068 (immutable date/time), EDR-069 (persistent data structures) — but
  it is **absent from the Design Principles list itself**.
- **Schema separate from representation** is nowhere stated. Gradual typing
  ([EDR-059](EDR-059-gradual-typing.md)) and constrained types
  ([EDR-080](EDR-080-constrained-types.md)) imply it; *transparent
  representation* ([`DATA_MODEL.md`](../../concepts/research/essential/DATA_MODEL.md)
  principle 5) appears to lean the other way. The tension must be resolved
  explicitly, not by silence.

Because [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) is locked
("Closed for modification"), any change requires a **Tier 1 EDR**
(Architecture category) per [`DECISION_PROCESS.md`](../../process/DECISION_PROCESS.md)
Guiding Rule 2, with Human Sign-off ([`AGENTS.md`](../../../AGENTS.md) §7.4).
The Decision Pipeline classifies this as a **Language** decision (D-03): it
declares what the Core Language means by "data" and by "schema".

**Research & pipeline trail:** user-originated analysis (2026-08-24)
mapping the four DOP principles onto existing Orthon decisions; see
`DECISION_LOG.md` entry for this EDR.

---

### Decision

1. **Declare the lineage.** Orthon's data model is an explicit instance of
   **Data-Oriented Programming**. Record DOP as the named paradigm behind
   `Data First`, and enumerate the four principles with the locations where
   Orthon already realizes each (Context table below).

2. **Adopt "Data Is Immutable by Default" as a Core Philosophy principle.**
   Data does not mutate; transformation produces a new version. Mutation is
   an explicit, opt-in exception marked with `mut`, visible in syntax
   (Explicitness). This promotes an existing Tier 1 decision
   ([`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) § Mutation) to
   constitutional status in the Design Principles list.

3. **Adopt "Schema Separate from Representation" as a Core Philosophy
   principle**, resolving the tension by distinguishing two orthogonal axes:

   - **Representation** — the structural shape of data (Value, Tuple,
     Reference, Sequence, Set, Option, Result). Always present and always
     transparent: it is part of the type and visible in the type signature
     (transparent representation, [`DATA_MODEL.md`](../../concepts/research/essential/DATA_MODEL.md)).
   - **Schema** — the nominal or constraint label imposed on data (an ADT
     variant, a trait bound, a Constrained Type). Optional, and applied at
     the boundary (construction, assignment, parameter passing), never
     embedded in the data representation itself.

   These axes are orthogonal: transparent representation concerns the
   *structural* axis and is unaffected; schema separation concerns the
   *semantic* axis. This is already Orthon's de-facto stance — "data has no
   imposed semantic meaning until a Data Modifier transforms it"
   ([`FOUNDATIONAL_ABSTRACTIONS.md`](../../concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md)),
   optional annotations ([EDR-059](EDR-059-gradual-typing.md)), and
   boundary-only validation ([EDR-080](EDR-080-constrained-types.md) —
   "constraint lives only on the type, checked at every boundary").

**Principle mapping (recorded in the EDR and the Glossary):**

| DOP principle | Realized in Orthon |
|---|---|
| Separate code from data | `Data First` — Data vs. Data Modifier; enforced by rejected-EDR precedent (EDR-075 prototype, EDR-078 class-as-primary-composition) |
| Represent data with generic structures | The seven Representations (`DATA_MODEL.md`); "data without imposed semantic meaning" |
| Data is immutable | `SEMANTIC_MODEL.md` § Mutation; EDR-041, EDR-061, EDR-068, EDR-069 |
| Schema separate from representation | Gradual typing (EDR-059); constrained types (EDR-080); Data Modifier boundary interpretation |

---

### Consequences

- **Positive:**
  - **Transparency satisfied.** DOP provenance becomes traceable, and
    Orthon can be checked against DOP's known trade-offs instead of being
    treated as an unmotivated original coinage.
  - **Constitution matches practice.** The language's strongest data
    guarantee (immutability) is no longer missing from the Design
    Principles list.
  - **The schema/representation tension is resolved explicitly** via two
    orthogonal axes, reconciling transparent representation, gradual
    typing, and constrained types without contradiction.
  - **LLM Generability.** Naming the paradigm gives the LLM toolchain a
    known reference point; the boundary-applied schema rule is a single,
    learnable rule easier for an LLM to follow than embedded-schema
    discipline.
  - **No semantic change.** This decision names and promotes existing
    practice; it introduces no new construct, primitive, or syntax.
- **Negative:**
  - Adds two named principles to Core Philosophy (a small learning-surface
    increase for the already-dense principle list).
  - Requires amendment of the locked [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)
    (authorized by this EDR) and a Glossary update.
  - Naming an external paradigm risks being read as importing DOP's full
    agenda; this EDR scopes DOP to the four principles mapped onto existing
    Orthon decisions and explicitly rejects DOP's maps/lists-only stance.

---

### Compliance

The decision is followed when:

- [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) lists **Data Is
  Immutable by Default** and **Schema Separate from Representation** in
  Core Philosophy (tree and body), and `Data First` names DOP as its
  lineage with a link to this EDR.
- [`GLOSSARY.md`](../../../what/GLOSSARY.md) defines **Data-Oriented
  Programming** and **Schema**, cross-referenced to Data, Data Modifier,
  Representation, Gradual Typing, and Constrained Type.
- Any future document that introduces a schema or validation mechanism
  places the constraint at a boundary (per EDR-080), never embedding it in
  the representation.
- No new language construct, primitive, or syntax is introduced by this
  decision (naming + constitution only).

---

### Alternatives Considered

> Populated from Concept Design Review Step 2 (Alternatives).

| # | Alternative | Rationale for Rejection |
|---|-------------|-------------------------|
| A | **Minimal — name the paradigm and record already-true principles only** | Rejected by author's scope choice: the schema/representation relationship must be decided explicitly, not left implicit. |
| B | **Adopt DOP wholesale (generic maps/lists only, no nominal types)** | Rejected — contradicts Orthon's accepted type system (ADTs EDR-039, traits EDR-019, constrained types EDR-080). DOP is a lineage, not a straitjacket. |
| C | **Reject the schema-separation principle** | Rejected — contradicts already-accepted decisions (EDR-059 optional annotations, EDR-080 boundary validation, "data without imposed meaning"). |
| D | **Do nothing — leave lineage unnamed and immutability outside the constitution** | Rejected — violates Transparency and leaves a Tier 1 decision outside the constitutional principles. |

---

### Gate Validation

> Required for all Architecture-category EDRs.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | A programmer gets a constitution that matches practice and a traceable lineage for the central idea. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | Two orthogonal axes (representation vs. schema) reconcile transparent representation with schema separation; no contradiction. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Names existing practice; adds no new mechanism or concept to the Core. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Sits at Level 0 (Data Model) and in Core Philosophy; no layer violation. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Principles are semantic contracts; enforcement mechanism remains delegated to Strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | Reduces future drift; naming the paradigm anchors subsequent data-model decisions. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Boundary-applied schema is a single learnable rule; the named paradigm aids LLM tooling and reasoning. |

**Gates not applied:** none — all seven apply to a Core Philosophy / Data
Model change.

**Detailed reasoning:** See `DECISION_LOG.md` entry for this EDR for
per-gate reasoning trail.

---

### Related Concepts

- [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) — § Data First, § Explicitness, § Transparency
- [`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) — § Identity, § Mutation
- [`DATA_MODEL.md`](../../concepts/research/essential/DATA_MODEL.md) — seven Representations; transparent representation
- [`GLOSSARY.md`](../../../what/GLOSSARY.md) — Data, Data Modifier, Representation, Schema, Data-Oriented Programming
- [EDR-059](EDR-059-gradual-typing.md) — gradual typing (optional schema)
- [EDR-080](EDR-080-constrained-types.md) — boundary-only validation
- [EDR-041](EDR-041-collection-literal-syntax.md), [EDR-061](EDR-061-copy-on-write.md), [EDR-068](EDR-068-immutable-date-time.md), [EDR-069](EDR-069-persistent-data-structures.md) — immutability decisions
- [EDR-075](EDR-075-reject-prototype.md), [EDR-078](EDR-078-reject-class-or-structure-as-primary-composition.md) — rejected-EDR precedent for "code separate from data"

---

### Supersedes

*None* — this is a new decision, not a replacement. It promotes existing
Tier 1/2 decisions to constitutional status and amends the locked
[`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) per the Tier 1 EDR
procedure.

---

> **Human Sign-off:** `Reviewed-by: mniedre · Date: 2026-08-24 · Verdict: LOCKED`
> Required before the EDR is finalized ([`AGENTS.md`](../../../AGENTS.md) §7.4 —
> non-automatable). Only the solo author's explicit confirmation counts.
