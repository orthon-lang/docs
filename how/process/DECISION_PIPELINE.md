# Decision Pipeline

> **⚠️ DRAFT — Placeholder for Phase 4.**
> This document defines the 10-question pipeline that every proposed
> language feature must pass before detailed design begins.
>
> **Status:** Placeholder — to be filled during Phase 4 of M1.
> **See also:** [`ROADMAP.md`](../../when/ROADMAP.md) § Phase 4,
> [`DECISION_PROCESS.md`](DECISION_PROCESS.md),
> [`Concept Design Review`](../../how/concept-design-review.md)

---

## The Pipeline

For each proposed feature, run through these 10 questions in order:

```
 1. What problem are we solving?
 2. Is this a language problem or a library problem?
 3. Can it be solved with existing primitives?
 4. Does it violate any Design Principle?
 5. Does it add new semantics (vs. syntactic sugar)?
 6. Can it be expressed through composition?
 7. Can it be syntactic sugar over existing primitives?
 8. Is this an optimisation, not semantics?
 9. Does it affect backward compatibility?
10. Is it worth adding at all?
```

## Decision Flow

```
Proposal
    │
    ▼
Q1–Q3 ──► If "library problem" or "exists in primitives" → REJECT as language feature
    │
    ▼
Q4     ──► If violates principle → REJECT (or escalate to EDR for principle change)
    │
    ▼
Q5–Q7  ──► If expressible as sugar or composition → mark as syntactic sugar
    │
    ▼
Q8     ──► If optimisation, not semantics → move to OPTIMIZATION_MODEL
    │
    ▼
Q9     ──► Assess compatibility impact
    │
    ▼
Q10    ──► Final value judgement → ACCEPT / DEFER / REJECT
```

## Pipeline Application

<!-- To be filled during Phase 4 — one entry per feature processed -->
