# Scientific Method

> Treat every claim about a language concept as a hypothesis to be
> tested. Separate what is known from what is assumed, and let
> evidence — not intuition — determine whether the concept is as
> simple and minimal as it claims to be.

---

## Origin

The systematic process of observation, hypothesis formation,
experimentation, and conclusion that underpins modern science.
Formalised by Francis Bacon, Galileo, and others; applied here to
design rather than nature.

---

## Application to Language Design

When evaluating a proposal's simplicity, the Scientific Method
disciplines the investigation: it forces the evaluator to distinguish
between established facts and unverified assumptions.

### 1. State the Hypothesis

Formulate the claim as a testable statement:

> "This concept is the simplest possible way to achieve [goal]."
> "No combination of existing constructs can express [behaviour]
> without this concept."

### 2. List What Is Known

Inventory existing concepts, modifiers, and patterns that already
cover parts of the problem space:

- Which constructs are already defined?
- What can they express in combination?
- Which aspects of the proposal are already achievable?

### 3. List What Is Unknown

Identify assumptions in the proposal:

- What programmer needs are assumed but not verified?
- Which interactions with existing constructs are assumed to "just
  work" but not documented?
- Are there edge cases where the simplicity claim might break?

### 4. Design the Simplest Experiment

What is the minimum Orthon code that would validate or falsify the
hypothesis? Write it. If the experiment is longer or more complex
than the proposal itself, the hypothesis is not testable.

### 5. Evaluate the Evidence

- Does the experiment confirm the simplicity claim?
- Does it reveal hidden complexity?
- Can the same result be achieved through composition of existing
  constructs? If yes, the concept fails the gate.

---

## Perspective

This method serves the **CONCEPTUAL_SIMPLICITY_GATE** — the gate
that asks whether a concept is as simple as it can be, or whether it
could be expressed through composition of existing concepts. It
enforces Orthon's commitment to a minimal core.

**Used by:** `CONCEPTUAL_SIMPLICITY_GATE`
