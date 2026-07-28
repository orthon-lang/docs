# Phase 4 Purpose — Derived Features & Decision Pipeline

> Saved from discussion on 2026-07-28.

Phase 4 is the step where every outstanding language concept that wasn't already covered by Phase 2 (Semantic Model) and Phase 3 (Primitive Blocks) gets formally designed through a rigorous pipeline.

## What Phase 4 does

1. **Decision Pipeline** — each concept passes through a 10-question filter:
   - Is this a language problem or a library problem?
   - Can it be expressed through existing primitives?
   - Does it violate any Design Principle?
   - Is it worth adding at all?

2. **Decomposition onto Primitive Blocks** — each concept is verified to decompose into primitives from Phase 3 (identifier, function, call, scope, etc.). If it can't, the primitive set is incomplete.

3. **Classification** — every feature is classified as:
   - **Language** — built-in semantics
   - **Standard Library** — implementable through the language
   - **External** — out of scope for v0.1

4. **Acceptance via EDR** — every Language-category concept gets an Engineering Decision Record and moves from `how/concepts/research/` (draft) to `what/concepts/` (accepted).

5. **Anti-pattern analysis** — 10 `imperative-crutch-*.md` researches are completed and their findings inform the design.

## End state

- Zero `⚠️ DRAFT` headers remain in `how/concepts/research/`
- Complete EDR registry for all accepted features
- Stable feature set ready for syntax design (Phase 5) and cross-cutting review (Phase 6)

## Dependencies

Depends on Phase 3 (Primitive Blocks) — you need primitives to decompose into.
