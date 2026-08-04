# Emergence as a Criterion of Architectural Adequacy

> Exploratory note. Answers: how does "emergence as a criterion of
> architectural adequacy" apply to Orthon, and where is it already
> (and not yet) captured?

## Thesis

An architecture is adequate when its desired system-level properties
*emerge* from the composition of simpler, orthogonal parts — rather
than being explicitly added as special cases. If a property cannot
emerge, it must be engineered in; every engineered-in property is
evidence of architectural inadequacy.

Applied to a language: complex capabilities should be *compositions*
of primitive operations, not bespoke features with their own rules.

## Orthon already embodies this

- `why/MANIFESTO.md` — "Composition over exceptions: new capabilities
  should emerge by composing existing mechanisms instead of adding
  special cases."
- `how/DESIGN_PRINCIPLES.md` § Orthogonality — "Complex behavior should
  emerge from combining simple, orthogonal building blocks."
- `how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture —
  6-level pyramid; Level 2 Language Patterns are "compositions of
  primitive operations that add convenience but no new semantics."
- `what/PRIMITIVE_BLOCKS.md` § Purpose & Scope — the adequacy test is
  stated explicitly: "A concept that cannot decompose onto this set
  signals an incomplete primitive set; a primitive that overlaps
  another signals a non-orthogonal set."

## Where the criterion is NOT yet explicit

Emergence is currently scattered across documents as a *property* of
good design, but it is not named as a *criterion of adequacy*:

1. **No fitness function.** `how/architecture/FITNESS_FUNCTIONS.md`
   has "Composition Surface" and "Orthogonality", but no check that
   measures decomposability of a new concept onto Primitive Blocks.
2. **No gate criterion.** `how/gates/_language-design.md` asks
   "Orthogonality" and "Minimality", but does not frame the
   decompose-or-fail test as an emergence check.
3. **No glossary term.** "Emergence" is not defined in
   `what/GLOSSARY.md`.
4. **No aphorism.** `why/ZEN.md` has "Simplicity is the result of
   orthogonality" but nothing about emergence as the adequacy test.

## Recommended capture points

| Document | What to add |
|----------|-------------|
| `how/architecture/FITNESS_FUNCTIONS.md` | New fitness function "Emergence / Decomposability" — every new concept must decompose onto Primitive Blocks (Level 1) or justify why it cannot |
| `how/gates/_language-design.md` | Criterion: "Does the desired behavior emerge from composition, or is it engineered in as a special case?" |
| `what/GLOSSARY.md` | Term "Emergence" with definition + cross-reference |
| `why/ZEN.md` | Optional aphorism, e.g. "What must be added is what failed to emerge." |

## The criterion as a check

Given a proposed capability C:

1. **Decompose** — does C decompose onto the Primitive Blocks
   (Level 1)? If no, either C is architecturally inadequate, or the
   primitive set is incomplete (fix the primitive set, not the
   feature).
2. **Overlap** — does C overlap an existing concept? If yes, C is not
   orthogonal; composition would already express it.
3. **Special case** — does C require a special case anywhere (syntax,
   semantics, tooling)? If yes, C is not emergent; it is accretion.

## Why it matters for the LLM audience

An emergent language has a small rule surface. An LLM learns the
primitives once and can generate arbitrary compositions correctly — no
feature-specific training required. Engineered-in special cases expand
the prediction space and multiply generated errors (`why/VISION.md`
§ The Pain of This Era, Pain 3).

## Sources

- `why/MANIFESTO.md` § Composition over exceptions
- `how/DESIGN_PRINCIPLES.md` § Orthogonality, § Extensibility over Built-in Magic
- `how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- `how/architecture/FITNESS_FUNCTIONS.md` § Orthogonality, § Composition Surface
- `what/PRIMITIVE_BLOCKS.md` § Purpose & Scope
- `what/GLOSSARY.md` § Minimal Core
- `notes/orthogonality-pain.md`
