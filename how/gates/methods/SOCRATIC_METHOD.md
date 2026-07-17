# Socratic Method

> Disciplined questioning to expose hidden contradictions, imprecise
> definitions, and unstated assumptions. A concept that survives
> Socratic interrogation without introducing special cases is
> internally sound.

---

## Origin

Attributed to Socrates as recorded in Plato's dialogues. The method
proceeds by asking a series of questions to expose the limits of a
definition or argument. It does not assume the answer — it tests the
reasoning.

---

## Application to Language Design

When evaluating a language proposal, the Socratic Method serves as a
logical-consistency gate: it forces every term and rule to be
precisely defined and tests whether they hold under scrutiny.

### 1. Define All Terms

For every concept the proposal introduces, ask:

- "What exactly does *this* mean?"
- "Is this term defined in the project Glossary?"
- "Does this definition conflict with any existing definition?"

### 2. Test with Counterexamples

Take the proposal's rules and apply them to edge cases:

- "If this rule holds, what happens when we apply it to [edge case]?"
- "Is the outcome consistent with how other constructs behave in the
  same situation?"

### 3. Follow the Contradiction

When two criteria seem to conflict, do not resolve by intuition:

- "Which principle gives way?"
- "Is the exception principled (derived from a higher rule) or
  arbitrary (introduced just to patch this case)?"
- "Would the same reasoning apply if roles were reversed?"

### 4. Play Devil's Advocate

Construct the strongest argument against the proposal using only its
own definitions. If the counterargument reveals a flaw, the proposal
must be revised — not dismissed.

---

## Perspective

This method serves the **LOGICAL_CONSISTENCY_GATE** — the gate that
asks whether a proposal is internally consistent and free of
contradictions. A concept that cannot be precisely defined cannot be
consistently implemented.

**Used by:** `LOGICAL_CONSISTENCY_GATE`
