# Concept Design Pipeline

This directory contains the design process for Orthon language concepts.

## Structure

- `research/` — Research inbox. Raw analysis of concepts from various
  programming languages. Files here are input material, not Orthon specification.
- (future) `designs/` — Active concept design documents in review.

## Pipeline

1. **Research** → file in `research/` documents analysis from other languages
2. **Decision Pipeline** → 10-question pre-filter (see [`DECISION_PIPELINE.md`](../process/DECISION_PIPELINE.md))
3. **Primitive Decomposition** → verify concept decomposes to Primitive Blocks
4. **Concept Design Review** → formal review process (see [`concept-design-review.md`](../concept-design-review.md))
5. **Acceptance** → EDR (Architecture category) + draft in `what/concepts/`
6. **Specification** → entry in `what/CORE_CONCEPTS.md`

## Adding Research

To research a new concept:

1. Create a file in `research/` following the research naming convention
2. Include: problem statement, examples from other languages, implications for Orthon
3. Reference [`_concept.md`](../templates/_concept.md) for the eventual design stage

## Graduation

A concept graduates from research to acceptance when it passes Concept Design
Review and receives an EDR (Architecture category). At that point:

- A refined Orthon-specific draft goes to `what/concepts/`
- An entry is added to `what/CORE_CONCEPTS.md`
