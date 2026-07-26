# Concept Design Review

> The 5-step procedure for designing and validating each language
> concept during Milestone 2 of the design process.
>
> **Applies to:** [ROADMAP](../when/ROADMAP.md) § Milestone 2
> **Design layer:** [PHILOSOPHY.md](PHILOSOPHY.md) § Layers 3–4
> **See also:** [`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md),
> [`_language-design.md`](gates/_language-design.md),
> [`_concept.md`](templates/_concept.md)

---

## Purpose

The Concept Design Review is the core design workflow of the Orthon
project. Each concept identified in the Language Inventory (Milestone 1)
undergoes a uniform 5-step procedure that ensures the design is:

- **Problem-driven** — starts from a real programmer need, not a
  technically interesting idea.
- **Principle-compliant** — verified against the Manifesto and Design
  Principles.
- **Validated** — passes all seven Decision Validation gates before
  acceptance.
- **Traceable** — every decision is recorded with rationale,
  alternatives considered, and an Engineering Decision Record.

---

## Relationship to Other Documents

The Concept Design Review is one of three complementary artifacts that
govern language design decisions. They serve distinct roles:

| Artifact | Role | Answers | Applied |
|----------|------|---------|---------|
| **5-step procedure** (this document) | **Process** — what to do, in what order | *How do we design a concept?* | During each concept review |
| **Decision Validation gates** (`DECISION_VALIDATION.md`) | **Criteria** — what the result must satisfy | *Is the design sound?* | Evaluates the completed 5-step output |
| **Language Design Gate** (`_language-design.md`) | **Checklist** — what to verify concretely | *Did we check everything?* | Operationalises the gates into a review form |

**The flow:** The 5-step procedure produces a concept design. That design
is then evaluated through the seven Decision Validation gates. The
`_language-design.md` checklist records the outcome of each check. Only
concepts that pass all gates and satisfy the checklist are accepted.

---

## Input

- **Language Inventory** (`LANGUAGE_INVENTORY.md`) from Milestone 1 —
  the list of concepts to design, with brief descriptions and dependency
  relationships.
- **Existing draft documents** — any preliminary concept notes (e.g.,
  `concepts/FOUNDATIONAL_ABSTRACTIONS.md`, `DATA_MODEL.md`, `EQUALITY.md`) are treated as
  input material, not as accepted designs.

---

## 5-Step Procedure

Each concept goes through the following 5 steps in order. Procedure steps
capture the creative design work; the deliverable (EDR + concept document)
is handled separately in the Output section below.

| # | Step | Purpose | Answer |
|---|------|---------|--------|
| 1 | Idea/Problem | What problem does this concept solve? | Problem description |
| 2 | Minimal Solution | What is the simplest valid solution? | Solution sketch |
| 3 | Principle Check | Which Design Principles does it satisfy? | Principle mapping |
| 4 | Examples | Show all canonical forms | Examples list |
| 5 | EDR | Record the decision with rationale | EDR file |

### Step Details

#### 1. Idea / Problem

What problem does this concept solve for the programmer? State the gap
or friction. If it solves no independently-statable problem, reject at
this step.

**Output:** A clear problem statement that a competent programmer would
recognise as genuine.

#### 2. Minimal Solution

Describe the simplest valid solution. What is the minimum semantic
addition that solves the problem? If the solution requires multiple
simultaneous additions, it is not orthogonal — split the concept.

**Output:** A solution sketch describing the minimal semantic addition.

#### 3. Principle Check

Map the concept to `DESIGN_PRINCIPLES.md`. Which principles does it
satisfy? Does it violate any principle? If it violates any
closed-for-modification principle, the concept requires a **Tier 1 EDR**
and is ineligible for solo-author acceptance.

**Output:** A table of principles with pass/fail per principle, and
notes for any flagged violations.

#### 4. Examples

Write all canonical forms — every equivalent way to express this
feature. Follow the *Show All Canonical Forms* principle from
`DESIGN_PRINCIPLES.md`. Examples precede semantic explanation.

**Output:** Code examples demonstrating minimal, typical, edge case,
and incorrect usage categories.

#### 5. EDR

Create an Engineering Decision Record (Architecture category) accepting
the concept. Use the [`_edr-architecture.md`](templates/_edr-architecture.md)
template. The EDR must cite the Problem, Solution, and Principle Check
from steps 1-3.

**Output:** One `decision_records/architecture/EDR-NNN-concept-name.md` file.

---

## Validation

After the 5-step procedure is complete, the concept design must be
validated through all 7 Decision Validation gates defined in
[`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md) — see that
document for the gate catalogue with criteria tables and gate selection
matrix.

Each gate produces a binary verdict (pass / fail) or a conditional flag.
A proposal with any **fail** or unresolved **flag** must be revised
before moving to specification.

The [`_language-design.md`](gates/_language-design.md) checklist provides
the concrete review form that records the outcome of each gate check.

---

## Output

The deliverables of a Concept Design Review are:

1. **Concept document** in `docs/how/concepts/research/` — the formal semantic
   definition of the concept (see the [`_concept.md`](templates/_concept.md)
   template for the required structure).
2. **EDR** in `docs/how/decision_records/architecture/` — records the decision, rationale, and
   alternatives considered (Architecture category).
3. **CORE_CONCEPTS.md registration** — the concept is entered into the
   accepted-concept registry at `what/CORE_CONCEPTS.md`.

---

---

## Acceptance Gate

The transition from **DRAFT** (concept research in `how/concepts/research/`) to
**Accepted** (registered in `what/CORE_CONCEPTS.md`) requires:

| Criteria | Requirement |
|----------|-------------|
| **Owner** | Solo author (per `TODO.md` convention) |
| **Procedure** | All 5 Concept Design Review steps completed |
| **EDR** | Architecture-category EDR filed in `decision_records/architecture/` |
| **Gate validation** | Passes all 7 Decision Validation gates (`DECISION_VALIDATION.md`) |
| **LLM Generability Gate** | Passes the 5-criteria LLM Generability Gate check — see [`llm-native-concept-shortlist.md`](../notes/llm-native-concept-shortlist.md) § LLM Generability Gate Requirement |
| **Template compliance** | Concept document follows the 8-section `_concept.md` template — audited per [`DOCUMENTATION_PRINCIPLES.md`](DOCUMENTATION_PRINCIPLES.md) § Template Compliance Checklist |

Once all criteria are met, the concept is entered into `what/CORE_CONCEPTS.md`,
the EDR is indexed in `decision_records/INDEX.md`, and the research document
graduates from `how/concepts/research/` to `what/concepts/`.

---

## How to Maintain This Document

- Keep the procedure in sync with the [`ROADMAP`](../when/ROADMAP.md)
  and [`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md).
- If the number or order of steps changes, update the table and all
  cross-references.
- If a new validation gate is added, update the "Maps To" column and
  the validation table.
