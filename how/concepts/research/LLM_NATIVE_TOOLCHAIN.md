# LLM-Native Toolchain

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

### The LLM code generation quality gap

LLMs are rapidly becoming a primary interface for writing, reviewing,
and transforming code. Yet most languages today are designed with only
human readers in mind. When an LLM generates code in these languages,
it faces three systematic problems:

1. **Hallucinated APIs and syntax** — LLMs lack a structured, machine-readable
   schema of the language. They rely on training data patterns, which
   inevitably produce calls to APIs that do not exist or syntax that
   is subtly wrong.

2. **Strategy-blind generation** — Generated code has no awareness of
   the active Implementation Strategy. Arena-allocated code mixed with
   GC-allocated patterns, or eager evaluation assumptions applied to
   lazy-defaults code, produce correct-yet-inefficient results.

3. **Inconsistent style** — Without canonical formatting rules exposed
   as a schema, LLMs produce code that varies in style, naming
   conventions, and structural patterns across generations.

### The documentation discovery problem

LLM tooling currently relies on scraped web documentation, which is
incomplete, out of date, or contradictory. There is no canonical,
machine-verifiable source of truth about the language that an LLM can
query at generation time.

### The feedback loop gap

When an LLM generates incorrect code, the only feedback is a
compilation error or a runtime failure. There is no structured
intermediate representation that the LLM can use to self-correct
before presenting code to the user.

## Principles

Which principles must not be violated?

### Minimal Core

The Core Language defines the concepts of a **Schema Provider** (the
machine-readable language contract) and **Toolchain Policy** (how LLM
tools interact with the language). It does not define specific LLM
models, inference protocols, or vendor APIs. Those belong in the
toolchain implementation.

### Intent Over Implementation

The programmer declares *what* LLM toolchain behaviour they need
(e.g., `Toolchain Policy = Standard`). The toolchain decides *how*
to fulfil it — which schema format, which completion strategy, which
analysis depth.

### Explicitness

LLM interaction modes must be explicitly declared. The programmer
chooses which Toolchain Policy is active; the language never silently
invokes LLM-based analysis or generation.

### Deterministic Behaviour (where applicable)

The Schema Provider must produce deterministic output for a given
language version and active strategy. Code generation, completion,
and analysis may be non-deterministic (LLM-dependent), but the schema
and validation paths are always reproducible.

## Model (What)

### Architecture

The LLM-Native Toolchain is a first-class architectural layer — not an
afterthought. It sits alongside the Language and Execution layers with
bidirectional interfaces:

``` mermaid
flowchart LR
    subgraph LANG["Language Layer"]
        CL["Core Language"]
        SX["Syntax"]
        SL["Standard Library"]
    end

    subgraph STRAT["Implementation Strategy"]
        ST["LLM Strategy"]
    end

    subgraph LLM["LLM Toolchain"]
        SP["Schema Provider"]
        CC["Code Completer"]
        CG["Code Generator"]
        SA["Static Analyser"]
        DG["Documentation Generator"]
        RM["Refactor / Migrate"]
    end

    subgraph EXEC["Execution Environment"]
        C["Compiler"]
        R["Runtime"]
    end

    LANG --> SP
    SP --> CC
    SP --> CG
    SP --> SA
    SP --> DG
    SP --> RM

    ST --> SP
    ST --> SA
    ST --> CG

    SA --> C
    C --> SA
    CG --> C
    CC --> C

    DG --> LANG
    RM --> LANG
    RM --> EXEC
```

### Components

**Schema Provider**
The central source of truth for the LLM Toolchain. Exposes:

- **Grammar Schema** — full language grammar in a machine-readable
  format (e.g., EBNF, JSON Schema, or a custom DSL). Covers syntax,
  keywords, operators, and delimiters.
- **Type System Schema** — the complete type system as structured
  data: primitive types, compound types, generic constraints,
  subtyping rules, and coherence conditions.
- **Standard Library Schema** — every public API, its signature,
  contracts, and policy annotations.
- **Strategy Schema** — active Implementation Strategy definitions,
  including all Policy values and their constraints.

The Schema Provider is versioned alongside the language and is the
*only* source of truth an LLM needs to generate valid Orthon code.

**Code Completer**
Provides context-aware code completion that understands:

- The language schema (syntax, types, stdlib)
- The active Implementation Strategy (Policies in effect)
- The surrounding code structure (scope, binding, type context)

