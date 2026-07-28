# EDR-044: Structural Typing — Structural Trait Satisfaction

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Type System)

---

### Context

Two approaches to polymorphism dominate modern languages:

- **Nominal typing** (Java interfaces, Rust traits) — a type must explicitly declare `implements Interface`. Makes intent clear but creates ceremony: every type that satisfies an interface must say so.
- **Structural typing** (Go interfaces, TypeScript, OCaml objects) — a type satisfies an interface if it has the required methods, regardless of explicit declaration.

With TRAITS (EDR-019) established as Orthon's nominal trait system with explicit `impl Trait for Type` syntax, the question is whether Orthon also supports structural trait satisfaction — "if it quacks like a duck, it is a duck" — as a default or opt-in mode.

The research document at `how/concepts/research/important/STRUCTURAL_TYPING.md` proposes structural typing as the default mode for trait satisfaction.

The core tension: structural typing offers flexibility (no prior arrangement needed) but sacrifices explicitness (which trait a type satisfies is determined by the compiler, not by source code). This directly touches Orthon's Explicitness principle.

---

### Decision

**1. Structural typing is accepted as a Language feature, but as an opt-in mode — NOT the default.** A trait may be declared `structural` to enable structural satisfaction:

```orthon
# Nominal — explicit impl required (default)
trait Serializable
    fn serialize(self) -> String

# Structural — implicit satisfaction by method shape
structural trait Show
    fn show(self) -> String
```

When a trait is `structural`:
- Any type with methods matching the trait's signature satisfies the trait automatically.
- No explicit `impl Show for Type` block is required.
- An explicit `impl` block, if present, takes priority over structural matching for that type.
- Static dispatch is used by default; `dyn Trait` is available as opt-in.

**2. Nominal is the default.** Most traits in Orthon are nominal — explicitness over convenience. The `structural` keyword is an explicit opt-in by the trait author, making the looser matching rule syntactically visible.

**3. `@derive` works with both nominal and structural traits.** The derive mechanism (EDR-029) generates explicit `impl` blocks, which take priority over structural matching for derived traits. This ensures deterministic behaviour: a derived implementation is always preferred.

**4. Conflict resolution:** If a type structurally matches two traits with conflicting method signatures, the compiler reports an ambiguity error. The programmer resolves by providing an explicit `impl` block.

---

### Consequences

- **Positive:**
  - Structural typing provides duck-typing flexibility for marker traits (`Show`, `Default`) where explicit `impl` is pure ceremony.
  - Nominal-by-default preserves explicitness for semantically meaningful traits (`Serializable`, `Authenticatable`).
  - The `structural` keyword makes the choice visible — no hidden structural matching.
  - `@derive` works transparently — derives generate explicit `impl` blocks that override structural matching.

- **Negative:**
  - Adds complexity to trait resolution — the compiler must check both explicit `impl` blocks and structural matching candidates.
  - Conflict resolution for structural ambiguity requires programmer intervention (explicit `impl`).
  - Cross-module structural matching may create coherence challenges — the same type in different modules may satisfy different traits structurally.

---

### Compliance

1. A `structural` trait must accept structural satisfaction by any type with matching method signatures (names, parameter types, return type).
2. An explicit `impl StructuralTrait for Type` must always take priority over structural matching.
3. Ambiguity errors must cite all conflicting structural matches.
4. Traits without the `structural` keyword must require explicit `impl` — no implicit satisfaction.
5. The `structural` keyword must be part of the trait declaration, not an annotation or attribute.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Structural typing as the default for all traits | Violates Explicitness — a type's trait satisfaction becomes implicit. Nominal-by-default preserves Orthon's explicitness commitment. |
| No structural typing at all — nominal only | Acceptable, but misses the ergonomic win for marker traits (`Show`, `Default`) where explicit `impl` is pure mechanical ceremony. |
| Structural only for specified built-in traits | Less flexible — the programmer cannot define new structural traits. The `structural` keyword is more orthogonal. |
| TypeScript-style structural typing (no trait definition — shapes are anonymous) | Violates Orthon's trait-centric polymorphism model. Traits define behavioural contracts; structural typing is an extension of trait resolution, not a replacement. |
| Structural matching only within a module | Adds a module boundary rule that complicates the model. Cross-module structural matching is solvable with explicit `impl` overrides. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Structural typing eliminates ceremony for marker traits. The `structural` keyword makes the trade-off explicit. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Clear precedence: explicit `impl` > structural match. Ambiguity detection prevents silent conflicts. No contradictions with nominal TRAITS (EDR-019). |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Two modes (nominal, structural) with clear, simple rules. The `structural` keyword is a single syntactically visible modifier. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on the existing TRAITS architecture (EDR-019). Structural matching is an extension of trait resolution — no new architectural layer. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Structural typing semantics (method signature matching, priority rules, conflict resolution) are independent of any specific memory layout or dispatch strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Nominal-by-default with opt-in structural is a proven pattern (Scala, OCaml). The two-mode approach avoids the coherence challenges of fully structural systems. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The `structural` keyword makes the mode explicit. LLMs can reliably determine when a type structurally satisfies a trait by checking method signatures. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-044 for per-gate reasoning trail.
