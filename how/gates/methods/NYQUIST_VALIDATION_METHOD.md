# Nyquist Validation

> Audit whether the selected checks and observations would reveal a
> significant violation of a stated semantic invariant.

---

## Origin and Limits

The name borrows an analogy from the Nyquist-Shannon sampling theorem:
an observation can miss a meaningful change. Software and language
semantics have no universal sampling frequency, so this method claims
neither a numerical threshold nor complete defect detection.

This is an Orthon design-review method, not GSD's similarly named
requirement-to-automated-test coverage audit. Orthon currently has a
documentation-only specification; a documented scenario is design
evidence, not a passing executable test.

---

## Application to Language Design

Use this method after the Decision Pipeline has framed a problem and
while preparing or reviewing gate evidence. Prioritise invariants that
cross concept boundaries, depend on event order, or must hold across
Implementation Strategies. It audits the sufficiency of existing
gates, examples, interaction analyses, and fitness functions; it adds
no validation gate or language semantics.

### 1. State the Invariant

Quote the canonical semantic rule and link its source. State the
observable behaviour that must remain unchanged.

### 2. Enumerate Meaningful Violations

List concrete counter-scenarios, including interactions with other
concepts, different event orders, and strategy changes when relevant.
Prioritise scenarios by the consequence of a missed violation.

### 3. Identify Discriminating Evidence

For each scenario, name the observation that would distinguish a valid
outcome from a violating one: a specified result, a required diagnostic,
an interaction-matrix entry, a gate reasoning trail, or a future
executable check. An example that merely exercises the construct is
insufficient if both outcomes would look the same in that example.

### 4. Compare and Record the Coverage Limit

Record the expected outcome, the available evidence, and the verdict
for each scenario: **covered**, **uncovered**, or **deferred**. State
which scenarios were examined. An uncovered significant violation
requires a stronger scenario or observation before claiming that the
relevant gate evidence is sufficient. A covered scenario does not
prove the invariant for all programs.

### Example: Strategy-Independent Behaviour

The [Deterministic Behavior fitness function](../../architecture/FITNESS_FUNCTIONS.md#deterministic-behavior)
requires the same source program to have the same observable behaviour
across Implementation Strategies. Suppose a value copy is implemented
eagerly in one strategy and with copy-on-write in another. An example
that checks only the copied value immediately after assignment cannot
reveal a later aliasing error. A discriminating scenario mutates the
original after the copy and compares both resulting values under each
strategy. The expected result is the same value semantics in both
cases. At the present design stage this is an illustrative semantic
scenario; it is not evidence that a compiler test has passed.

---

## Perspective

Nyquist Validation asks whether significant semantic violations would
be *detectable by the chosen evidence*. It complements the seven
[Decision Validation gates](../DECISION_VALIDATION.md) and the
[fitness functions](../../architecture/FITNESS_FUNCTIONS.md).

**Used by:** Concept Design Review validation and cross-cutting review.