Completions are ranked by both syntactic validity and policy
compatibility.

**Code Generator**
Generates Orthon code from natural language, higher-level
specifications, or migration scripts. Validates output through the
Schema Provider and Static Analyser before presentation.

**Static Analyser**
Performs deep semantic analysis on generated or hand-written code.
Checks for:

- Type correctness
- Policy violations
- Strategy compatibility
- Common LLM generation errors (hallucinated APIs, incorrect lifetimes,
  mismatched allocation patterns)

The Static Analyser is the LLM's internal feedback loop — it catches
mistakes before the compiler does, reducing iteration time.

**Documentation Generator**
Produces human-readable and LLM-parseable documentation. Maintains a
machine-readable knowledge base (vector index, structured metadata)
that LLMs can query via RAG. Ensures documentation and schema are
always in sync.

**Refactor / Migration Tool**
Automates safe, semantics-preserving code transformations. Guided by
the Schema Provider and Static Analyser, it can:

- Migrate code between Implementation Strategies
- Apply consistent style transformations
- Refactor across API boundaries
- Generate migration scripts for language version changes

### Schema Format Requirements

The Schema Provider's output must satisfy:

1. **Complete** — every valid language construct is representable.
2. **Deterministic** — identical inputs produce identical schema.
3. **Versioned** — each language version has an immutable schema.
4. **Strategy-parametric** — schema adjusts to active Policy values.
5. **Self-describing** — the schema includes metadata about its own
   structure and version.

### Interaction Modes

The Toolchain supports three interaction modes, governed by the
**Toolchain Policy**:

| Mode | Schema Provider | Code Completer | Code Generator | Static Analyser | Docs Gen | Refactor |
|------|:---:|:---:|:---:|:---:|:---:|:---:|
| **Minimal** | ✓ | — | — | — | — | — |
| **Standard** | ✓ | ✓ | — | ✓ | — | — |
| **Full** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

## Default Strategy

By default, the LLM-Native Toolchain follows the **Toolchain Policy**
specified in the active Implementation Strategy. For the LLM Strategy,
this is `Toolchain Policy = Full` — all toolchain components are
active and available.

See [`LLM_STRATEGY.md`](../../strategies/LLM_STRATEGY.md)
and [`IMPLEMENTATION_POLICIES.md`](../../IMPLEMENTATION_POLICIES.md)
§ Toolchain Policy.

## Alternative Strategies

Alternative toolchain profiles are selected by changing the
**Toolchain Policy** (and associated LLM-specific Policies) in a
different strategy profile.

| Profile | Toolchain Policy | Use Case |
|---------|:----------------:|----------|
| `LLM_STRATEGY.md` | `Full` | Maximum LLM assistance during development |
| `DEFAULT_STRATEGY.md` | `Standard` | Schema + completion + static analysis |
| `EMBEDDED_STRATEGY.md` | `Minimal` | Schema only; no LLM dependency at build time |
| `HIGH_PERFORMANCE_STRATEGY.md` | `Minimal` | Schema only; zero toolchain overhead |

Possible Toolchain Policy values: `Minimal`, `Standard`, `Full`,
`Offline`. Each represents a different point in the trade-off space
between LLM capability and toolchain independence.

See [`IMPLEMENTATION_POLICIES.md`](../../IMPLEMENTATION_POLICIES.md)
§ Toolchain Policy for the full catalogue.

## Open Questions

1. Should the Schema Provider expose a single unified schema or
   multiple specialised schemas (grammar, types, stdlib, strategy)?
2. Should the Static Analyser be a separate component or integrated
   into the compiler's diagnostic pipeline?
3. What is the canonical machine-readable format for the schema
   (JSON Schema, Protocol Buffers, custom DSL)?
4. How does the Code Completer interact with the LSP server? Should
   they share an internal completion engine?
5. Should the Documentation Generator produce RAG-indexable output as
   a first-class artifact?

## Decision History

Why were alternatives rejected?

| Decision | Rationale |
|----------|-----------|
| Toolchain as a separate layer (not embedded in compiler) | Keeps the Core Language independent of LLM concerns; toolchain can evolve independently |
| Schema Provider as the single source of truth | Prevents schema drift between documentation, LSP, and LLM tooling |
| Toolchain Policy as a first-class Policy type | Enables declarative control across all four strategy profiles without duplication |
| Static Analyser as external feedback loop | Reduces LLM iteration time by catching errors before compilation |
