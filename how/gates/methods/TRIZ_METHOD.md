# TRIZ — Theory of Inventive Problem Solving

> When a proposal seems to require a specific implementation strategy,
> TRIZ helps find the abstract solution that works across all
> strategies without compromise. If the only solutions are
> strategy-specific workarounds, the concept is not yet
> implementation-independent.

---

## Origin

Developed by Genrich Altshuller and colleagues starting in 1946. TRIZ
is based on the observation that technical problems and their
solutions repeat across industries — and that contradictions can be
resolved systematically by applying known inventive principles rather
than by compromise.

---

## Application to Language Design

When evaluating implementation independence, TRIZ tests whether a
concept can be defined purely semantically — without leaking
strategy-specific details.

### 1. Identify the Apparent Contradiction

> "To implement [feature], the runtime must [specific behaviour],
> which contradicts [strategy X's constraints]."

Example: "To implement reference counting, the runtime must track
ownership at runtime — but the Embedded strategy has no runtime
overhead budget."

### 2. Apply Separation Principles

Resolve the contradiction by separating concerns:

- **Separation in time** — The concept behaves one way during
  compilation, another at runtime.
- **Separation in space** — Different parts of the concept live in
  different architecture layers.
- **Separation on condition** — The concept's definition does not
  change, but each strategy implements it differently.
- **Separation by scale** — The concept is defined at the semantic
  level; strategy details are below that level.

### 3. Consult Known Patterns

Has a similar "must work across strategies" problem been solved
elsewhere?

- How does Orthon's existing Data Modifier system handle this?
- How do other languages with multiple back-ends solve this?
- Does the principle of "Semantics Before Optimization" apply here?

### 4. Formulate the Abstract Solution

Define the concept purely in terms of what it does:

> "[Concept] transforms [input] into [output] with [semantic
> guarantee]. Each strategy realises this transformation according
> to its own constraints."

If the abstract formulation is empty or requires strategy-specific
language, the concept fails this gate.

---

## Perspective

This method serves the **IMPLEMENTATION_INDEPENDENCE_GATE** — the
gate that asks whether a concept can be defined semantically without
tying it to a specific implementation strategy. It enforces Orthon's
separation of *what* from *how*.

**Used by:** `IMPLEMENTATION_INDEPENDENCE_GATE`
