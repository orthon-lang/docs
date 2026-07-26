# Concept Research Inbox

This directory contains raw research and analysis of language concepts
from various programming languages. Files here are **input material**,
not Orthon specification.

## Directory Structure

Research files are triaged by importance into tier directories.
A new concept must always be placed into the appropriate tier directory,
never directly into `research/` root.

| Directory | Priority | Description | Decision Pipeline |
|-----------|----------|-------------|-------------------|
| `essential/` | Must-have | Semantic bedrock — the language's skeleton. Without these, Orthon cannot exist. | Phase 4, first pass |
| `important/` | Important | Makes the language usable and expressive — the language's muscles. | Phase 4, second pass |
| `deferrable/` | Nice-to-have | Sugar, domains, tooling, or features deferrable to v0.2/v0.3 — the language's accessories. | Phase 4, deferred |
| `reject/` | Contradicts principles | Concepts that contradict Orthon's stated principles; candidates for formal rejection via EDR. | Phase 4, rejection decision |

The `research/` root itself contains only meta-files that are not feature
proposals:
- `README.md` — this file
- `imperative-crutch-*.md` — anti-pattern analysis (informs design, not a feature)
- `imperative-crutches-index.md` — index of anti-pattern analysis
- `language-llm-comparison.md` — language comparison reference

## Adding Research

To add a new concept research document:

1. Determine its tier (essential / important / deferrable) based on how
   foundational the concept is to Orthon's semantic identity.
2. Place it in the corresponding subdirectory.
3. Follow the research format below.
4. Reference [`_concept.md`](../../templates/_concept.md) for the eventual design stage.

**Rule:** Do NOT place new concept files directly into `research/` root.
They must go into a tier directory.

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
