# Concept Research Inbox

This directory contains raw research and analysis of language concepts
from various programming languages. Files here are **input material**,
not Orthon specification.

## Purpose

- Analyse how other languages solve specific problems
- Identify patterns, anti-patterns, and implications for Orthon
- Provide evidence for design decisions during Concept Design Review

## Contents

The directory includes two kinds of research:

- **Orthon-proposed concept analysis** — files named `CONCEPT_NAME.md`
  (e.g., `DATA_MODEL.md`, `FUNCTIONS.md`, `OWNERSHIP.md`). These are
  draft concept analyses from Milestone 0, awaiting formal review.
- **Problem-domain research** — files named `imperative-crutch-TOPIC.md`.
  These analyse imperative anti-patterns from Python/Java to inform
  Orthon's declarative design.

## Research Format

Each research file should include:

1. **Problem** — what problem does this concept solve?
2. **Examples** — how do other languages (Python, Rust, Java, etc.) solve it?
3. **Implications for Orthon** — what does this mean for Orthon's design?
4. **Open Questions** — what needs further investigation?

## Graduation

When a concept passes Concept Design Review:

1. Create an EDR in `how/decision_records/architecture/`
2. Create an Orthon-specific concept draft in `what/concepts/`
3. Add an entry to `what/CORE_CONCEPTS.md`
