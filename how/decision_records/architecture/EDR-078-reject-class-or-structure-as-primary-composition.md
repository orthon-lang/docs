# EDR-078: Reject Class or Structure as Primary Composition Unit

**Status:** Rejected

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Project

---

### Context

The Class / Structure as Primary Composition Unit concept (from `how/concepts/research/deferrable/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`) proposes a "gentleman's set" combination of three orthogonal tools — data aggregate (derive), behaviour mixin (mixin-role trait), and structural contract (structural trait satisfaction) — as the idiomatic composition pattern for Orthon types. The document explores when to use each tool and when a subset suffices.

While the research itself is thoughtful and identifies genuine composition design tensions, the underlying question it addresses — whether a class or structure should serve as the *primary* composition unit — must be answered with a definitive "no" for Orthon.

### Decision

**Formal Rejection.** Class or structure as the primary composition unit is rejected. Orthon will not designate any single language construct as the "default" or "primary" way to compose types. Composition is achieved through the orthogonal combination of Orthon's primitive building blocks, not through a privileged composition construct.

### Rationale

**1. Data First principle violation.** Orthon's `Data First` principle states that data is the primary abstraction — language constructs operate by transforming data into different representations. Making a class or structure the primary composition unit would impose behaviour on data (the act of bundling data with methods as the default pattern) rather than treating data as something that is transformed by behaviour.

In a class-as-primary model, the programmer first defines a data layout (fields) and then attaches behaviour (methods) to it. This bundles data and behaviour by default, making it harder to transform data independently. Orthon's model — where data is separate and behaviour is expressed through functions and traits — keeps data as the primary abstraction.

**2. Minimal Core violation (PRIMITIVE_BLOCKS D-03 decision).** PRIMITIVE_BLOCKS.md (D-03 decision) establishes that class is a composition of primitives, not a primitive itself. The primitive blocks identified in the Semantic Dependency Architecture are `literal`, `identifier`, `pack`/`unpack` (Data Primitives) and `assignment`, `function`, `call`, `attribute access`, `scope`, `reference` (Data Operations Primitives). A class or structure is composed from `pack` (field grouping) + `function` (methods) + `scope` (method body) + `identifier` (type and field names). Making class the primary composition unit would promote a derived construct to the level of primitives, violating the RISC-like core philosophy.

From PRIMITIVE_BLOCKS.md § Composition Rules:
> "Class/structure type definition → `pack` (field layout) + `function` (constructors/methods) + `scope` + `identifier` (type name)."

Class is a pattern, not a primitive.

**3. Orthogonality violation.** Class hierarchies create coupling that prevents independent composition. A class serves three roles simultaneously:
- **Data layout** — what fields the type has
- **Behaviour binding** — what methods operate on those fields
- **Type identity** — the class name becomes the type name

This bundling prevents independent composition: you cannot change data layout without affecting type identity, and you cannot add behaviour without extending the data layout. Orthon's approach separates these concerns into orthogonal constructs:
- Data layout: `derive`-based data aggregates or `pack`/`unpack` composition
- Behaviour binding: trait `impl` blocks (EDR-019)
- Type identity: nominal trait binding or structural satisfaction

**4. The research document itself shows the problem.** The CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION research identifies a genuine tension between `TRAITS.md` (nominal, no data fields, explicit `impl`) and `STRUCTURAL_TYPING.md` / `MIXIN.md` (structural satisfaction, possible fields). Rather than resolving this tension by designating a "primary" composition unit, Orthon resolves it by keeping all three tools (data aggregate, trait contract, mixin) orthogonal and independent — the programmer composes them as needed without any single tool being primary.

### Consequences

- **Positive:** Composition remains truly orthogonal — no single construct is privileged, so every combination of data, behaviour, and contract is equally valid.
- **Positive:** The primitive core remains small — class is not a primitive, keeping the RISC-like semantic ISA clean.
- **Positive:** Forcing composition through separate constructs (data aggregates, trait impl blocks, structural satisfaction) prevents the accidental complexity of class hierarchies.
- **Negative:** Programmers coming from class-first languages (Java, C++, C#) must learn a decomposition model where data, behaviour, and contracts are defined separately rather than in a single class declaration.
- **Negative:** Some simple patterns (a type with fields and methods) require more declarations than a single class statement — though this is mitigated by `derive` and trait ergonomics.

### Compliance

The Language Specification must not designate any construct as the "primary" or "default" composition unit. Every concept design review must check that no single composition mechanism is implicitly privileged over others. The Phase 6 (CROSS_CUTTING.md) interaction matrix must verify that all composition mechanisms (derive, traits, structural satisfaction, mixins) remain orthogonal and non-hierarchical.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Primary composition unit with opt-out | Rejected — even an opt-in class-as-primary would bias language learning materials, examples, and community patterns toward a privileged construct. The bias itself is the problem. |
| "Gentleman's set" three-tool recipe | Rejected — the three-tool recipe is useful as guidance but must not be enshrined as the language's "primary" pattern. The decision of which tool to use belongs to the programmer per-problem. |
| Class as syntactic sugar over primitives | Rejected as primary — class as sugar is consistent with Minimal Core if class is one pattern among many, not the primary one. This is compatible with D-03 (class is composition of primitives) but rejecting the "primary" designation. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | **Fail** | Designating any single construct as the "primary" composition unit is inconsistent with Orthon's orthogonal core. The concept would introduce a privileged mechanism where none should exist. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | **Fail** | A "primary composition unit" appears simpler (one way to compose everything) but introduces hidden coupling between data, behaviour, and type identity. The apparent simplicity masks the complexity of class hierarchies. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | **Fail** | PRIMITIVE_BLOCKS.md (D-03) explicitly classifies class/struct as a composition of primitives, not a primitive. Making it the primary composition unit would contradict the established architectural decomposition. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | **Fail** | Language history (Java, C++, C#) shows that class-as-primary leads to hierarchical designs that resist refactoring. Orthon's orthogonal approach prevents this coupling from the start. |

**Gates not applied:** `USER_VALUE_GATE`, `IMPLEMENTATION_INDEPENDENCE_GATE`, `LLM_GENERABILITY_GATE` — as a rejected concept, these gates are not relevant for the rejection verdict.
