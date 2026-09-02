---
date: "2026-09-02 00:00"
promoted: false
---

# Quantitative Type Theory and Orthon

## The question

How does Quantitative Type Theory (QTT) relate to Orthon? Is it
applicable, does it make sense, and is QTT part of Type-Driven
Development?

## Findings

### QTT is a formal counterpart, not a language feature

QTT (Atkey 2018; McBride, "I Got Plenty o' Nuttin'", 2016) assigns a
multiplicity to every variable use: `0` (erased), `1` (exactly once —
linear), `ω` (unrestricted). It is the unified formalism behind
substructural typing — affine types (Rust), linear types, and the core of
Idris 2.

Orthon already states the semantic content that QTT's `1`-multiplicity
expresses, but deliberately does not adopt QTT as the enforcement
mechanism:

- `what/SEMANTIC_MODEL.md` § Ownership: "every value has exactly one
  owner" (Semantic Invariant 1), shared XOR mutable (Invariant 2),
  explicit transfer (Invariant 6). The formal foundation named there is
  **Separation Logic** (Reynolds 2002, O'Hearn 2019), not QTT.
- "Ownership is a semantic invariant, not an enforcement mechanism" — the
  borrow checker, escape analysis, and other mechanisms are all
  "compatible implementations of the same semantic contract." QTT-based
  typing would be one more.
- EDR-061 (`COPY_ON_WRITE.md`) is the accepted memory model: "No borrow
  checker — Safety comes from CoW semantics, not from linear type
  analysis." Default is value semantics + CoW, with `shared` for explicit
  reference-counted sharing.

**Conclusion:** in Orthon's frame, QTT is a Strategy-level enforcement
option (Lifetime/Ownership Policy), not a Core-Language concept. The repo
already draws this line for `REGION_BASED_MEMORY_MANAGEMENT.md`
(`.planning/todos/pending/move-policy-level-essential-concepts-out-of-pipeline.md`).

### Applicability — conditional

Applicable only as a strategy-scoped mechanism (e.g. the
`HIGH_PERFORMANCE_STRATEGY` Lifetime Policy for zero-overhead compile-time
safety). The hard constraint is the **LLM Generability Gate** (EDR-014):
linear/affine typing is exactly what LLMs generate unreliably (Rust borrow
errors), which conflicts with Orthon's LLM-native positioning
(`notes/llm-native-tool-strategy.md`).

### Does it make sense?

- As core language: no — violates minimal core, comfort-by-construction
  (`why/WORKING_BACKWARDS.md` calls the borrow checker "a notoriously
  difficult mental model"), and the Generability Gate. Also redundant,
  since the semantics are already captured.
- As a formal model / soundness argument: yes — a QTT/graded-monoid model
  of the single-owner invariant could strengthen the specification without
  changing the language.
- As a Strategy option: yes — alongside CoW in `how/strategies/`.

### QTT and Type-Driven Development

Partially, at different levels. Type-Driven Development (Brady, Idris) is
a *methodology*; QTT is a *type-system formalism*. They meet in Idris 2,
whose core is built on QTT — the current state of the art of Type-Driven
Development runs on QTT. QTT is not itself TDD.

### The subtle point (open)

Does the semantics/mechanism boundary itself undermine Orthon's LLM bet?
If an LLM is not required to prove `1`-multiplicity at generation time,
where is an ownership violation caught at all — and does the default CoW
strategy eliminate the error class entirely (plain data simply copies;
only the resource subset is governed by ownership), making Orthon's
answer *stronger* than QTT-level typing rather than weaker? Tracked as a
research question (`.planning/research/questions.md` § QTT boundary).
