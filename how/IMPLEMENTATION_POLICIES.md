# Implementation Policies

> A **Policy** is a declarative constraint or preference within a
> single domain-specific area of implementation. Each Policy makes
> decisions independently within its area of responsibility. Policies
> are not part of the language — they are implementation choices.

## Relationship to Strategy

A **Strategy** is a named set of Policies. Policies are the building
blocks; a Strategy assembles them into a coherent profile.

```
Language Specification
        │
        ▼
Implementation Strategy  ← named set of Policies
        │
        ├── Allocation Policy
        ├── Algorithm Policy
        ├── Evaluation Policy
        ├── Lifetime Policy
        ├── Concurrency Policy
        └── ...
        │
        ▼
Implementation
```

## The Declarative Principle

**A Policy is declarative, not procedural.**

A Policy states *what* is preferred, not *how* to achieve it.

| ❌ Procedural (avoid) | ✅ Declarative (preferred) |
|---|---|
| `if speed then TimSort else MergeSort` | `Algorithm Policy = Adaptive` |
| `if low_memory then malloc else mmap` | `Allocation Policy = Arena` |

This keeps Policies orthogonal, composable, and independent of the
concrete implementation beneath them.

## Policy Catalogue

Each Policy type relates to one or more language concepts. A Policy
type is added when its first corresponding concept is designed (see
[`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md) —
`IMPLEMENTATION_INDEPENDENCE_GATE`).

> **Note:** The exact many-to-many mapping between concepts and Policy
> types is pending validation. When a concept is designed, its Policy
> footprint must be determined (see `_language-design.md` § Policy footprint).

### Allocation Policy

**Related Concepts:** [`concepts/ALLOCATION.md`](../what/concepts/ALLOCATION.md) *(+ pending validation)*

Controls memory acquisition and deallocation strategy.

| Value | Description |
|-------|-------------|
| `Heap` | General-purpose heap allocator (malloc/free) |
| `Arena` | Region-based allocation, bulk deallocation |
| `Linear` | Sequential bump allocation, no deallocation |
| `GC` | Tracing garbage collection |
| `Static` | Compile-time fixed allocation, no runtime allocator |

### Algorithm Policy

**Status:** *Pending concept design.*

Controls the choice of algorithm implementations (sorting, searching,
hashing, etc.) where multiple correct implementations exist.

| Possible Values | Description |
|-----------------|-------------|
| `Adaptive` | Selects algorithm based on input characteristics |
| `MemoryOptimized` | Prefers algorithms with minimal memory footprint |
| `Parallel` | Prefers algorithms that exploit parallelism |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Evaluation Policy

**Status:** *Pending concept design.*

Controls when and how expressions are evaluated.

| Possible Values | Description |
|-----------------|-------------|
| `Lazy Defaults` | Default arguments and some expressions evaluated lazily |
| `Eager` | All expressions evaluated immediately |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Lifetime Policy

**Status:** *Pending concept design.*

Controls how object lifetimes are managed and when resources are
released.

| Value | Description |
|-------|-------------|
| `ReferenceCounting` | Reference-counted automatic memory management |
| `GarbageCollected` | Tracing garbage collection, deferred reclamation |
| `Manual` | Explicit allocation and deallocation by the programmer |
| `RegionBased` | Region- or arena-based bulk lifetime management |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Concurrency Policy

**Status:** *Pending concept design.*

Controls how concurrent tasks are scheduled and executed.

| Possible Values | Description |
|-----------------|-------------|
| `ThreadPool` | Fixed pool of worker threads |
| `WorkStealing` | Dynamic work distribution across threads |
| `EventLoop` | Single-threaded event-driven concurrency |
| `None` | Single-threaded execution, no concurrency |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Syntax Policy

**Status:** *Pending concept design.*

Controls how syntax is parsed and interpreted, accommodating both
hand-written and LLM-generated code.

| Possible Values | Description |
|-----------------|-------------|
| `Strict` | Full parser; rejects any deviation from canonical syntax |
| `Lenient` | Tolerant parser; accepts common LLM-generated syntactic patterns (e.g., trailing commas, minor whitespace inconsistencies) |
| `Recovery` | Error-recovering parser; produces best-effort AST from malformed input |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Verbosity Policy

**Status:** *Pending concept design.*

Controls the level of detail in generated code, error messages,
documentation, and diagnostic output.

| Possible Values | Description |
|-----------------|-------------|
| `Concise` | Minimal output; short identifiers, compact formatting |
| `Balanced` | Moderate verbosity; human-readable with sufficient context |
| `Explicit` | Full qualification, elaborated error messages, comprehensive diagnostics |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Style Policy

**Status:** *Pending concept design.*

Controls code formatting and stylistic conventions, ensuring
consistent output from both human and LLM-generated code.

| Possible Values | Description |
|-----------------|-------------|
| `Canonical` | Enforces Orthon's canonical formatting rules |
| `Flexible` | Accepts stylistic variation within defined bounds |
| `LLMOptimized` | Prefers patterns that LLMs generate most predictably (e.g., explicit delimiters, consistent indentation, unambiguous grouping) |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Toolchain Policy

**Status:** *Pending concept design.*

Controls which LLM Toolchain components are active and how they
interact with the language layer and execution environment.

| Possible Values | Description |
|-----------------|-------------|
| `Minimal` | Schema Provider only |
| `Standard` | Schema Provider + Code Completer + Static Analyser |
| `Full` | All Toolchain components enabled |
| `Offline` | No Toolchain components; no external LLM dependency |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Error Policy

**Status:** *Pending concept design.*

Controls how errors are reported, formatted, and surfaced to the
programmer or LLM.

| Possible Values | Description |
|-----------------|-------------|
| `Human` | Natural-language error messages with suggestions and context |
| `Structured` | Machine-readable error codes and structured diagnostic data |
| `LLM` | Errors formatted for LLM consumption — structured context with actionable repair hints |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Inference Policy

**Status:** *Pending concept design.*

Controls the depth and strategy of type and semantic inference
performed by the compiler and toolchain.

| Possible Values | Description |
|-----------------|-------------|
| `Local` | Inference limited to single expressions or statements |
| `Function` | Full function-body inference |
| `Global` | Cross-function and cross-module inference |
| `LLMAssisted` | Inference augmented by LLM-based semantic analysis for ambiguous or context-dependent cases |

*Specific values will be formalised when the corresponding language
concept is designed.*

## Extending the Catalogue

New Policy types are added as new language concepts are designed. Each
new Policy must:

1. Have at least one corresponding language concept (documented in
   `docs/what/concepts/`).
2. Pass the [`IMPLEMENTATION_INDEPENDENCE_GATE`](gates/DECISION_VALIDATION.md#gate-catalogue):
   the concept must be definable without reference to the Policy.
3. Define declarative values — not procedures.
4. Be added to this catalogue and cross-referenced from each
   related concept document.
5. If the Policy type already exists, verify compatibility with
   existing related concepts before adding new ones.

## See Also

- [`ARCHITECTURE.md`](architecture/ARCHITECTURE.md) § Implementation Strategy
- [`IMPLEMENTATION_STRATEGIES.md`](strategies/IMPLEMENTATION_STRATEGIES.md)
- [`DEFAULT_STRATEGY.md`](strategies/DEFAULT_STRATEGY.md)
- [`EMBEDDED_STRATEGY.md`](strategies/EMBEDDED_STRATEGY.md)
- [`HIGH_PERFORMANCE_STRATEGY.md`](strategies/HIGH_PERFORMANCE_STRATEGY.md)
