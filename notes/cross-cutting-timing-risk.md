# Cross-Cutting Timing Risk — Why Phase 6 as a separate phase is problematic

> Saved from discussion on 2026-07-28.

## The problem

Phase 4 (Derived Features) designs and accepts concepts via EDR. Phase 6 (Cross-Cutting Review) checks all concept pairs for conflicts — but only after Phase 4 is fully complete.

This creates a **late-discovery risk**: if a conflict is found in Phase 6, it may require revising an EDR from Phase 4, which in turn affects syntax already designed in Phase 5. The cost of fixing grows with each subsequent phase.

## Why the current design exists

- Phase 6 requires the **full concept set** — you cannot check `Generator × Error Handling` until both are designed.
- Doing O(N²) pair checks after *every* new concept would be wasteful: most pairs don't interact, and early checks would miss pairs involving concepts not yet designed.

## Proposed improvement: per-wave cross-check gate

Instead of deferring all cross-cutting to Phase 6, integrate **lightweight cross-checks as a gate at the end of each wave** within Phase 4:

```
Phase 4
├── Wave 1: Concepts A, B, C
│   └── Gate: check A×B, A×C, B×C
├── Wave 2: Concepts D, E
│   └── Gate: check all pairs A..E
└── Wave 3: ...
    └── Gate: check all pairs
```

This catches conflicts **before** subsequent waves build on them, while avoiding O(N²) after every single concept. The cross-check is scoped to the growing set of accepted concepts, so by the final wave it naturally converges to the same coverage as a dedicated Phase 6 — but conflicts are found incrementally.

If a conflict is found:
1. Resolve it immediately (revise the relevant EDR)
2. Document it in `CONFLICT_REGISTRY.md` as it happens
3. Proceed to the next wave with the resolved design

Phase 6 then becomes a **final verification pass** (not a discovery phase), confirming no conflicts were missed and that all documented resolutions are consistent.

## Relationship to other notes

- See `notes/phase-4-purpose.md` for Phase 4 overview
- See `notes/interaction-matrix-format.md` for interaction matrix format research
- See `what/CONFLICT_REGISTRY.md` for conflict tracking (populated incrementally)
