# Phase 4: Derived Features & Decision Pipeline — Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-07-27
**Phase:** 04-derived-features-and-decision-pipeline
**Areas discussed:** Processing order & batching strategy, Decision Pipeline depth per concept, Language vs StdLib classification criteria, Anti-pattern to concept design integration, Essential-tier edge cases, Rejection EDR depth

---

## Processing Order & Batching Strategy

**Area selected:** Multi-select from all 6 presented options.

| Option | Description | Selected |
|--------|-------------|----------|
| Essential-first by tier | Process all 22 essential-tier concepts first, then 36 important-tier. Clean tier-based ordering. | ✓ |
| Family-based batching | Group concepts by family (Data, Control, Meta) — process each family as a batch. | |
| Decision Pipeline first | Finalize DECISION_PIPELINE.md first, then run ALL ~58 concepts through just that pipeline for triage. | |

**User's choice:** Essential-first by tier.
**Notes:** Also selected "deep design" approach overall.

### Sub-question: Within essential tier ordering

| Option | Description | Selected |
|--------|-------------|----------|
| Clean concepts first | Known language features first, then Policy-level and unclassified edge cases in second wave. | ✓ |
| ROADMAP priority | Follow ROADMAP.md: Core → Control → Data → Meta. | |
| Alphabetical | Simple alphabetical ordering. | |

**User's choice:** Clean concepts first.
**Notes:** ROADMAP priority table used for sub-ordering within clean concepts.

### Sub-question: Anti-pattern research sequencing

| Option | Description | Selected |
|--------|-------------|----------|
| Before concept design | Complete all anti-pattern research first so findings inform concept designs. | |
| Alongside matching concepts | Process each anti-pattern alongside its related concept family. | |
| Minimum first batch then parallel | Do a few first, then run the rest in parallel with concept design waves. | |

**User's choice:** [Free text] "crutches уже разобраны по концепциям. этот блок можно пропустить"
**Notes:** Imperative-crutch anti-patterns already integrated into concept research docs. No separate processing needed.

---

## Decision Pipeline Depth per Concept

### Sub-question: Pipeline results recording

| Option | Description | Selected |
|--------|-------------|----------|
| Documented in pipeline | Each concept gets a documented Q&A entry in DECISION_PIPELINE.md's application log. | |
| EDR-only output | Run pipeline mentally, only produce EDR. | |
| Tier-dependent | Essential-tier gets documented; important-tier gets EDR-only. | |

**User's choice:** [Free text] "некоторые концепты размазаны по нескольким файлам с гипотизами. рассматривать их по отдельности не имеет смысл. Для каждой концепции проходим весь пайплайн принятия решения. все гейты. регистрируем результаты. создаем EDR. Затем создаем общий агрегирующий EDR как делали на предыдущих фазах."
**Notes:** Aggregate related research files per concept. Full pipeline + all gates + EDR per concept + aggregating EDR (following EDR-013/016 pattern).

### Sub-question: Concept Design Review depth

| Option | Description | Selected |
|--------|-------------|----------|
| Full Design Review | All 5 steps per concept with concrete Orthon examples and canonical forms. | ✓ |
| Pipeline + Gates + EDR | Decision Pipeline + 7 validation gates + EDR. | ✓ |
| Tier-dependent | Essential gets full Design Review; important gets Pipeline + Gates + EDR. | |

**User's choice:** [Free text] "1 + 2"
**Notes:** Both options — full Concept Design Review (5-step) AND Pipeline + Gates + EDR. Full treatment for every concept.

---

## Language vs StdLib Classification Criteria

### Sub-question: Primary criterion

| Option | Description | Selected |
|--------|-------------|----------|
| Semantic uniqueness | Language if it adds new semantics not expressible through composition of existing primitives. | |
| Compiler dependency | Language if the compiler MUST understand it. | |
| Both criteria | Both semantic uniqueness AND compiler dependency, with semantic uniqueness as primary gate. | ✓ |

**User's choice:** Both criteria.
**Notes:** Semantic uniqueness is primary; compiler dependency is secondary.

### Sub-question: Boundary documentation

| Option | Description | Selected |
|--------|-------------|----------|
| Standalone + EDR field | LIBRARY_BOUNDARY.md as canonical reference; each concept EDR includes classification field. | |
| EDR field only | Each concept EDR records classification; no standalone document. | ✓ |
| Standalone only | Single LIBRARY_BOUNDARY.md table as sole classification authority. | |

**User's choice:** EDR field only.
**Notes:** But DERIV-02 requires LIBRARY_BOUNDARY.md — user opted for lightweight summary table derived from EDRs.

---

## Essential-Tier Edge Cases

### Sub-question: Policy-level concepts

| Option | Description | Selected |
|--------|-------------|----------|
| Run through pipeline but classify as Policy | Run Decision Pipeline, note they're Policy, file under Implementation Strategies area. | ✓ |
| Skip pipeline, file directly | These are Implementation Policy decisions — skip the concept pipeline. | |
| Light pipeline | Quick Decision Pipeline pass (3-5 questions) + EDR. | |

**User's choice:** Run through pipeline but classify as Policy.
**Notes:** ALLOCATION.md, REGION_BASED_MEMORY_MANAGEMENT.md, EXECUTION_PROGRAM.md.

### Sub-question: Unclassified essential concepts

| Option | Description | Selected |
|--------|-------------|----------|
| Phase 4 (this phase) | Process here alongside other essential concepts. | |
| Phase 5 (Syntax) | Syntax-level concerns — better addressed during Syntax Design. | |
| Phase 2/3 revisit | May affect Semantic Model or Primitive Blocks. | |

**User's choice:** [Free text] "рассматриваем на этой фазе, но если потребуется вносим коррективы в Semantic Model or Primitive Blocks"
**Notes:** CONTEXT_PARAMETERS.md and REPRESENTATION_MODIFIERS.md processed in Phase 4 with permission to correct Semantic Model/Primitive Blocks.

---

## Rejection EDR Depth

| Option | Description | Selected |
|--------|-------------|----------|
| Full EDR per concept | Full EDR for each — context, rationale, which principles violated, alternatives. | ✓ |
| Single grouped EDR | One EDR rejecting all 4 with per-concept rationale sections. | |
| Inline gate entry | Add entries to DECISION_LOG.md or the concept docs themselves. | |

**User's choice:** Full EDR per concept.
**Notes:** PROTOTYPE, SIGNIFICANT_WHITESPACE, DYNAMIC_TYPING, CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION. Each includes context, principle violations, alternatives, verdict. Move to reject/ directory.

---

## the agent's Discretion

- Exact ordering within concept families (which Core concept to process first).
- Combined vs. separate EDRs for closely related concepts (e.g., CONCURRENCY_MODEL.md and ACTORS.md).
- Pipeline application log format, LIBRARY_BOUNDARY.md format, deferral table format.
- Canonical form examples formatting in Design Reviews.

## Deferred Ideas

- ~54 deferrable-tier concepts → v0.2/v0.3 with documented rationale.
- Literate Programming — if External, out of scope for M1.
- LLM Toolchain components — lighter gate track, timing TBD.
- "Orthon for LLM" research beyond LLM Generability Gate → post-Freeze.
