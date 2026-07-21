# Concept Design Review

> The 11-step procedure for designing and validating each language
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
undergoes a uniform 11-step procedure that ensures the design is:

- **Problem-driven** — starts from a real programmer need, not a
  technically interesting idea.
- **Principle-compliant** — verified against the Manifesto and Design
  Principles.
- **Validated** — passes all six Decision Validation gates before
  acceptance.
- **Traceable** — every decision is recorded with rationale,
  alternatives considered, and an Engineering Decision Record.

---

## Relationship to Other Documents

The Concept Design Review is one of three complementary artifacts that
govern language design decisions. They serve distinct roles:

| Artifact | Role | Answers | Applied |
|----------|------|---------|---------|
| **11-step procedure** (this document) | **Process** — what to do, in what order | *How do we design a concept?* | During each concept review |
| **Decision Validation gates** (`DECISION_VALIDATION.md`) | **Criteria** — what the result must satisfy | *Is the design sound?* | Evaluates the completed 11-step output |
| **Language Design Gate** (`_language-design.md`) | **Checklist** — what to verify concretely | *Did we check everything?* | Operationalises the gates into a review form |

**The flow:** The 11-step procedure produces a concept design. That design
is then evaluated through the six Decision Validation gates. The
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

## 11-Step Procedure

Each concept goes through the following 11 steps in order. Every step
maps to one or more validation gates or checklist criteria, as shown in
the third column.

| Step | Section | Description | Maps To |
|------|---------|-------------|---------|
| 1 | **Problem** | What problem does this concept solve? Without which problem would its existence be meaningless? | `USER_VALUE_GATE` · [`_language-design.md`](gates/_language-design.md) § Problem-first |
| 2 | **Use Cases** | Where is it used? What real problems does it solve? | `USER_VALUE_GATE` |
| 3 | **Necessity** | Can we live without it? If removed, what becomes impossible? What becomes inconvenient? | `CONCEPTUAL_SIMPLICITY_GATE` |
| 4 | **Sufficiency** | Is the concept too large? Can it be split? Does it contain unnecessary capabilities? | `CONCEPTUAL_SIMPLICITY_GATE` · [`_language-design.md`](gates/_language-design.md) § Minimality |
| 5 | **Alternatives** | What alternative implementations exist? (e.g., generator → yield → iterator → stream → pipeline → callback) | `LONG_TERM_MAINTAINABILITY_GATE` |
| 6 | **Principles Check** | Verified against all Design Principles (Simplicity, Orthogonality, Explicitness, etc.) | `LOGICAL_CONSISTENCY_GATE` · [`_language-design.md`](gates/_language-design.md) § Principle compliance |
| 7 | **Interactions** | How does this concept interact with every other concept? Any non-orthogonal behaviours? | `ARCHITECTURAL_INTEGRITY_GATE` · [`_language-design.md`](gates/_language-design.md) § Interaction analysis |
| 8 | **Rationale** | Why was this specific option chosen? Why were others rejected? | [`_language-design.md`](gates/_language-design.md) § Traceability |
| 9 | **Examples** | Minimal, typical, edge case, and incorrect usage examples | Comprehension check (no gate) |
| 10 | **Open Questions** | What remains unresolved? | `LONG_TERM_MAINTAINABILITY_GATE` |
| 11 | **EDR** | If a decision is made, it is formalised as an Engineering Decision Record (Architecture category) | [`_language-design.md`](gates/_language-design.md) § Decision journal |

### Step Details

#### 1. Problem

State the problem in **user terms**, not implementation terms. A
realistic code example should demonstrate the pain point. The problem
must be justified by the project's Vision.

**Output:** A clear problem statement that a competent programmer would
recognise as genuine.

#### 2. Use Cases

Describe where and when this concept is used. Cover the common cases,
not just the edge cases. Use cases help validate that the problem is
real and the concept addresses it.

**Output:** A list of concrete scenarios with representative inputs and
expected outcomes.

#### 3. Necessity

Test whether the concept earns its place. Ask: without this concept, is
the problem genuinely unsolvable or just inconvenient? If the answer is
"inconvenient", assess whether the inconvenience justifies the cognitive
cost of a new concept.

**Output:** A necessity verdict (essential / valuable / marginal) with
reasoning.

#### 4. Sufficiency

Check that the concept is not too large. Can it be split into smaller
orthogonal concepts? Does it contain capabilities that belong elsewhere?
A concept should solve exactly one problem.

**Output:** A sufficiency verdict (atomic / could be split / overgrown)
with proposed splitting if applicable.

#### 5. Alternatives

Survey the design space. For each alternative, document what it does
well, what it does poorly, and why it was rejected. This step prevents
the team from rediscovering rejected options later.

**Output:** A table of alternatives with pros, cons, and rejection
rationale.

#### 6. Principles Check

Verify the concept against all Design Principles. If a violation is
found, it must be intentional and documented — never accidental.

**Output:** A table of principles with pass/fail per principle, and
notes for any flagged violations.

#### 7. Interactions

For each existing accepted concept, describe how the new concept
interacts with it. Identify any non-orthogonal behaviours, special
cases, or context-dependent semantics. This step directly feeds the
interaction matrix consolidated in Milestone 3.

**Output:** Per-existing-concept interaction descriptions, feeding into
the [interaction matrix](../notes/interaction-matrix-format.md) in
Milestone 3.

#### 8. Rationale

Document why the chosen option was selected over the alternatives.
This is the decision record — it captures the reasoning so future
designers understand why things are the way they are.

**Output:** A narrative explanation of the decision, referencing the
alternatives considered (step 5) and principles check (step 6).

#### 9. Examples

Provide four categories of examples:

- **Minimal** — the simplest possible correct usage.
- **Typical** — realistic usage in context.
- **Edge case** — unusual but valid inputs or states.
- **Incorrect** — common mistakes that should be caught.

**Output:** Code examples demonstrating each category.

#### 10. Open Questions

Record any unresolved questions about the concept. These may be
deferred to Milestone 3 (Cross-cutting Review) or noted as future
work. If there are no open questions, state that explicitly.

**Output:** A list of open questions with owner (if assigned) and
target milestone for resolution.

#### 11. EDR (Architecture)

Formalise the decision as an Engineering Decision Record using the
[EDR Architecture template](templates/_edr-architecture.md). The EDR captures
the context, decision, consequences, and links to the concept document.

**Output:** One `decision_records/architecture/EDR-NNN-concept-name.md` file per accepted concept.

---

## Validation

After the 11-step procedure is complete, the concept design must be
validated through all six Decision Validation gates defined in
[`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md):

| Gate | What It Checks |
|------|----------------|
| `USER_VALUE_GATE` | Does this solve a real problem for the programmer? |
| `LOGICAL_CONSISTENCY_GATE` | Is the proposal internally consistent? |
| `CONCEPTUAL_SIMPLICITY_GATE` | Is it as simple as it can be? |
| `ARCHITECTURAL_INTEGRITY_GATE` | Does it fit within the layered architecture? |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Can it be defined strategy-agnostically? |
| `LONG_TERM_MAINTAINABILITY_GATE` | Will this decision age well? |

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

---

## How to Maintain This Document

- Keep the procedure in sync with the [`ROADMAP`](../when/ROADMAP.md)
  and [`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md).
- If the number or order of steps changes, update the table and all
  cross-references.
- If a new validation gate is added, update the "Maps To" column and
  the validation table.
