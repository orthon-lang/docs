# Logical Analysis

> Derive the consequences of a proposal from explicit premises and
> verify they are consistent with the architecture. A concept that
> requires hidden premises to justify its place in the architecture
> is not yet ready.

---

## Origin

Rooted in deductive reasoning and formal logic (Aristotle, Frege).
Logical analysis tests whether conclusions follow necessarily from
premises — and whether those premises are themselves valid.

---

## Application to Language Design

When evaluating architectural fit, Logical Analysis tests whether
the proposal's implications are compatible with the project's
architectural rules.

### 1. State the Premises

Identify the architectural rules the proposal explicitly depends on:

- "Layers are independent."
- "Core Language has no Standard Library dependencies."
- "Each construct has exactly one syntactic form."
- (Add specific rules from `../../architecture/ARCHITECTURE.md`.)

### 2. Deduce the Consequences

If the proposal is accepted, what must be true?

- What must the compiler do differently?
- What must the runtime support?
- What must the programmer understand?
- Which existing documents would need to change?

### 3. Test for Contradiction

Do any deduced consequences contradict the stated premises?

- "The proposal requires the Standard Library to access compiler
  internals — but premise 1 says layers are independent.
  Contradiction."

### 4. Identify Hidden Premises

What unstated assumptions does the deduction require?

- "The proposal assumes type information is available at runtime —
  but this has never been stated as a design constraint."
- "The proposal assumes all strategies support garbage collection."

Hidden premises must either be explicitly accepted (and documented)
or the proposal must be revised to remove the dependency.

---

## Perspective

This method serves the **ARCHITECTURAL_INTEGRITY_GATE** — the gate
that asks whether the proposal fits within Orthon's layered
architecture and composes freely with existing constructs.

**Used by:** `ARCHITECTURAL_INTEGRITY_GATE`
