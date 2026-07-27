# EDR-077: Reject Dynamic Typing

**Status:** Rejected

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Project

---

### Context

The Dynamic Typing concept (from `how/concepts/research/deferrable/DYNAMIC_TYPING.md`) proposes providing an escape hatch for dynamic runtime behaviour within Orthon's primarily static type system — analogous to C#'s `dynamic` keyword, TypeScript's `any`, or Java's reflection + `Object`. The research document evaluates how a `dynamic`-like type would bypass compile-time type checking and resolve member access at runtime.

The concept was placed in the deferrable tier during Phase 1 triage (SEED-001), flagged as a candidate for outright rejection.

### Decision

**Formal Rejection.** Dynamic typing is rejected as a language feature for Orthon v0.1. Orthon will not provide a `dynamic` type, an `any` escape hatch, or any mechanism that bypasses compile-time type checking in user code.

### Rationale

**1. Declarative With Static Guarantees violation.** Orthon's `Declarative With Static Guarantees` principle states that the language provides declarative abstractions whose semantic guarantees are verified at compile time — never left to the programmer's manual discipline. Dynamic typing trades compile-time guarantees for flexibility: a `dynamic` variable defers all type checking to runtime, shifting the correctness burden from the compiler to the programmer.

This directly contradicts the principle that "what can be guaranteed statically should be guaranteed statically." The entire purpose of a `dynamic` escape hatch is to opt out of static guarantees, which is the opposite of Orthon's design direction.

**2. Explicitness violation.** With dynamic typing, type errors become runtime failures. The programmer writes code that looks correct but may fail at any point with a "method not found" or "type mismatch" runtime exception. This violates Explicitness because the type safety of the code is not apparent from its surface form — two calls that look identical may succeed or fail depending on runtime values.

**3. Correctness Before Performance violation.** Orthon's `Correctness Before Performance` principle states that correctness takes precedence over performance considerations. Dynamic typing optimizes for iteration speed (write now, debug later) over correctness (compiler guarantees correctness before the program runs). The principle requires that the compiler catch errors early, which dynamic typing systematically defeats.

**4. Minimal Core violation.** Dynamic typing introduces a fundamentally different type-checking regime — a dynamic type is checked at runtime, while all other types are checked at compile time. This requires the compiler and runtime to support two separate type systems, violating Minimal Core by adding a parallel mechanism rather than composing with the existing one.

**5. LLM Generability concerns.** A `dynamic` type undermines LLM code generation quality — without static type constraints, the LLM has no type-level feedback to guide correct API usage. The compiler cannot provide type errors for dynamic code, so LLM-generated code with `dynamic` types will produce runtime failures that are harder to diagnose.

### Consequences

- **Positive:** Every Orthon program has full static type information available to the compiler, enabling better diagnostics, optimisation, and tooling.
- **Positive:** No language-level escape hatch that encourages unsafe patterns — interop with dynamically-typed systems must go through explicit, visible APIs.
- **Positive:** Compiler errors catch type mismatches at compile time — no surprise "method not found" at runtime.
- **Negative:** Interop with dynamically-typed languages or protocols requires explicit marshalling code rather than an implicit `dynamic` conversion.
- **Negative:** Rapid prototyping patterns that rely on `any`/`dynamic` (common in TypeScript and C#) require more upfront type design in Orthon.

### Compliance

The Type System specification must not include a `dynamic` type, `any` escape hatch, or any mechanism that defers type checking from compile time to runtime for user code. Runtime dispatch remains available through `dyn Trait` (opt-in dynamic dispatch with static trait bound checking) — this is not dynamic typing because the trait contract is still verified at compile time.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| `dynamic` keyword (C#-style) | Rejected — fundamentally incompatible with Declarative With Static Guarantees. |
| `any` type (TypeScript-style) | Rejected — same incompatibility. `any` is worse because it silently disables checking on all operations on the value. |
| `unknown` type (TypeScript-safe alternative) | Rejected in its escape-hatch form — if `unknown` requires explicit type narrowing before use, it is not dynamic typing but rather a top type with mandatory checking, which is already covered by Orthon's trait system. |
| Reflection API (Java-style) | Rejected as a language-level dynamic escape hatch — reflection is a separate concern (see `REFLECTION_ALTERNATIVES.md`) and belongs in the StdLib or compiler intrinsics, not as a bypass of static typing. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | **Fail** | Dynamic typing introduces a type system within the type system — one that checks at runtime rather than compile time. This dual regime is internally inconsistent with Orthon's single, static type-checking model. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | **Fail** | Dynamic typing appears to simplify prototyping (fewer type annotations) but introduces a class of runtime errors that static typing eliminates. The trade-off increases overall cognitive load: the programmer must reason about two type-checking regimes. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | **Fail** | A dynamic type requires the compiler to generate runtime type checks for all operations on dynamic values, adding a second type system to the runtime architecture. This violates the architectural decision that Orthon is a statically-typed language by default. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | **Fail** | Dynamic types in APIs propagate unchecked — a single `dynamic` parameter in a library function disables static type checking for all callers, degrading type safety across the entire dependency graph. This long-term erosion of type safety is unacceptable. |

**Gates not applied:** `USER_VALUE_GATE`, `IMPLEMENTATION_INDEPENDENCE_GATE`, `LLM_GENERABILITY_GATE` — as a rejected concept, user value and implementation independence are not relevant. LLM generability concerns are cited in the rationale but the concept fails the structural gates regardless.
