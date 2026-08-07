# ATAM / CBAM Architecture Evaluation

> How Orthon evaluates its own architecture using adapted versions of the SEI methods ATAM and CBAM.

**Date:** 2026-08-07

---

## Purpose

This note records how Orthon evaluates its own architecture. The project
relies on **adapted** versions of two SEI (Software Engineering Institute)
evaluation methods:

- **ATAM** — Architecture Tradeoff Analysis Method
- **CBAM** — Cost Benefit Analysis Method

The methods are not applied "as-is" because Orthon is a language
**specification**, not a deployed software system. The methodology is
transferred, while the quality attribute model, scenario set, and
stakeholder mechanism are re-targeted to a design-only, solo-authored
project.

---

## Why adaptation is needed

| Native ATAM/CBAM assumption | Orthon reality |
|---|---|
| Evaluates a software system (components, connectors, deployment) | Evaluates layers of contracts: Core Language, Syntax, Standard Library, Implementation Strategy (Policies), Execution Environment |
| Runtime + lifecycle quality attributes (performance, availability, security) | Semantic / design attributes: orthogonality, consistency, implementation independence, LLM generability, evolvability |
| Stakeholder workshop — main value is stakeholder alignment | Solo-authored — the method becomes a self-audit; the workshop effect is lost |
| CBAM: cost and benefit in money and effort | Cost = design effort; benefit = speculative future value for humans and LLMs |

---

## Mechanism components (adapted ATAM)

1. **Utility tree** — business drivers → quality attributes → refinements → scenarios, each with priority and risk.
2. **Quality-attribute scenarios** — stimulus → environment → response → measure.
3. **Sensitivity points** — an architectural decision affects a single attribute.
4. **Tradeoff points** — a decision affects multiple attributes (e.g., minimal core vs expressiveness).
5. **Risk themes** — unverified claims that could invalidate the architecture.

## CBAM-lite

When selecting which candidate concepts enter v0.1, score each candidate as:

```
benefit × probability / design cost
```

- **benefit** — value to the two target audiences (humans + LLMs).
- **probability** — likelihood the benefit materialises.
- **design cost** — effort to specify the concept correctly.

This is a lightweight ranking heuristic, not an economic model. It
formalises what the Decision Pipeline
([`../how/process/DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md))
already does intuitively.

---

## Applications

### 1. ATAM-lite self-audit — before the v0.1 freeze

Run at the Phase 8 / M1 boundary (see
[`../when/ROADMAP.md`](../when/ROADMAP.md)). Produce: a utility tree, a
systematic scenario catalogue with measures, and an inventory of tradeoff
points, sensitivity points, and risk themes. This is the cheapest point to
fix design flaws — corrections at specification level cost little.

### 2. CBAM-lite — at concept selection

Score candidate concepts (especially from the `deferrable/` research tier)
when deciding v0.1 scope.

### 3. Full ATAM with runtime quality attributes — M4 (Implementation)

Deferred until a compiler and runtime exist (Milestone 4, implementation
in a separate repository). Only then do runtime quality attributes
(performance, availability, deployability) become measurable. The
evaluation must then target concrete Implementation Strategies, not
specification layers.

---

## Mapping to existing artifacts

| ATAM mechanism | Existing Orthon artifact |
|---|---|
| Quality-attribute scenarios / gates | [`../how/gates/DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md) — seven independent gates, each with a lens and method |
| Fitness functions against decay | [`../how/architecture/FITNESS_FUNCTIONS.md`](../how/architecture/FITNESS_FUNCTIONS.md) |
| Composability matrix / forbidden pairs | [`../../what/CROSS_CUTTING.md`](../../what/CROSS_CUTTING.md) |
| Conflict inventory | [`../../what/CONFLICT_REGISTRY.md`](../../what/CONFLICT_REGISTRY.md) |
| Decision registration | EDR system + Decision Pipeline |

---

## Example scenarios with measures

| Scenario | Stimulus | Environment | Response | Measure |
|---|---|---|---|---|
| Strategy interchangeability | A new Implementation Strategy (e.g., GPU execution) is added | Any specification layer | No Core Language change required | Diff across layers = 0 |
| Composability | A new concept is proposed | Orthon core | Passes all seven gates, no forbidden pair added | Interaction matrix without new forbidden pairs |
| LLM generability | An LLM generates code from the schema of a construct | Toolchain | Correct code, detectable errors | Schema round-trip accuracy |
| Evolution | A concept changes after the v0.1 freeze | Evolution model | Versioned without breaking contracts | Zero broken contracts without deprecation |

---

## References

- [`../how/architecture/FITNESS_FUNCTIONS.md`](../how/architecture/FITNESS_FUNCTIONS.md)
- [`../how/gates/DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md)
- [`../when/ROADMAP.md`](../when/ROADMAP.md) — M1 (freeze) and M4 (Implementation)
- [`../how/process/DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md)
