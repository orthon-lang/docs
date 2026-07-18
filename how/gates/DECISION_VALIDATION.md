# Decision Validation

> Every language decision must pass a set of independent validation
> gates. Each gate examines the proposal from a different perspective
> using a dedicated method.

---

## Overview

A language design proposal is not complete when it works in one
context — it must be sound across multiple independent dimensions.
Decision Validation formalises this requirement: before a proposal
enters the formal specification, it is evaluated through a set of
**validation gates**, each with its own lens and method.

| Gate | Core Question | Method |
|------|---------------|--------|
| `USER_VALUE_GATE` | Does this solve a real problem for the programmer? | [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md) |
| `LOGICAL_CONSISTENCY_GATE` | Is the proposal internally consistent and free of contradictions? | [Socratic Method](methods/SOCRATIC_METHOD.md) |
| `CONCEPTUAL_SIMPLICITY_GATE` | Is the concept as simple as it can be? | [Scientific Method](methods/SCIENTIFIC_METHOD.md) |
| `ARCHITECTURAL_INTEGRITY_GATE` | Does it fit within the layered architecture? | [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md) |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Can it be defined without tying to a specific strategy? | [TRIZ](methods/TRIZ_METHOD.md) |
| `LONG_TERM_MAINTAINABILITY_GATE` | Will this decision age well? | [Einstein's Method](methods/EINSTEIN_METHOD.md) |

These gates are **independent** — a proposal that passes one gate may
still fail another. Each gate produces a binary verdict (pass / fail)
or a conditional flag. A proposal with any **fail** or unresolved
**flag** must be revised before moving to specification.

---

## Gate Selection

Not every proposal needs to pass every gate. The set of required
gates depends on the **type of decision** being validated.

| Decision Type | Gates Typically Required | Optional Gates |
|---------------|-------------------------|----------------|
| New language construct | All six | — |
| Syntax change | `LOGICAL_CONSISTENCY_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE` | `USER_VALUE_GATE` (if purely syntactic sugar), `IMPLEMENTATION_INDEPENDENCE_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` |
| Semantic refinement | `LOGICAL_CONSISTENCY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE` | `CONCEPTUAL_SIMPLICITY_GATE` (if narrowing existing concept) |
| Standard Library addition | `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` | `ARCHITECTURAL_INTEGRITY_GATE` (if within Library), `IMPLEMENTATION_INDEPENDENCE_GATE` |
| Compiler optimisation | `IMPLEMENTATION_INDEPENDENCE_GATE` | All others (optimisations do not change semantics) |

The default for any proposal that adds, changes, or removes a
language rule is **all six gates**.

---

## Gate Catalogue

### `USER_VALUE_GATE`

**Perspective:** *Does this solve a real problem justified by the
project's Vision?*

A language feature must earn its place by serving a tangible
programmer need, not by being technically interesting. The burden of
proof is on the proposal.

**Method:** [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md) —
start from the programmer's experience and reason backwards to the
language construct.

| Criterion | Pass | Flag | Fail |
|-----------|------|------|------|
| The problem is stated in user terms (not implementation terms) | Clearly described from the programmer's perspective | Partially user-stated | Stated only in implementation terms |
| The problem is justified by `../../why/VISION.md` | Directly serves a Vision pillar | Indirectly aligned | Contradicts or ignores Vision |
| A realistic code example shows the pain point | Concise, concrete example | Example exists but is contrived | No example provided |
| The benefit outweighs the cognitive cost | Obvious net gain | Marginal trade-off | Cognitive cost exceeds benefit |

**Fail conditions:**
- The problem does not exist outside the compiler implementer's
  experience.
- The feature could be implemented as a library without language
  changes.
- The use case is speculative (no evidence that real code would need
  it).

---

### `LOGICAL_CONSISTENCY_GATE`

**Perspective:** *Is the proposal internally consistent and free of
contradictions?*

A language concept must follow from its own definitions without
producing paradoxes, ambiguous cases, or context-dependent behaviour.

**Method:** [Socratic Method](methods/SOCRATIC_METHOD.md) —
disciplined questioning to expose hidden contradictions and tacit
assumptions.

| Criterion | Pass | Flag | Fail |
|-----------|------|------|------|
| All terms are precisely defined | Every term has a single, clear definition | One ambiguous term | Multiple undefined or contradictory terms |
| No self-referential paradoxes | Rules apply consistently to the concept itself | Edge case identified | Paradox exists (e.g., a rule that invalidates itself) |
| Behaviour is deterministic across contexts | Same input always produces same result | Context-dependent corner case | Behaviour changes by invisible context |
| Composition with existing concepts is defined | Interactions documented for all existing concepts | Partial coverage | Interactions ignored |

**Fail conditions:**
- The definition of the concept contradicts established definitions
  in `../../what/GLOSSARY.md`.
- Two different uses of the same construct produce different
  semantics without explicit syntactic distinction.
- The proposal introduces a special case that only exists to patch
  an inconsistency.

---

### `CONCEPTUAL_SIMPLICITY_GATE`

**Perspective:** *Is the concept as simple as it can be, or could it
be expressed through composition of existing concepts?*

This gate enforces Orthon's commitment to a minimal core. Every new
concept must justify its existence against the question: *"Can this
be achieved with what already exists?"*

**Method:** [Scientific Method](methods/SCIENTIFIC_METHOD.md) —
treat the claim of simplicity as a hypothesis; separate what is known
from what is assumed.

| Criterion | Pass | Flag | Fail |
|-----------|------|------|------|
| Expressible through composition | Cannot be built with existing concepts | Partial overlap with existing concepts | Fully expressible as composition |
| Single responsibility | Solves exactly one problem | Solves one problem with side effects | Solves multiple unrelated problems |
| No keyword or syntax bloat | No new keywords needed (or exactly one justified keyword) | One new keyword with weak justification | Multiple new keywords without justification |
| Minimal learning surface | Concept is learnable in isolation | Depends on understanding two+ other new concepts | Requires understanding a web of new concepts |

**Fail conditions:**
- A library function or existing modifier sequence can produce the
  same result.
- The feature duplicates the responsibility of an existing concept.
- The proposal introduces syntax sugar that could be a named
  function.

---

### `ARCHITECTURAL_INTEGRITY_GATE`

**Perspective:** *Does the proposal fit within Orthon's layered
architecture and compose freely with existing constructs?*

This gate ensures the decision does not violate the project's
architectural boundaries or introduce coupling between layers
that are meant to be independent.

**Method:** [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md) —
derive the consequences of the proposal from explicit premises and
verify consistency with the architecture.

| Criterion | Pass | Flag | Fail |
|-----------|------|------|------|
| Respects layer boundaries | Operates entirely within one architecture layer | Crosses layers with clear justification | Violates layer separation |
| Composes orthogonally | Combines freely with all existing constructs | One known incompatibility | Multiple special-case restrictions |
| No privileged position | Treats all constructs equally | Minor preferential treatment | Some constructs are "more equal" |
| Consistent with `../../how/architecture/ARCHITECTURE.md` | Fully aligned | Minor deviation documented | Contradicts the architecture |

**Fail conditions:**
- The proposal requires the Standard Library to know about compiler
  internals (or vice versa).
- An existing concept must be modified to accommodate the new one.
- The feature creates a layering violation that forces future
  proposals to work around it.

**See also:** `../../how/architecture/ARCHITECTURE.md` for the
layered architecture definition.

---

### `IMPLEMENTATION_INDEPENDENCE_GATE`

**Perspective:** *Can the concept be defined semantically without
tying it to a specific implementation strategy?*

Orthon separates *what* from *how*. A concept must be expressible as
a semantic definition that any implementation strategy can realise
without changing the programmer's mental model.

**Method:** [TRIZ](methods/TRIZ_METHOD.md) — systematically resolve
contradictions between semantic requirements and strategy constraints
without compromising either.

| Criterion | Pass | Flag | Fail |
|-----------|------|------|------|
| Semantic definition is strategy-agnostic | Behaviour fully defined without referencing implementation | Minor implementation leak | Definition requires specific strategy |
| All strategies can support it | All defined strategies can implement it naturally | One strategy requires workaround | At least one strategy cannot implement it |
| Performance is a strategy concern | Performance characteristics are documented as strategy-dependent | Performance implied in semantics | Behaviour changes by strategy |
| Consistent with `../../how/strategies/IMPLEMENTATION_STRATEGIES.md` | Fully aligned | Minor tension documented | Contradicts the strategy model |

**Fail conditions:**
- The proposal's semantics are expressed in terms of how the
  compiler or runtime works internally.
- Different implementation strategies would produce different
  observable behaviour (not just different performance).
- The concept cannot be implemented in at least one supported
  strategy.

**See also:** `../../how/strategies/IMPLEMENTATION_STRATEGIES.md`
for the strategy model, and individual strategy documents for
specific constraints.

---

### `LONG_TERM_MAINTAINABILITY_GATE`

**Perspective:** *Will this decision age well? Is the evolution path
clear, and does the proposal incur conceptual debt?*

A language lives for decades. This gate examines whether the decision
can survive evolution without becoming a constraint or a regret.

**Method:** [Einstein's Method](methods/EINSTEIN_METHOD.md) — radical
simplification as a maturity test: if you cannot explain a concept
simply, it is not yet ready.

| Criterion | Pass | Flag | Fail |
|-----------|------|------|------|
| Evolution path documented | Clear how the feature can be extended without breaking changes | Partial evolution path | No evolution path |
| Conceptual debt assessed | No conceptual debt | Minor debt with documented mitigation | Significant conceptual debt |
| Reversible decision | Decision can be deprecated without ecosystem breakage | Deprecation is costly but possible | Decision is irreversible once adopted |
| Compatibility impact is bounded | Affects only new or isolated scope | Affects existing syntax minimally | Breaks existing code or mental models |

**Fail conditions:**
- The proposal paints the language into a corner where a desirable
  future feature would be impossible.
- Deprecating or removing the feature would break programs in
  unpredictable ways.
- The concept would need to be fundamentally rethought in the next
  major version.
- The proposal relies on a syntactic or semantic pattern that would
  block orthonormal extension.

---

## Methods Overview

Each validation gate applies a distinct reasoning method. These are
not random techniques — they are chosen to cover complementary modes
of thinking, ensuring no single cognitive bias dominates the
validation process.

| Mode | Method | Gate | What it prevents |
|------|--------|------|------------------|
| Empathic | [Working Backwards](methods/WORKING_BACKWARDS_METHOD.md) | `USER_VALUE_GATE` | Building features nobody asked for |
| Critical | [Socratic Method](methods/SOCRATIC_METHOD.md) | `LOGICAL_CONSISTENCY_GATE` | Contradictions hidden by imprecise language |
| Empirical | [Scientific Method](methods/SCIENTIFIC_METHOD.md) | `CONCEPTUAL_SIMPLICITY_GATE` | Over-engineering disguised as completeness |
| Deductive | [Logical Analysis](methods/LOGICAL_ANALYSIS_METHOD.md) | `ARCHITECTURAL_INTEGRITY_GATE` | Layering violations that seem harmless |
| Inventive | [TRIZ](methods/TRIZ_METHOD.md) | `IMPLEMENTATION_INDEPENDENCE_GATE` | Strategy-specific compromises accepted too early |
| Reductive | [Einstein's Method](methods/EINSTEIN_METHOD.md) | `LONG_TERM_MAINTAINABILITY_GATE` | Complexity that becomes permanent debt |

Each method is described in its own file under [`methods/`](methods/),
with the full technique, application steps, and the gate it serves.

---

## Gate Flow

The recommended sequence for a full validation (all six gates) is:

```mermaid
flowchart LR
    UV["USER_VALUE_GATE\nWorking Backwards"] --> LC["LOGICAL_CONSISTENCY_GATE\nSocratic Method"]
    LC --> CS["CONCEPTUAL_SIMPLICITY_GATE\nScientific Method"]
    CS --> AI["ARCHITECTURAL_INTEGRITY_GATE\nLogical Analysis"]
    AI --> II["IMPLEMENTATION_INDEPENDENCE_GATE\nTRIZ"]
    II --> LM["LONG_TERM_MAINTAINABILITY_GATE\nEinstein's Method"]
```

When gates are selectively applied (see [Gate Selection](#gate-selection)),
they should still follow this priority order: user value first,
long-term impact last.

**Preconditions:** Before entering certain gates, the proposal must
satisfy prerequisites established during the Concept Design Review
(see [Relationship to the Design Process](#relationship-to-the-design-process) § Gate Preconditions).

**Rules:**
- A **Fail** at any gate sends the proposal back for revision. The
  revision must address the specific fail condition before
  re-entering the same gate.
- A **Flag** means the proposal may proceed but the flagged issue
  must be resolved before the final gate.
- Gates may be revisited as the proposal evolves. A change that
  addresses a previous fail condition may surface new issues in a
  later gate.
- Optional gates are skipped only when the decision type explicitly
  does not concern that perspective. When in doubt, apply the gate.

---

## Relationship to the Design Process

The validation gates are applied at specific stages of the design
process, defined in the [ROADMAP](../../when/ROADMAP.md). Each
milestone maps to a subset of gates relevant to the work at that
stage.

| Milestone | Primary Gates | Notes |
|-----------|--------------|-------|
| 0 — Vision | — (gates apply to concepts, not vision) | Vision sets the frame; gates operate within it |
| 1 — Inventory | — (cataloguing, not deciding) | No gates needed |
| 2 — Concept Design | All 6 gates per concept | Each concept validated independently |
| 3 — Cross-cutting | `ARCHITECTURAL_INTEGRITY_GATE`, `LOGICAL_CONSISTENCY_GATE` | Conflict detection at concept boundaries |
| 4 — Consistency | `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` | Language-wide audits, redundancy removal |
| 5–7 | — (specification, not decision) | Gates already passed |

### Gate Preconditions

Before entering certain gates, the proposal must satisfy prerequisites
established during the Concept Design Review (Milestone 2):

| Gate | Precondition |
|------|-------------|
| `USER_VALUE_GATE` | A clear **Problem statement** must exist (step 1 of Concept Design Review). Without a well-defined problem, user value cannot be assessed. |
| `ARCHITECTURAL_INTEGRITY_GATE` | **Interactions** with all existing concepts must be documented (step 7 of Concept Design Review). The gate evaluates fit within the architecture, which requires knowing how the concept composes with existing ones. |

These preconditions ensure that no gate is applied in isolation — each
builds on analysis done earlier in the design process.

---

## Relationship to the Language Design Gate

The existing [`_language-design.md`](_language-design.md) is a
fill-in checklist that operationalises the validation gates into a
single concrete review form. While Decision Validation defines
*what* must be checked and *why*, the Language Design Gate provides
the *how* — a structured template for recording the outcome of each
check.

When using the Language Design Gate, each criterion in that checklist
should be traceable to one or more of the validation gates above.
This ensures broad coverage and prevents any single perspective from
dominating the review.

---

## Decision Journal

Use this table to record decisions validated through these gates.
Each row captures one proposal and its outcome across the applied
gates.

| Date | Proposal | Gates Applied | Verdict | Notes |
|------|----------|---------------|---------|-------|
|      |          |               |         |       |
