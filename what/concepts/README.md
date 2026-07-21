# Orthon Accepted Concepts

This directory contains concept documents that have passed the
**Decision Pipeline** → **Primitive Decomposition** → **Concept Design Review**
pipeline and been accepted via **EDR** (Architecture category).

Each file here is an Orthon-specific concept design — not a research
analysis. Research (from any language) lives in `how/concepts/research/`.

## Pipeline

```
how/concepts/research/    ─┐
                           ├──► Decision Pipeline (10 questions)
Semantic Model ────────────┤         │
Primitive Blocks ──────────┘         ▼
                          Primitive Decomposition Check
                                    │
                                    ▼
                          Concept Design Review
                                    │
                                    ▼
                          EDR acceptance
                                    │
                                    ▼
                          what/concepts/{NAME}.md
                                    │
                                    ▼
                          what/CORE_CONCEPTS.md (registry)
```

## Current Status

**Currently empty** — all existing concept drafts are in
`how/concepts/research/` awaiting formal processing through the
Decision Pipeline (Phase 4 of M1).

Once accepted, concepts graduate to this directory, and an entry is
added to `what/CORE_CONCEPTS.md`.
