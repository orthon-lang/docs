# Orthon Glossary

> A unified reference of all project terminology.
> Each entry includes a definition, cross-references to source documents, and links to related terms.

---

## A

### Architecture

The layered structure of Orthon: Core Language → Syntax → Standard Library → Implementation Strategy.

The central architectural invariant: the **Core Language** and **Standard
Library** define interfaces and contracts; the **Implementation Strategy**
fulfills them. User code depends only on the interfaces, never on the
strategy.

- **Source:** `../how/architecture/ARCHITECTURE.md`
- **See also:** [Core Language](#core-language), [Implementation Strategy](#implementation-strategy), [Standard Library](#standard-library)

---

## B

### Binding Identity

Whether two *names* currently refer to the same storage location — a
compiler/runtime aliasing concern (borrow tracking, aliasing analysis),
not a first-class equality operator exposed on ordinary values. Distinct
from Value Identity; the Semantic Model's Identity dimension exists
partly to keep the two from being conflated.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity
- **See also:** [Value Identity](#value-identity), [Semantic Dimension](#semantic-dimension), [Orthogonality](#orthogonality)

---

## C

### Canonical Form

One of the equivalent syntactic ways to express a language construct. All canonical forms of a feature must be documented together (see *Show All Canonical Forms* principle).

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Documentation Principle
- **See also:** [Operator Equivalence](#operator-equivalence)

### Core Language

The minimal, stable set of language semantics and semantic contracts,
independent of any particular implementation. The Core defines *what
programs mean*, not *how they execute*. It specifies the interfaces
that every Implementation Strategy must fulfill.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Core Language, `../how/DESIGN_PRINCIPLES.md` § Minimal Core
- **See also:** [Architecture](#architecture), [Implementation Strategy](#implementation-strategy), [Standard Library](#standard-library)

---

## D

### Data

### Dependency Flow

The central invariant of the Semantic Dependency Architecture: each
semantic layer depends only on layers below it; no layer may reference
or rely on constructs from a layer above it. This creates a natural
Dependency Inversion at the language level, enforced architecturally
rather than by convention.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Language Pattern](#language-pattern), [Primitive Operation](#primitive-operation), [Semantic Layer](#semantic-layer)

### Data

The primary abstraction in Orthon. Values viewed without imposed semantic meaning — the raw material that modifiers transform.

```
(1, 2, 3)    → data, without semantic label
tuple(1, 2, 3) → the same data, now explicitly a Tuple
```

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Data, `how/concepts/research/DATA_MODEL.md`
- **See also:** [Data Modifier](#data-modifier), [Representation](#representation)

### Data Modifier

A construct that transforms data from one representation to another. Modifiers express programmer intent; the compiler determines the most efficient implementation.

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Data Modifiers
- **See also:** [Data](#data), [Representation](#representation)

### Declaration Kind (`fun` / `proc` / `new`)

The three mutually exclusive function-declaration kinds that carry
Orthon's Mutation contract at the declaration site rather than the call
site: **`fun`** (read-only, never mutates `self`, always returns a
value), **`proc`** (mutates `self`, identity preserved, may return a
value or nothing), and **`new`** (never mutates `self`, always produces
a distinct value). There is no `mut` modifier and no caller-side
annotation — the contract lives entirely in which of the three keywords
a declaration uses.

- **Source:** `../what/SEMANTIC_MODEL.md` § Mutation
- **See also:** [Explicit Semantics](#explicit-semantics), [Semantic Dimension](#semantic-dimension)

### Declaration Model (Unified)

Variables, functions, types, classes, and modules follow the same declaration principles and modifier system. No special-case declaration syntax for different kinds of entities.

- **Source:** `../why/MANIFESTO.md` § A unified declaration model

### Decorator

A Language Pattern (Level 2) that wraps a callable with additional behaviour
without modifying its declaration. In Orthon, a decorator is expressed as
function composition — a function that takes a callable and returns a
callable — rather than special syntax (`@` annotation).

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Language Pattern](#language-pattern), [Lambda](#lambda)

### Descriptor

A Language Pattern that intercepts attribute access (`.` or `@`) on a
composite type and redirects it to a programmable handler — analogous to
Python's descriptor protocol (`__get__`/`__set__`/`__delete__`). In Orthon,
descriptors operate over `@`-prefixed metadata access as well as `.`
attribute access, and are explicit rather than implicit.

- **Source:** Phase 4 design (Derived Features)
- **See also:** [Primitive Operation](#primitive-operation), [Language Pattern](#language-pattern)

### Deterministic Behavior

The same source code must produce identical observable behavior across optimization levels and implementations. Only performance characteristics may differ.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Deterministic Behavior
- **See also:** [Explicit Semantics](#explicit-semantics), [Semantics Before Optimization](#semantics-before-optimization)

---

## D (cont.)

### Decision Log

The detailed, per-gate reasoning trail behind a Tier 1–2 decision —
one entry per validated decision, one subsection per gate applied,
each working through that gate's method against the actual proposal
and recording the verdict it produces. Distinct from an artifact's own
terse Validation summary (verdict + citation only) and from the
Decision Journal (a one-row-per-decision index); the Decision Log is
where the reasoning that produced a verdict is actually recorded, not
just its conclusion.

- **Source:** `../how/gates/DECISION_LOG.md`, established by
  [EDR-014](../how/decision_records/process/EDR-014-decision-log.md)
- **See also:** [Decision Validation](#decision-validation), [Validation Gate](#validation-gate)

### Decision Validation

A framework of seven independent validation gates that every language
design decision must pass. Each gate examines the proposal from a
different perspective: user value, logical consistency, conceptual
simplicity, architectural integrity, implementation independence,
long-term maintainability, and LLM generability.

- **Source:** `../how/gates/DECISION_VALIDATION.md`
- **See also:** [Language Design Gate](#language-design-gate), [Validation Gate](#validation-gate)

---

## E

### EDR (Engineering Decision Record)

A unified record of a consequential engineering decision, classified
by one of 15 categorical lenses: Architecture, Technology, Tooling,
Process, Delivery, Operations, Quality, Security, Governance, Data,
AI, Documentation, Knowledge, Collaboration, or Product.

EDRs replace the earlier dual ADR (Architecture) / TDR (Tools) system.
ADR and TDR are now categories of EDR. All EDRs are stored in
`docs/how/decision_records/{category}/EDR-NNN-slug.md` with a master
index at `decision_records/INDEX.md`.

- **Source:** `../how/decision_records/process/EDR-001-edr-system.md`
- **See also:** [Architecture](#architecture), [Decision Validation](#decision-validation)

### Execution Descriptor

A declarative, first-class manifest of what a program requires to
execute. Describes language version, runtime, standard library,
dependencies, implementation strategy, target platform, permissions,
resources, and configuration. The Descriptor is explicitly declared
and versioned alongside the Program — it is never inferred from
filesystem layout or environment variables.

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Execution Descriptor
- **See also:** [Execution Program](#execution-program), [Program Enricher](#program-enricher)

### Execution Engine

Any consumer that takes an Execution Program and does something with
it — runs, debugs, tests, compiles, materialises, or deploys.
Interpreters, REPLs, notebook kernels, test runners, debuggers, JIT
compilers, AOT compilers, OCI builders, MicroVM builders, and WASM
builders are all Execution Engines. They differ only in *how* they
execute, not in *what* they execute.

```
Execution Program
    ├── Interpreter
    ├── REPL
    ├── Notebook Kernel
    ├── Test Runner
    ├── Debugger
    ├── JIT / AOT Compiler
    ├── OCI Builder
    ├── MicroVM Builder
    ├── WASM Builder
    └── Remote Executor
```

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Execution Engine
- **See also:** [Execution Program](#execution-program), [Program Enricher](#program-enricher)

### Explicit Optimization

Performance-oriented execution strategies are enabled intentionally by the programmer, never applied silently. The default execution model favors predictability over performance.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Explicit Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Semantics Before Optimization](#semantics-before-optimization)

### Execution Program

The central abstraction of the Orthon execution model. A fully-defined,
reproducible, content-addressed canonical representation of everything
needed to execute a program. Includes source code, runtime, standard
library, dependencies, implementation strategy, configuration,
permissions, resources, and metadata.

The Execution Program is the **canonical representation** — not a
binary, not a Docker image, not a virtual machine. It is an
infrastructure-independent model of an executable program.

```
Execution Program
├── Program
├── Runtime
├── Standard Library
├── Dependencies
├── Implementation Strategy
├── Configuration
├── Permissions
├── Resources
└── Metadata
```

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md`, `../how/architecture/ARCHITECTURE.md` § Execution Program Pipeline
- **See also:** [Execution Descriptor](#execution-descriptor), [Execution Engine](#execution-engine), [Program Enricher](#program-enricher)

### Explicit Optimization

Performance-oriented execution strategies are enabled intentionally by the programmer, never applied silently. The default execution model favors predictability over performance.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Explicit Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Semantics Before Optimization](#semantics-before-optimization)

### Explicit Semantics

Whenever an operation changes the meaning, lifetime, ownership, or behavior of data, it must be expressed explicitly in the syntax. No hidden conversions or implicit semantic changes.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Explicit Semantics
- **See also:** [Deterministic Behavior](#deterministic-behavior)

---

## F

### Foreign Function Interface (FFI)

A mechanism that allows Orthon programs to call functions written in
other languages (primarily C). The FFI defines the boundary between
Orthon's type system and memory model and those of foreign languages.

- **Source:** `../when/ROADMAP.md` § Milestone 8
- **See also:** [Standard Library](#standard-library)

### Fresh-Value Exemption

The rule that an unbound temporary — a literal, a constructor call, or
any expression result not yet bound to a name — has no prior owner and
no aliasable Binding Identity, and so may be passed directly into an
ownership-consuming context (e.g. `delegate(List())`) without an
explicit transfer marker. Once a value is bound to a name, the
exemption no longer applies and any subsequent transfer must be
syntactically explicit.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity, § Ownership
- **See also:** [Binding Identity](#binding-identity), [Semantic Dimension](#semantic-dimension)

---

## I

### Implementation Strategy

A named set of Policies that governs how language semantics are
realised. A Strategy is a coherent profile — for example, the
Default Strategy uses `Arena` allocation while the Embedded Strategy
uses `Static` allocation. Strategies are interchangeable: replacing
one must not change program semantics.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Implementation Strategy, `../how/strategies/IMPLEMENTATION_STRATEGIES.md`
- **See also:** [Architecture](#architecture), [Core Language](#core-language), [Policy](#policy), [Standard Library](#standard-library)

### Intent Over Implementation

The programmer describes *what* should happen; the compiler decides *how* to implement it efficiently.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Intent Over Implementation
- **See also:** [Explicit Semantics](#explicit-semantics)

### Program Enricher

The component that combines a Program with its Execution Descriptor
into a fully-defined Execution Program. Internally coordinates
dependency, runtime, strategy, platform, permission, and resource
resolvers. Externally it is a single step — the boundary between
"incomplete" and "fully-defined" program.

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Program Enricher
- **See also:** [Execution Descriptor](#execution-descriptor), [Execution Program](#execution-program)

---

## L

### Lambda

An anonymous function expression — a function value defined inline without a
name. Lambdas are first-class values and share the same call syntax as named
functions. Closure capture is explicit.

```
let double = fn (x: Int) -> Int
    x * 2

double(3)  // 6
```

- **Source:** `../how/concepts/research/essential/FUNCTIONS.md`
- **See also:** [Canonical Form](#canonical-form), [Language Pattern](#language-pattern)

### Language Design Gate

A quality checklist that every language design proposal must satisfy
before entering formal specification. Operationalises the Decision
Validation gates into a review form.

- **Source:** `../how/gates/_language-design.md`
- **See also:** [Decision Validation](#decision-validation), [Validation Gate](#validation-gate)

### Language Pattern

A construct in the Semantic Dependency Architecture (Level 2) that
composes Primitive Operations (Level 1) and Data Model (Level 0)
constructs to provide convenience without adding new semantics. A
Language Pattern can be fully expressed as a composition formula
showing which lower-level constructs produce it. Examples:
`context` = closure + callable + try/catch + defer;
`decorator` = callable + closure + metadata.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Dependency Flow](#dependency-flow), [Primitive Operation](#primitive-operation), [Semantic Layer](#semantic-layer)

### Language Consistency Principles

The set of cross-cutting design rules that ensure uniform behaviour across Orthon:

- **Representation Symmetry** — Reversible transformations use symmetric syntax (prefix creates a representation, postfix restores it).
- **Deterministic Behavior** — Same source, same observable behaviour (see [Deterministic Behavior](#deterministic-behavior)).
- **Stable Mental Model** — Programmers reason about language semantics, not compiler internals.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Language Consistency

### Language Server Protocol (LSP)

An open protocol that standardises communication between a language
server — which provides language features like autocompletion,
diagnostics, and go-to-definition — and a code editor or IDE. Orthon's
tooling implements an LSP server to provide IDE support.

- **Source:** `../when/ROADMAP.md` § Milestone 9
- **See also:** [LLM Toolchain](#llm-toolchain)

### LLM-Native

A design property of the language and its toolchain: Orthon is
*LLM-native*, not merely *LLM-compatible*. This means every language
construct is exposed as structured, machine-readable schema (not just
free-text documentation); the toolchain is strategy-aware and adjusts
its output based on active Policies; and all interfaces are
bidirectional — LLMs both consume language metadata and produce
enriched artifacts back into the toolchain.

- **Source:** `../how/architecture/ARCHITECTURE.md` § LLM Toolchain — Design Rationale
- **See also:** [LLM Strategy](#llm-strategy), [LLM Toolchain](#llm-toolchain)

### LLM Strategy

A named set of Policies governing how LLM-based tools interact with
Orthon code — covering syntax tolerance, verbosity, style, toolchain
activation, error formatting, and inference depth. An LLM Strategy
is a specialised Implementation Strategy that applies to the
LLM Toolchain layer rather than the execution environment.

- **Source:** `../how/IMPLEMENTATION_POLICIES.md`, `../how/architecture/ARCHITECTURE.md` § LLM Toolchain
- **See also:** [Implementation Strategy](#implementation-strategy), [LLM Toolchain](#llm-toolchain), [Policy](#policy)

### LLM Toolchain

A first-class architectural layer of tools and services that enable
LLMs to generate, analyse, complete, and transform Orthon code with
high fidelity. Components include the Schema Provider, Code Completer,
Code Generator, Static Analyser, Documentation Generator, and
Refactor/Migration Tool. The Toolchain sits alongside the Language and
Execution layers with bidirectional interfaces.

- **Source:** `../how/architecture/ARCHITECTURE.md` § LLM Toolchain
- **See also:** [LLM Strategy](#llm-strategy), [LLM-Native](#llm-native), [Architecture](#architecture)

---

## M

### Minimal Core

Complex language features should emerge from composition of simple primitives rather than from introducing new keywords or execution models. The language is *grown*, not *invented*.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Minimal Core, `../why/MANIFESTO.md` § Minimal core, maximum expressiveness
- **See also:** [Orthogonality](#orthogonality)

---

## N

### Named Before Symbolic

Every symbolic operator must have an equivalent named function. Symbols improve brevity; named functions improve readability. Both express the same semantics.

Example: `&x` and `ref(x)` are equivalent.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Named Before Symbolic
- **See also:** [Canonical Form](#canonical-form), [Operator Equivalence](#operator-equivalence)

---

## O

### Operator Equivalence

Every language operator has an equivalent named function. Operators are syntactic sugar over ordinary functions.

```
&x          == ref(x)
x->         == sequence(x)
```

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Operators and Named Functions, `../how/DESIGN_PRINCIPLES.md` § Named Before Symbolic
- **See also:** [Canonical Form](#canonical-form), [Named Before Symbolic](#named-before-symbolic)

### Orthogonality

Each language construct solves exactly one problem and combines freely with other constructs. No special cases, no context-dependent syntax, no conflicting rules. What you learn in one part of the language transfers directly to every other part.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Orthogonality, `../why/VISION.md` § Comfortable by Design, `../why/ZEN.md`, `../why/MANIFESTO.md`
- **See also:** [Minimal Core](#minimal-core), [Uniformity](#uniformity)

---

## P

### Primitive Operation

An atomic language operation in the Semantic Dependency Architecture
(Level 1) that cannot be decomposed into simpler operations. Primitive
Operations define the minimal set of actions on data. Examples:
variables, functions, call `()`, pack/unpack `*`, attribute access `.`,
metadata access `@`, condition, loop, closure, exceptions.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Dependency Flow](#dependency-flow), [Language Pattern](#language-pattern), [Semantic Layer](#semantic-layer)

### Policy

A declarative constraint or preference within a single domain-specific
area of implementation (allocation, algorithm selection, evaluation
mode, lifetime management, concurrency, etc.). Each Policy makes
decisions independently in its area of responsibility. Policies are
not part of the language — they are implementation choices assembled
into named Strategies.

- **Source:** `../how/IMPLEMENTATION_POLICIES.md`
- **See also:** [Implementation Strategy](#implementation-strategy), [Architecture](#architecture)

### Principle of Least Astonishment

The behaviour of a construct must match what a competent programmer intuitively expects. Surprises belong in discovery, not in semantics.

- **Source:** `../why/VISION.md` § Comfortable by Design
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Semantics](#explicit-semantics)

---

## R

### Reference

A handle to data that provides indirection without implying ownership. Two
forms: shared read-only reference and exclusive mutable reference. Reference
is a Primitive Operation (Level 1) — it underlies class identity, borrowing,
and delegate semantics.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity, `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Primitive Operation](#primitive-operation), [Representation](#representation), [Binding Identity](#binding-identity)

### Representation

A specific view of data. Orthon provides several fundamental representations:

- **Value** — A single, immutable value.
- **Tuple** — A fixed-size ordered collection.
- **Reference** — A handle to data (`&x` / `ref&`).
- **Sequence** — Values produced over time (see [Sequence](#sequence)).
- **Set** — An unordered collection of unique values.
- **Option** — A value that may be absent.
- **Result** — A value that may be an error.

- **Source:** `how/concepts/research/DATA_MODEL.md` § Model (What)
- **See also:** [Data](#data), [Data Modifier](#data-modifier)

### Representation Symmetry

Reversible transformations use symmetric syntax. Prefix form creates an alternative representation; postfix form restores the original.

```
&value   → Reference
ref&     → Value

*values  → Pack
pack*    → Values
```

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Representation Symmetry
- **See also:** [Explicit Semantics](#explicit-semantics), [Named Before Symbolic](#named-before-symbolic)

---

## S

### Semantic Dimension

One of six independent, orthogonal facets a Semantic Model uses to
fully characterize what an Orthon program means: **Identity** (what
does "the same" mean), **Ownership** (who is accountable for a value),
**Mutation** (how and when values change), **Evaluation** (when
expressions are evaluated), **Visibility** (what is reachable from
where), and **Lifetime** (how long a value lives). Each dimension
answers exactly one question, is defined independent of syntax and
Implementation Strategy, and composes orthogonally with the other five
— see `SEMANTIC_MODEL.md`'s Cross-Dimension Consistency section for all
fifteen pairwise interactions.

- **Source:** `../what/SEMANTIC_MODEL.md` § Semantic Dimensions
- **See also:** [Semantic Invariant](#semantic-invariant), [Core Language](#core-language), [Orthogonality](#orthogonality)

### Semantic Invariant

One of six cross-cutting rules that hold across all six Semantic
Dimensions at once, rather than belonging to any single dimension —
for example, "every value has exactly one owner at any point in the
program" or "mutation requires exclusive access; read access may be
shared." Semantic Invariants are the semantic floor every
Implementation Strategy must guarantee; a Strategy may choose its own
enforcement mechanism but may never relax an invariant.

- **Source:** `../what/SEMANTIC_MODEL.md` § Semantic Invariants
- **See also:** [Semantic Dimension](#semantic-dimension), [Implementation Strategy](#implementation-strategy)

### Semantic Layer

A level in the Semantic Dependency Architecture hierarchy. Each
Semantic Layer has a defined responsibility, a set of allowed
constructs, and a dependency rule: it may only reference constructs
from lower layers (never from higher layers). The six layers are:
Data Model (0), Primitive Operations (1), Language Patterns (2),
Standard Library (3), Frameworks (4), Applications (5).

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Dependency Flow](#dependency-flow), [Language Pattern](#language-pattern), [Primitive Operation](#primitive-operation)

### Semantics Before Optimization

Language semantics are independent of optimization. A correct Orthon program must produce the same observable behavior regardless of optimization level.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Semantics Before Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Optimization](#explicit-optimization)

### Sequence

A fundamental type representing a sequence of values produced over time. Unlike traditional generators or streams, a Sequence describes *what the result is*, not *how it is produced*. It is a normal object: it can be returned, stored, passed, transformed, or consumed incrementally.

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Sequence and the `emit` Keyword
- **See also:** [Representation](#representation)

### Structural Equality

The `==` comparison semantics for value-typed data: two values are
equal exactly when their structure (fields/contents) matches,
regardless of storage location. The default, and only, meaning of `==`
for plain (non-reference) types in Orthon — it is never overloaded to
mean identity comparison, and never silently answers the Binding
Identity question ("are these the same storage").

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity
- **See also:** [Value Identity](#value-identity), [Data](#data)

---

## V

### Validation Gate

One of seven independent perspectives used in [Decision Validation](#decision-validation)
to assess a language design proposal. Each gate has its own criteria,
pass/fail conditions, and scope of examination.

- **Source:** `../how/gates/DECISION_VALIDATION.md`
- **See also:** [Decision Validation](#decision-validation), [Language Design Gate](#language-design-gate)

### Value Identity

Whether two values are considered the same persistent entity across
time, independent of their current structural content. Meaningful only
for explicit, opt-in shared/reference types — a plain (structural)
value has no Value Identity beyond its structure. Distinct from
Binding Identity; Structural Equality (`==`) always answers the
value-identity question in value-semantics terms, never the
binding-identity question.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity
- **See also:** [Binding Identity](#binding-identity), [Structural Equality](#structural-equality)

### Stable Mental Model

Programmers should reason about language semantics, not compiler internals. Users should never need to understand implementation details to predict program behaviour.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Stable Mental Model
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Semantics](#explicit-semantics)

### Standard Library

The layer that defines public abstractions exposed by the language. It specifies *behaviour*, not *implementation*. The Standard Library is the interface between user code and the Implementation Strategy.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Standard Library
- **See also:** [Architecture](#architecture), [Core Language](#core-language), [Implementation Strategy](#implementation-strategy)

---

## U

### Uniformity

Equivalent concepts should be expressed in equivalent ways. Once a user learns a language pattern, the same pattern applies consistently throughout the language.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Uniformity
- **See also:** [Orthogonality](#orthogonality)

---

## W

### WASM (WebAssembly)

A binary instruction format for stack-based virtual machines, designed
as a portable compilation target. In Orthon, WASM is one of several
execution targets — alongside interpreters, AOT compilers, and OCI
builders — that consumes the same Execution Program artifact without
modification, following the Execution Program / Execution Engine
separation.

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Execution Engine, `../how/architecture/ARCHITECTURE.md` § Execution Program Pipeline
- **See also:** [Execution Engine](#execution-engine), [Execution Program](#execution-program)

---

## Z

### Zen of Orthon

The four guiding aphorisms of the language:

> Every special case creates complexity.
> Orthogonality removes exceptions.
> Consistency defeats complexity.
> Simplicity is the result of orthogonality.

- **Source:** `../why/ZEN.md`
- **See also:** [Orthogonality](#orthogonality)

---

## How to Maintain This Glossary

### Trigger Conditions

A new glossary entry is required when:
- A new concept document is created in `docs/how/concepts/research/`
- An existing concept introduces a domain-specific term not already in GLOSSARY.md
- An EDR introduces new terminology or redefines an existing term

### Process

1. **Identify** — when creating or modifying a concept document, identify any new terms that are not yet in GLOSSARY.md
2. **Add** — insert the term in the correct alphabetical section with a definition, source link, and "See also" links to related terms
3. **Cross-reference** — add a link from the source document's Affected Documents checklist to GLOSSARY.md
4. **Verify** — ensure no term has conflicting definitions across documents (a term must mean the same thing everywhere)

### Review Cadence

At every **phase boundary** (before starting a new phase), verify that all new terms introduced during the completed phase are registered in GLOSSARY.md. This is part of the phase completion checklist.

### Consistency Check

If the same term appears in multiple documents, verify that all definitions are consistent. Inconsistent definitions must be reconciled before the phase is considered complete. The Glossary is the source of truth — if a source document disagrees, the source document must be corrected.

### Rules

1. **Add terms proactively.** When a new concept is introduced in any design document, add it here.
2. **Keep definitions in sync.** If a term's meaning evolves, update this file and all source documents simultaneously.
3. **Prefer cross-references over duplication.** The Glossary defines *what* a term means; source documents explain *why and how* it matters.
4. **Alphabetical order.** Each section uses the term's first letter as a heading anchor.
