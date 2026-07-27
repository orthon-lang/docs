# EDR-075: Reject Prototype-Based Object Model

**Status:** Rejected

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Project

---

### Context

The Prototype concept (from `how/concepts/research/deferrable/PROTOTYPE.md`) proposes supporting a prototype-based object model as a complement to Orthon's class/trait-based composition model. In prototype-based languages (JavaScript, Self, Lua), objects are created as clones of existing objects (their **prototype**), and property/method lookups that fail on the object itself are delegated up the prototype chain at runtime.

The concept was researched and placed in the deferrable tier during Phase 1 triage (SEED-001), flagged as a candidate for outright rejection because it contradicts Orthon's stated design principles.

### Decision

**Formal Rejection.** Prototype-based object model is rejected as a language feature for Orthon v0.1. It will not be deferred — it is structurally incompatible with Orthon's core design principles and will not appear in any future version of the language specification.

### Rationale

**1. Data First principle violation.** Orthon's `Data First` principle states that data is the primary abstraction and language constructs operate by transforming data into different representations. Prototypes conflate data and behaviour — an object is simultaneously a data container and a behaviour delegation chain. This violates the fundamental separation that Data First requires: data is separate from the operations that transform it.

In a prototype system, `obj.foo` could be:
- A data property directly on `obj`
- A data property inherited from a prototype
- A method inherited from a prototype
- A getter that executes code

The receiver cannot distinguish data from behaviour at the call site. This directly contradicts Data First.

**2. Explicitness violation.** Orthon's `Explicitness` principle requires that behaviour-changing operations be visible in the syntax. Prototype delegation is implicit — the programmer writes `obj.foo` and the runtime walks the prototype chain silently. The delegation is invisible in the surface syntax. The programmer cannot tell from reading the code which object provides `foo`.

Dynamic dispatch by default (common in prototype languages) further violates Explicitness — the method resolved at runtime depends on the prototype chain's current state, which may have been mutated elsewhere.

**3. Orthogonality violation.** Prototypal inheritance conflates composition and delegation into a single mechanism. The prototype chain simultaneously serves as:
- Object composition (what data an object has)
- Behaviour sharing (what methods an object supports)
- Dynamic dispatch (which implementation runs at call time)

This violates Orthogonality's requirement that each construct solve exactly one problem. Three concerns (data membership, behaviour reuse, dispatch) are merged into one mechanism.

**4. Minimal Core violation.** Prototype delegation can be expressed as a library pattern using Orthon's existing trait and delegation mechanisms — a `Delegator<T>` type that forwards method calls to a dynamically-assigned delegate object (as noted in the research document itself). Adding prototypes to the core language when they are expressible via composition violates Minimal Core.

### Consequences

- **Positive:** Core language remains clean; no prototype-specific syntax, no prototype chain walking in the compiler, no inline cache requirements for the implementation strategy.
- **Positive:** Forces the programmer to use Orthon's explicit composition mechanisms (traits, delegation, extension functions) which are more analysable and optimisable.
- **Negative:** Programmers coming from JavaScript or Lua will not find a direct analogue and must adapt to Orthon's trait/composition model.
- **Negative:** Some dynamic delegation patterns (e.g., JavaScript's `Object.create(null)` for dictionary objects) require more explicit ceremony.

### Compliance

Every future concept design review must check for prototype-like semantics. Any proposal that introduces implicit delegation chains or runtime-mutable behaviour resolution must be rejected unless accompanied by a Tier 1 EDR changing the Data First, Explicitness, or Orthogonality principles (which would require a fundamental re-evaluation of Orthon's design identity).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Defer to v0.2 | Rejected — prototypes are structurally incompatible, not merely incomplete. Deferring implies they might be acceptable later, which they are not. |
| Limited form with static types | Rejected — a statically-typed prototype chain is indistinguishable from trait delegation. The "prototype" name would add confusion without benefit. |
| Library pattern (Delegator\<T\>) | Accepted as the correct approach — prototypes belong in userland, not the language core. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | **Fail** | Prototypes conflate data and behaviour, violating Data First. The concept is internally consistent (prototypes work as described) but inconsistent with Orthon's semantic framework. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | **Fail** | Prototype chains introduce implicit delegation that degrades the programmer's ability to reason about code. The model is not simpler than explicit composition. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | **Fail** | Prototypal inheritance violates architectural integrity by merging composition, delegation, and dispatch into a single mechanism. These must remain separate per Orthon's layered architecture. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | **Fail** | Runtime-mutable prototype chains prevent static analysis and degrade compiler optimisation opportunities. The maintainability cost over the language's lifetime is unacceptable. |

**Gates not applied:** `USER_VALUE_GATE`, `IMPLEMENTATION_INDEPENDENCE_GATE`, `LLM_GENERABILITY_GATE` — as a rejected concept, user value, implementation independence, and LLM generability are not relevant. The concept fails the four structural gates and is rejected on principle grounds.
