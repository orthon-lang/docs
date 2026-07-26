# Semantic Annotations

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language bridge the gap between what code *does* and what it *means* — for both human readers and LLMs?

This concept addresses two directions of the code-to-specification relationship:

- **Code → Specification (extraction)** — Given code, can a reader (human or LLM) determine what invariants, preconditions, and design intent the author had?
- **Specification → Code (generation)** — Given a specification, can an LLM generate code that satisfies it?

Traditional approaches:

- **Doc comments** (Javadoc, Doxygen, Rustdoc) — natural language descriptions attached to code. Readable but unstructured, compiler-ignored, and prone to drift.
- **Type annotations** — machine-checkable but limited in expressiveness. Types describe *what* flows, not *why* or *under what constraints*.
- **Assertions** — runtime-checkable but not part of the public API surface. No caller-side visibility.
- **Formal specifications** (TLA+, Alloy, ACSL) — expressive but external to the code. Require separate tooling and expertise.
- **Contract annotations** (see [`CONTRACTS.md`](CONTRACTS.md)) — pre/post/invariant as part of the language. This is a related orthogonal concept: contracts specify *what the code guarantees*, while semantic annotations specify *what the code means*.

The core problem: **code and its specification live in different representations — one machine-executable, the other human-readable — with no synchronisation mechanism**. For LLMs, this means the specification is either absent (the LLM must infer intent from code alone) or unstructured (natural language doc comments that the LLM must parse).

Semantic annotations bridge this gap by providing **structured, machine-parseable metadata attached to code points** — annotations that both the compiler and LLM Toolchain can read, verify, and use for generation.

## Principles

1. **Annotations are structured, not free-form** — A semantic annotation has a defined schema (name, fields, values). It is not arbitrary text. The compiler can parse, validate, and verify annotations.

2. **Annotations are code-adjacent, not code-separate** — An annotation is attached to a specific code point (function, type, parameter, expression). It is not a separate file or a separate specification.

3. **Compiler-visible, not compiler-enforced** — The compiler validates that annotations are well-formed (correct syntax, valid values) but does not enforce their semantics. Enforcement is deferred to the Design by Contract system (see [`CONTRACTS.md`](CONTRACTS.md)) where applicable.

4. **LLM-parseable by design** — The Schema Provider exposes annotations as structured data. An LLM can read annotations to understand intent, and generate annotations to document generated code.

5. **No runtime overhead** — Annotations are compile-time metadata only. They are stripped from release binaries unless explicitly preserved for reflection.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Annotation Policy | Governs which annotation namespaces are valid and whether annotations are mandatory or optional |
| Annotation Preservation Policy | Controls whether annotations are preserved in compiled output, debug symbols, or schema output |
| Annotation Inheritance Policy | Determines whether annotations are inherited by subtypes or overriding methods |
| Annotation Validation Policy | Specifies the level of validation the compiler performs on annotations (syntax only, schema check, or semantic check) |

## Model (What)

### Annotation Syntax

Annotations use `@` syntax and are attached to declarations:

```orthon
@description("Computes the square root of a non-negative number")
@example("sqrt(4.0) == 2.0")
@example("sqrt(0.0) == 0.0")
fn sqrt(x: Float) -> Float
    requires x >= 0.0
    ensures result * result ≈ x
```

### Standard Annotation Namespaces

The language provides a set of built-in annotation namespaces:

```orthon
@description("...")          # Natural language description
@example("...")              # Usage example (checked if runnable)
@invariant("...")            # Class/module invariant (natural language)
@deprecated("use X instead") # Deprecation marker with migration hint
@since("0.1")                # Version when this API was introduced
@see("../other_file.md")     # Cross-reference to related documentation
```

### Custom Annotations

Users can define their own annotation namespaces. The compiler validates syntax but not semantics:

```orthon
@metric("latency_p50")
@metric("latency_p99")
fn process_request(req: Request) -> Response
```

### LLM-Oriented Annotations

Annotations specifically targeting LLM consumption:

```orthon
@llm:hint("This function is commonly used with sort_descending")
@llm:context("Call this after authentication is established")
@llm:pattern("Factory method — returns a configured instance")
fn create_processor(config: Config) -> Processor
```

### Annotation on Parameters and Types

Annotations can be attached to any declaration:

```orthon
fn connect(
    @description("Hostname or IP address") host: String,
    @description("TCP port, typically 80 or 443") port: Int,
    @sensitive() password: String
) -> Result<Connection, Error>

@description("Represents a parsed HTTP response")
type Response
    @description("HTTP status code (e.g., 200, 404)") status: Int
    @description("Response body as bytes") body: Bytes
```

