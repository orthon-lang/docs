# Core Inclusion Filter

> **Last updated:** 2026-07-21

> A procedural decision rule for determining whether a new language
> feature belongs in the Core Language or should be expressed at a
> higher layer of the Semantic Dependency Architecture.
>
> Every new abstraction must first prove that it cannot be expressed
> as a composition of existing primitives. Only then may it become
> part of the language core.

---

## Problem

The most common source of language bloat is adding a new construct to
the core when it could have been a library function or a composition
of existing patterns. Each such addition increases the learning
surface, the specification surface, and the implementation burden
— permanently.

The Core Inclusion Filter provides a **procedural decision rule** that
must be applied *before* a proposal reaches the formal Language Design
Gate. It answers the question: *at which layer of the Semantic
Dependency Architecture does this feature belong?*

---

## Filter Rule

```
Can it be expressed as a Standard Library function?
        │
      Yes ───────────────► Level 3 (Standard Library)
        │                  No language change required.
       No
        │
Can it be expressed as a Language Pattern
(composition of existing primitives)?
        │
      Yes ───────────────► Level 2 (Language Pattern)
        │                  Composition formula must be documented.
       No
        │
Is it a new Primitive Operation on existing Data Model concepts?
        │
      Yes ───────────────► Level 1 (Primitive Operation)
        │                  Requires Architecture-category EDR
        │                  justifying why decomposition is impossible.
       No
        │
        └────────────────► Level 0 (Data Model)
                           Requires Architecture-category EDR.
                           Changes the semantic foundation —
                           highest bar in the language.
```

### Decision Procedure

For any proposed language feature, follow these steps in order:

1. **Library test** — Can the feature be implemented as a Standard
   Library function using only existing Core Language constructs and
   other Standard Library functions? If yes, it belongs in the
   Standard Library. No language change is needed.

2. **Pattern test** — Can the feature be expressed as a composition
   of existing Primitive Operations (Level 1) and Data Model (Level 0)
   constructs? If yes, it belongs in Language Patterns (Level 2).
   The composition formula must be documented — show *which* primitives
   compose to produce the pattern.

3. **Primitive test** — Is the feature an atomic operation on data
   that cannot be decomposed into simpler operations? If yes, it may
   be a new Primitive Operation (Level 1). An Architecture-category
   EDR is required, with explicit justification of why the operation
   cannot be decomposed.

4. **Data Model test** — Does the feature change what exists as a
   semantic primitive in the language? If yes, it modifies the
   Data Model (Level 0). This has the highest bar in the language:
   an Architecture-category EDR is required, and the proposal must
   demonstrate that no lower level can express the concept.

---

## Examples

### Example 1: A `retry()` utility

```
Proposal: Add retry(n, fn) that retries fn up to n times on failure.

Library test:   Can be implemented as a higher-order function using
                existing Core constructs (functions, loops, exceptions).
                → PASS — belongs in Standard Library (Level 3).
```

### Example 2: A `context` manager pattern

```
Proposal: Add with-scope syntax for setup/teardown.

Library test:   Requires syntactic structure beyond a function call.
                → FAIL (cannot be a library).

Pattern test:   Can be expressed as a composition of existing primitives:
                functions + closures + exceptions + scope + metadata.
                The syntactic sugar is convenience, not new semantics.
                → PASS — belongs in Language Patterns (Level 2).
```

### Example 3: A `try/except` construct

```
Proposal: Add exception handling.

Library test:   Requires fundamental control flow changes — cannot be
                implemented as a library function.
                → FAIL.

Pattern test:   Cannot be composed from existing primitives — no
                existing primitive provides non-local control transfer
                on error.
                → FAIL.

Primitive test: Exception handling is an atomic operation: it
                introduces a new control flow mechanism (non-local
                exit through the call stack) that cannot be decomposed.
                → PASS — belongs in Primitive Operations (Level 1).
```

---

## Relationship to the Validation Gates

The Core Inclusion Filter is a **pre-gate procedure**: it should be
applied before the feature enters formal validation through the
Decision Validation gates (`DECISION_VALIDATION.md`).

| Gate | Relationship |
|------|-------------|
| `CONCEPTUAL_SIMPLICITY_GATE` | The Library test and Pattern test directly implement this gate's "Expressible through composition" criterion. Passing both tests means the concept fails this gate. |
| `ARCHITECTURAL_INTEGRITY_GATE` | The filter's layer assignment ensures the proposal fits within the Semantic Dependency Architecture. |
| `USER_VALUE_GATE` | Even if the filter assigns a feature to Level 3 (Library), the User Value gate still applies — a library feature must earn its place too. |

> **Note:** The filter assigns a feature to a layer, but all layers
> proceed through the same validation gates. This is inconsistent:
> Level 0 (Data Model) and Level 3 (Standard Library) have
> fundamentally different bars, yet share the same procedure.
> A future refinement should define layer-specific validation
> procedures that branch after the filter's decision tree.

See also:
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) § Semantic Dependency Architecture — the 6-level pyramid
- [`DESIGN_PRINCIPLES.md`](../DESIGN_PRINCIPLES.md) § Minimal Core — Core Change Criterion
- [`_language-design.md`](../gates/_language-design.md) — formal gate criteria (Minimality, Core stability, Semantic layer classification)
- [`DECISION_VALIDATION.md`](../gates/DECISION_VALIDATION.md) — full validation gate catalogue
