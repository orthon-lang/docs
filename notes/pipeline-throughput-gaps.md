# Pipeline Throughput Gaps

> Analysis of systemic throughput problems in the Orthon design pipeline, with a proposed minimal solution.

**Date:** 2026-07-21

---

## Context

After three days of work, the Orthon project has produced 7 validation gates, an 11-stage concept design review, fitness functions, a process registry, and 10 EDRs — all about *how to design*. Zero language design decisions have been made. The `CORE_CONCEPTS.md` registry is empty, and 30+ research drafts sit in `how/concepts/research/`. The machinery consumes effort and produces process, not concepts.

---

## Four Gaps

### Gap 1 — The pipeline has never been used

The full pipeline (research → design review → gate validation → EDR → CORE_CONCEPTS.md) has not completed a single round-trip. The mechanism exists but has never been exercised end-to-end. The project has:

- 7 validation gates
- 11-stage concept design review
- 13 fitness functions monitoring document structure
- 10 EDRs (all meta-process — zero language design decisions)

**Consequence:** The pipeline is untuned. We don't know which steps add value, which are redundant, or whether the output quality justifies the overhead.

### Gap 2 — Process calibrated for teams, not solo author

The concerns documented in `how/PROCESS_INVENTORY.md` ("Without this…") assume multi-contributor coordination costs, groupthink risk, and institutional amnesia. A solo designer who can hold the entire design in one head does not need 7 independent validation perspectives.

**Consequence:** Over-engineered process overhead slows the solo author without proportional quality gain.

### Gap 3 — Decision process is fragmented and hard to find

A human designer cannot discover the process from `README.md`. `AGENTS.md` — labelled "For AI agents" — is actually the decision-making guide. Three documents (`DECISION_VALIDATION.md`, `_language-design.md`, `concept-design-review.md`) duplicate the same gate criteria. The pipeline, gates, EDRs, and concept registry have no connecting Published Language.

**Consequence:** Process discovery friction discourages use. A solo author won't read four documents to understand how to design one concept.

### Gap 4 — No pipeline observability

13 fitness functions monitor document structure; none measure decision throughput, cycle time, or concepts-in-flight.

**Consequence:** We can detect an orphaned cross-reference but cannot detect that no concept has been accepted in 3 days.

---

## Proposed Solution (Top 5)

| # | Action | Impact |
|---|--------|--------|
| **1** | **Run ONE concept TODAY.** Take `EQUALITY.md` or `DATA_MODEL.md`, push through a 3-step pipeline (Problem → Principle Check → Examples), accept into `CORE_CONCEPTS.md`. This breaks the logjam and calibrates the pipeline. | 🔴→🟢 |
| **2** | **Collapse 11 steps to 5.** Full 11-step + 7-gate mechanism may be a retroactive refinement pass during Epic 4, not a blocking gate on first pass. | 🔴→🟡 |
| **3** | **Create `DECISION_PROCESS.md`.** One page: "Here is how you design a language concept". Extract from `AGENTS.md` and `concept-design-review.md`. Both `README.md` and `AGENTS.md` reference it. | 🟡→🟢 |
| **4** | **Add throughput fitness functions.** Minimum: concepts-in-flight, oldest open proposal age. | 🟡→🟡 |
| **5** | **Next EDR must be a language design decision.** Reorient the EDR system to its stated purpose. | 🟡→🟢 |

---

## Minimal Pipeline (solo author)

The simplest decision process that works for an individual language designer:

```text
Idea → Problem Description → Minimal Solution → Principle Check → Examples → EDR → CORE_CONCEPTS.md
```

Not 11 steps. Not 7 independent gates. Five steps, one decision at a time, each accepted concept building on the previous one. Add process only when a real concept breaks in a way the simple pipeline could not detect.

---

## Related Documents

- `how/PROCESS_INVENTORY.md` — Process registry with traceability matrix
- `how/concept-design-review.md` — Current 11-stage review (candidate for collapse)
- `how/gates/DECISION_VALIDATION.md` — 6 validation gates (candidate for consolidation)
- `how/gates/_language-design.md` — Language design gate checklist (candidate for consolidation)
- `AGENTS.md` — Decision guide currently hidden behind "AI agents" label
- `what/CORE_CONCEPTS.md` — Accepted concept registry (currently empty)
- `how/concepts/research/` — 30+ draft analyses (never promoted)
- `how/architecture/FITNESS_FUNCTIONS.md` — Fitness functions (no throughput metrics)
