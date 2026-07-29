# Nondeterminism Gaps — Open Concerns

> **Date:** 2026-07-29
> **Source:** Gap analysis derived from checking BAML gap §7 ("Nondeterminism
> as a First-Class Semantic Concern") against Orthon's existing design.
> **See also:**
> [`LLM_AS_DELEGATE_IMPL.md`](../how/concepts/research/important/LLM_AS_DELEGATE_IMPL.md),
> [`PARALLELISM_EXECUTOR_MODEL.md`](../how/concepts/research/important/PARALLELISM_EXECUTOR_MODEL.md),
> [`IDEMPOTENT_GENERATION.md`](../how/concepts/research/deferrable/IDEMPOTENT_GENERATION.md),
> [`orthon-testability-vs-baml-test-dsl.md`](orthon-testability-vs-baml-test-dsl.md),
> [`baml-concepts-orthon-gap-analysis.md`](baml-concepts-orthon-gap-analysis.md)

---

## Context

Orthon's `act`/`delegate` architecture confines nondeterminism to delegate
implementations (`impl llm`, `impl remote`, etc.) while keeping the core
language semantic model deterministic (per `DESIGN_PRINCIPLES.md` §
Deterministic Behavior). This is an **intentional architectural decision**,
not an oversight.

However, three gaps remain open:

---

## Gap A: Transitivity Through the Call Graph

**Problem:** If `act A` calls `act B` via `impl llm`, then everything that
calls `A` transitively is also nondeterministic. The language does not track
or mark this propagation.

**What exists:** `LLM_AS_DELEGATE_IMPL.md` acknowledges nondeterminism as a
property of delegate implementations but proposes no mechanism for tracking
it through the call graph.

**What is missing:**
- A type-level marker (e.g. `fun foo() -> Int !nondet`)
- An effect system for nondeterminism
- A compiler facility: "this function is transitively nondeterministic"

**Status:** ❌ Not addressed

---

## Gap B: Testing Nondeterministic Code

**Problem:** If Orthon gains native LLM calls, `fun`'s determinism guarantee
breaks. There is no concept of quorum/replay/snapshot testing for
nondeterministic delegates.

**What exists:** `orthon-testability-vs-baml-test-dsl.md` (notes/) identifies
the problem. `LLM_AS_DELEGATE_IMPL.md` mentions "replay and snapshot testing"
as a possibility, but without specification.

**What is missing:**
- Quorum tests (BAML: `test "..." with quorum { ... }`)
- A record/replay mechanism for LLM responses at the language level
- A standard way to mock an LLM delegate with a deterministic `impl`

**Status:** ❌ Not addressed (blocks on LLM-calling concept)

---

## Gap C: Execution Model — Nondeterminism Not Accounted For

**Problem:** `EXECUTION_MODEL.md` is a placeholder for Phase 7. When that
phase begins, delegate nondeterminism must be integrated.

**What exists:** Placeholder. No decisions.

**What is missing:**
- Description of non-replayable delegates
- Impact of nondeterminism on execution order guarantees
- Reconciliation of delegate nondeterminism with IR determinism
  (`IR.md`: "The same source program always produces the same IR")

**Status:** ⏳ Phase 7 — to be addressed; awareness captured here

---

## Related: Architectural Decision Not Documented

The decision to confine nondeterminism to delegate implementations rather
than introducing a first-class semantic dimension or effect system is not
formally recorded as an EDR or gate entry.

**Recommendation:** Create a gate entry in `_language-design.md` when ready,
documenting: "Nondeterminism is a property of delegate implementation, not
language semantics. The language does not introduce an effect system for
nondeterminism at this stage."

---

## Recommended Actions (ordered by priority)

| # | Action | Location | Priority |
|---|--------|----------|----------|
| 1 | Add a "Nondeterminism" awareness section to `EXECUTION_MODEL.md` (before Phase 7) | `what/EXECUTION_MODEL.md` | Medium |
| 2 | Create gate entry documenting the confine-nondeterminism decision | `how/gates/_language-design.md` | Medium |
| 3 | Plan quorum-testing concept alongside LLM-calling concept | Future LLM concept phase | Low (blocks on LLM) |
