# Orthogonality: The Pain It Removes

> Exploratory note. Answers: what pain does orthogonality close?

## Thesis

Orthogonality is the discipline that prevents a language from growing
by accretion. Each pain below is a distinct way that non-orthogonal
languages tax their users.

## Pain 1 — Special cases multiply cognitive load

Every special case, exception, and context-dependent rule must be
learned, remembered, and applied correctly on every encounter.

- Non-orthogonal: "this operator means X here, but Y next to a colon."
- Orthogonal: "what you learn in one part transfers to every other
  part" (`why/ZEN.md`).

Orthogonality removes exceptions — the largest source of surprise when
learning a language.

## Pain 2 — Memorization replaces understanding

When rules do not transfer, users stop reasoning and start memorizing.
Knowledge becomes brittle: a remembered rule fails exactly where the
language is inconsistent.

## Pain 3 — Features cannot compose

Overlapping features cannot combine freely; every combination needs a
bridge, a special case, or glue code. Orthogonality guarantees
composition: complex behavior emerges from combining simple, orthogonal
building blocks (`how/DESIGN_PRINCIPLES.md` § Orthogonality).

## Pain 4 — "Which form should I use?" ambiguity

Overlapping forms create confusion about which form to use
(`how/DESIGN_PRINCIPLES.md`). The user must choose between
near-equivalents instead of expressing intent.

## Pain 5 — Prediction-space growth for LLMs

Every special case and context-dependent rule expands the prediction
space for an LLM and multiplies generated errors (`why/VISION.md` § The
Pain of This Era, Pain 3). Orthogonality keeps the surface minimal and
the rules consistent, so an LLM learns the language once and applies it
everywhere.

## Pain 6 — Decay by accretion

Languages that are not kept orthogonal accumulate features over decades
(`why/VISION.md` § The Pain of This Era, Pain 1). Orthogonality is the
anti-accretion discipline: each construct must earn its place and
combine freely, or it is not admitted to the core.

## What orthogonality does NOT claim

Orthogonality does not reduce expressiveness; it relocates complexity
into the compiler and the Implementation Strategy (the Simplicity
principle, `how/DESIGN_PRINCIPLES.md`).

## Sources

- `why/ZEN.md` — "Orthogonality removes exceptions", "Simplicity is the
  result of orthogonality"
- `how/DESIGN_PRINCIPLES.md` § Orthogonality, § Simplicity
- `what/GLOSSARY.md` § Orthogonality
- `why/VISION.md` § Comfortable by Design, § The Pain of This Era
