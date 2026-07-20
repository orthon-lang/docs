# Type System

> **Last updated:** 2026-07-20

## Overview

The Type System validates the semantic correctness of an Orthon program through type checking and type inference. It bridges the parsed AST (from the name resolver) and the typed IR (consumed by code generation). The type system guarantees that well-typed programs do not encounter runtime type errors, while inferring types to reduce syntactic burden on the programmer.

## Role in Architecture

The Type System operates between the name resolver and IR construction stages:
- **Input:** Resolved AST (identifiers mapped to declarations)
- **Output:** Typed AST / typed IR (every expression has an inferred or annotated type)
- **Guarantee:** Type soundness — no well-typed program produces a runtime type error
- **Constraint:** The type system is strategy-independent — type rules do not depend on Implementation Strategy choices

## Type System Invariants

1. **Soundness** — Every well-typed program is free of runtime type errors. This is the fundamental contract: if the type checker accepts a program, no type error can occur at runtime.
2. **Completeness** — All valid programs are typeable. The type system must not reject programs that are semantically valid. (Practical exceptions, e.g., undecidable cases, must be explicitly documented.)
3. **Principal types** — Every expression has a unique most-general type (its principal type). This ensures that type inference produces predictable results and that there is no ambiguity about what type the system assigns.
4. **Determinism** — The same program always produces the same type assignment. Type checking is deterministic and reproducible.
5. **Termination** — Type checking and type inference always terminate. No infinite loops or unbounded recursion in the type checker.

## Interface Contracts

| Consumer | Guarantee | Violation |
|----------|-----------|-----------|
| IR construction | Typed IR nodes for all program expressions | Type error → compilation aborted with diagnostic |
| Code generator | Type information available for code generation | Missing type → code generator cannot select representation |
| Strategy selector | Type information guides strategy decisions (e.g., allocation size) | Insufficient type information → conservative default |
| Error reporter | Specific, accurate type error messages with source locations | Vague error → poor developer experience |

## Key Design Decisions

1. **Hindley-Milner-based inference as default** — Hindley-Milner type inference (HM) is the default approach. It is proven, predictable, and well-understood. HM supports parametric polymorphism (generics) without requiring explicit type annotations in most contexts. Explicit annotations are required at module boundaries to serve as documentation and compiler-checked contracts.
2. **Structural typing over nominal** — Types are compared by structure, not by name. This aligns with the Data-first philosophy: two values with the same structure are interchangeable regardless of how they were declared. Nominal typing (newtypes, branded types) is available as an explicit opt-in.
3. **Explicit annotations at module boundaries** — Function signatures, type aliases, and module exports require explicit type annotations. This ensures that the public interface of every module is self-documenting and compiler-verified. Internal expressions within a function body are fully inferred.
4. **Type system is strategy-independent** — Type rules do not reference Allocation Policy, Lifetime Policy, or any other Implementation Policy. The same program, when type-checked, produces the same type assignment regardless of the selected strategy.

## Relationships

| Document | Relationship |
|----------|-------------|
| [NAME_RESOLUTION.md](NAME_RESOLUTION.md) | Type checker consumes resolved identifiers from the name resolver |
| [IR.md](IR.md) | Type checker produces typed IR nodes for code generation |
| [PARSER.md](PARSER.md) | Type checker operates on the resolved AST produced by the parser + name resolver pipeline |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Type system is part of the Core Language layer |
| [CORE_CONCEPTS.md](../what/concepts/CORE_CONCEPTS.md) | Data representations map to types in the type system |

## Open Questions

1. Should the type system support higher-kinded polymorphism? Deferred to Phase 3.
2. How are type errors reported for complex generic types? The error message strategy (e.g., error context, type provenance tracking) needs specification.
3. Should row polymorphism be supported for structural records? Deferred to Phase 3.
4. What is the exact relationship between the type system and the Representation system (Value, Tuple, Reference, Sequence, etc.)?
