# Whitepaper Strategy

> When and why to publish whitepapers for the Orthon language.

---

## What a whitepaper is for a programming language

A whitepaper is an authoritative, external-facing document that explains
the motivation, architecture, and key innovations of a system to a
broader audience — potential users, adopters, investors, or the academic
community.

For a programming language, a whitepaper typically covers:

- **The problem** — why existing languages fall short
- **The solution** — how the new language addresses that gap
- **Architectural innovations** — e.g., Execution Program / Engine separation,
  LLM Generability Gate, orthogonal core
- **Technical depth** — enough substance for an informed reader to judge
  the design's validity

A whitepaper differs from:

| Document | Audience | Purpose |
|----------|----------|---------|
| **EDR / ADR** | Internal design team | Record *why* a decision was made |
| **SPEC** | Implementers | Fix the language precisely |
| **Whitepaper** | External community | Explain, persuade, attract |

---

## When to write a whitepaper

| Stage | Timing | Recommendation |
|-------|--------|----------------|
| **Design phase** (M0–M6) | ❌ Too early | Language design is fluid; a whitepaper would be obsolete before publication |
| **Post-Freeze** (M7 ✓) | ✅ Ideal | Specification is stable; can explain design rationale with confidence |
| **Pre-implementation** (M8–M10) | ✅ Good | Attracts contributors and early adopters |
| **Launch / Release** | ✅ Essential | Accompanies public announcement |

---

## Ideal whitepaper topics for Orthon

Based on Orthon's architectural vision, these whitepapers would have the
strongest impact:

1. **"Orthon: Execution Program / Engine Separation"**
   — The flagship concept. Analogous to how Docker explained containerisation:
   separating *what* a program means from *how* it executes.

2. **"Designing for the LLM Era: The LLM Generability Gate"**
   — Orthon's unique approach: every language construct validated for
   LLM-generability. A strong topic for academic or engineering audiences.

3. **"Orthon: An Explicit, Orthogonal Language for Reliable Software"**
   — General language overview for engineers evaluating adoption.

4. **"From DevOps to LangOps: The Execution Program Model"**
   — How Orthon collapses dev/deploy boundaries by producing a single
   Execution Program deployable via interpreter, AOT compiler, OCI
   builder, or WASM — without rebuilding.

---

## Current status

The project is at **Milestone 0 (Vision)**, still in design phase.
Writing whitepapers now would describe decisions not yet made, creating
documents guaranteed to diverge from reality.

**Recommendation:** Revisit this strategy at **Milestone 7 (Freeze)**.
Add whitepaper authoring as part of the Freeze or post-Freeze rollout
plan.

---

## See also

- [ROADMAP.md](../when/ROADMAP.md) — milestone sequence
- [VISION.md](../why/VISION.md) — architectural vision
- [EXECUTION_PROGRAM.md](../what/concepts/EXECUTION_PROGRAM.md) — the Execution Program model
