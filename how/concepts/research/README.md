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

**Note (2026-07-26):** `essential/` is not exclusively "Phase 4, first pass."
Roughly half of its files (~20 of ~42) define one of the six Semantic Model
dimensions or a primitive construct/composition rule and feed Phase 2 or Phase 3
directly instead of Phase 4; a further 3 are Policy-level material pending
relocation out of this pipeline entirely; a further 2 were moved into this tier
mid-cycle and still await a Phase 2/3/4 classification call. See
`.planning/notes/2026-07-26-tier-vs-phase-mapping.md` for the split.
`important/` and `deferrable/` remain entirely Phase 4 material as described
above.

The `research/` root itself contains only meta-files that are not feature
proposals:
- `README.md` — this file
- `imperative-crutch-*.md` — anti-pattern analysis (informs design, not a feature)
- `imperative-crutches-index.md` — index of anti-pattern analysis
- `language-llm-comparison.md` — language comparison reference

> **Note (2026-08-15):** Syntax hypotheses previously kept here (for example
> `TYPE_ANNOTATION_SYNTAX.md`, `SHADOWING_SYNTAX.md`) and in `important/`
> (`FUNCTION_ARGUMENT_SYNTAX.md`, `FUNCTION_RETURN_SYNTAX.md`,
> `FUNCTION_RETURNING_FUNCTION.md`, `BOUNDS_IN_ANGLE_BRACKETS.md`) moved to
> [`../syntax/`](../syntax/) — the syntax hypothesis inbox (EDR-087).

> **Note (2026-08-17):** `EFFECT_FOOTPRINT.md` added to `essential/` — the
> unified effect-footprint model (read/mutate/consume × {self, capture,
> global}) unifying the declaration kinds, `using`, and `@modifies`.
>
> **Note (2026-08-17):** `TYPE_ALIAS.md` added to `important/` — type synonym
> hypothesis (same type, new name; syntactic sugar). Scopes `alias` to type
> synonyms only, excluding import aliasing (`USING_DIRECTIVES.md`) and
> refinement types (`REFINEMENT_TYPES.md`). See
> `../../../syntax/FUNCTION_RETURNING_FUNCTION.md` for the function-type
> synonym coupling.
>
> **Note (2026-08-23):** `SYSTEM_ERROR.md` added to `essential/` — failure
> taxonomy hypothesis: the domain/system boundary (representable-as-value vs
> defined-termination) and the SYSTEM_ERROR family classified by fault owner
> (PROGRAM_ERROR / EXECUTION_ERROR / PLATFORM_ERROR), with COMPILE_ERROR as
> a separate phase and LANGUAGE_ERROR as a forbidden meta-state. Allocation
> failure is policy-dependent: recoverable domain error under
> `Arena`/`Static`, terminal PLATFORM_ERROR under `Heap`/GC.
>
> **Note (2026-08-23):** `SYSMEM_ERROR.md` folded into `SYSTEM_ERROR.md` —
> allocation failure is not a separate class. Recoverable exhaustion
> (`Arena`/`Static`) is a domain error; terminal exhaustion (`Heap`/GC) is
> **PLATFORM_ERROR**. The classification rule and open questions now live in
> `SYSTEM_ERROR.md`; a dedicated MEMORY_ERROR class is deliberately not
> introduced.
>
> **Note (2026-08-23):** `SYSTEM_ERROR.md` **graduated** — accepted as an
> Orthon concept via [EDR-089](../../decision_records/architecture/EDR-089-system-error-taxonomy.md)
> (Concept Design Review, 2026-08-23). The accepted concept lives at
> [`what/concepts/SYSTEM_ERROR.md`](../../../what/concepts/SYSTEM_ERROR.md);
> this research file is retained as provenance.

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