### Annotations in the Schema Provider

The Schema Provider exposes annotations alongside type information:

```json
{
    "name": "sqrt",
    "type": "(Float) -> Float",
    "annotations": [
        {"name": "description", "value": "Computes the square root..."},
        {"name": "example", "value": "sqrt(4.0) == 2.0"},
        {"name": "since", "value": "0.1"}
    ],
    "contract": {
        "requires": "x >= 0.0",
        "ensures": "result * result ≈ x"
    }
}
```

## Core Inclusion Filter

Before entering formal validation, this concept is run through the Core Inclusion Filter procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 2 (Language Pattern) — annotations are syntactic sugar
            over metadata associations rather than a new primitive operation.

Library test:   Can Semantic Annotations be implemented as a Standard
                Library function using existing Core constructs?

                Annotations require:
                - A new `@` syntax for attaching metadata to declarations
                - Compiler-level parsing and validation
                - Schema Provider exposure as structured data
                A library can provide annotation-like behaviour through
                decorator/wrapper functions, but cannot alter the language's
                syntax or make annotations a first-class, compiler-validated
                construct.

                → FAIL (cannot be a library).

Pattern test:   Can Semantic Annotations be expressed as a composition
                of existing Primitive Operations and Data Model constructs?

                Metadata associations could be approximated using:
                - A `metadata` function that attaches key-value pairs to
                  values at runtime.
                - Attribute access for reading annotations.
                However, the declarative, compile-time nature of annotations
                — validation, schema checking, Schema Provider exposure —
                requires a new syntactic construct. The runtime approximation
                loses the key benefit (compiler-visible, no-overhead metadata).

                → FAIL (cannot be composed from existing primitives without
                  losing key properties).

Primitive test: Is a Semantic Annotation an atomic operation on data that
                cannot be decomposed?

                A semantic annotation as a *syntactic construct* (`@name(value)`)
                is a new syntactic form. However, its semantic content —
                attaching structured metadata to a declaration — could be
                expressed as a Language Pattern if the language already had a
                built-in metadata association primitive. Since no such
                primitive exists yet, the annotation syntax itself is new.

                The annotation concept straddles the boundary: the `@` syntax
                is a syntactic-level addition (close to Level 1), but the
                metadata attachment semantics are a Language Pattern (Level 2).
                The Schema Provider exposure is Level 3 (Toolchain).

                → PASS — belongs in Language Patterns (Level 2), with
                  syntactic support at Level 1 (Primitive Operation).

Filter result: Level 2 (Language Pattern), with syntactic `@` support at
               Level 1 (Primitive Operation). Annotation consumption by the
               LLM Toolchain is Level 3 (Standard Library / Toolchain).
```

## Default Strategy

By default, annotations are syntax-checked and schema-validated by the compiler. They are preserved in debug builds and Schema Provider output, but stripped from release binaries. Custom annotation namespaces are allowed but must be declared at module level.

The `@description`, `@example`, and `@since` annotations are provided by the standard library.

## Alternative Strategies

| Strategy | Description |
|---|---|
| **Annotations as first-class runtime values** | Annotations are preserved in the compiled output and accessible via reflection. Useful for frameworks, serialisation, and dynamic behaviour. Higher runtime overhead. |
| **Annotations as comments** | `@` syntax is ignored by the compiler and treated as a comment. Only the LLM Toolchain and documentation generator read them. Zero language change, but no validation. |
| **No annotations beyond built-in** | Only the language-defined annotations (`@deprecated`, `@since`) are recognised. Custom annotations are not supported. Simpler specification but less extensible. |
| **External annotation files** | Annotations are stored in separate `.ann` files alongside source code. Like TypeScript's `.d.ts` — keeps source clean but introduces synchronisation burden. |

## Open Questions

1. Should annotations affect the type system? For example, should `@sensitive()` on a parameter affect how the value is handled across FFI boundaries?
2. Should annotations be inheritable? If a parent class has `@description("...")`, should the child class inherit it?
3. Should `@example` annotations be executable (like Rust doctests)?
4. How do annotations interact with the module system — are module-level annotations visible to consumers?
5. Should the `@` symbol overlap with the existing `@` meaning (metadata access, per Semantic Purity), or does `@` have a single meaning that covers both?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/DESIGN_PRINCIPLES.md` — Semantic Purity (`@` symbol)
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [x] `how/concepts/research/LITERATE_PROGRAMMING.md`
- [x] `how/concepts/research/CONTRACTS.md`
- [x] `how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`
- [ ] `how/strategies/LLM_STRATEGY.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
