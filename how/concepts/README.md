# Concept Design Pipeline

This directory contains the design process for Orthon language concepts.

## Structure

- `research/` — Research inbox. Raw analysis of concepts from various
  programming languages. Files here are input material, not Orthon specification.
  Concepts are triaged by importance into tier subdirectories
  (`essential/`, `important/`, `deferrable/`, `reject/`).
  See [`research/README.md`](research/README.md) for details.
- (future) `designs/` — Active concept design documents in review.

## Pipeline

See [`CONCEPT_PIPELINE.md`](../CONCEPT_PIPELINE.md) for the full 11-stage lifecycle — research inbox, decision pipeline, primitive decomposition, layer classification, concept design review, validation gates, language design gate, EDR, accepted draft, cross-cutting review, and registry entry.

## Adding Research

To research a new concept:

1. Determine its tier (essential / important / deferrable) based on how
   foundational the concept is to Orthon's semantic identity
2. Create a file in `research/{tier}/` following the research naming convention
3. Include: problem statement, examples from other languages, implications for Orthon
4. Reference [`_concept.md`](../templates/_concept.md) for the eventual design stage

**Important:** Do not place new concept files directly into `research/` root.
They must go into the appropriate tier subdirectory.

## Graduation

A concept graduates from research to acceptance when it passes Concept Design
Review and receives an EDR (Architecture category). At that point:

- A refined Orthon-specific draft goes to `what/concepts/`
- An entry is added to `what/CORE_CONCEPTS.md`
