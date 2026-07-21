# LLM Strategy

> Maximum LLM assistance during development. All toolchain components
> active; generated and hand-written code prioritises clarity,
> predictability, and machine-parseability.

## Policy Profile

| Policy | Value | Status |
|--------|-------|--------|
| **Syntax Policy** | `Lenient` | *Pending concept design* |
| **Verbosity Policy** | `Balanced` | *Pending concept design* |
| **Style Policy** | `LLMOptimized` | *Pending concept design* |
| **Toolchain Policy** | `Full` | Active |
| **Error Policy** | `LLM` | *Pending concept design* |
| **Inference Policy** | `LLMAssisted` | *Pending concept design* |

## Description

The LLM Strategy optimises for the development workflow where LLMs
are a primary interface for writing, reviewing, and transforming code.
It is the default strategy for early-stage exploration, rapid
prototyping, and projects where LLM collaboration is the norm.

- **Syntax Policy = Lenient** — the parser accepts common LLM-generated
  syntactic patterns such as trailing commas, consistent minor whitespace
  variations, and optional delimiters. This reduces edit-reject cycles
  when LLMs generate code that is logically correct but stylistically
  imperfect.
- **Verbosity Policy = Balanced** — code generation produces moderately
  verbose output: identifiers are descriptive but not overly long,
  comments explain intent rather than mechanics, and diagnostics include
  sufficient context for both human and LLM readers.
- **Style Policy = LLMOptimized** — formatting conventions favour
  patterns that LLMs generate most predictably: explicit delimiters
  (braces, parentheses) over indentation-sensitive grouping, consistent
  line breaks, and unambiguous operator precedence through explicit
  parenthesisation.
- **Toolchain Policy = Full** — all LLM Toolchain components are active:
  Schema Provider, Code Completer, Code Generator, Static Analyser,
  Documentation Generator, and Refactor/Migration Tool. The language
  server is LLM-aware and exposes schema endpoints.
- **Error Policy = LLM** — errors are formatted specifically for LLM
  consumption: structured diagnostic payloads with machine-readable
  error codes, repair hints, and sufficient context for the LLM to
  self-correct before presenting to the user.
- **Inference Policy = LLMAssisted** — type and semantic inference
  is augmented by LLM-based analysis for ambiguous or context-dependent
  cases. The compiler performs fast local inference; the LLM provides
  deeper cross-module analysis when needed.

## Applicability

The LLM Strategy is suitable when:

- **LLM collaboration is the primary workflow** — the team writes,
  reviews, and refactors code primarily through LLM chat or agent
  interfaces.
- **Rapid exploration** — correctness and clarity matter more than
  raw performance or minimal resource usage.
- **Mixed human-LLM teams** — code must be equally readable by humans
  and predictable to LLMs.
- **Prototyping phases** — before a specific Implementation Strategy
  is chosen, the LLM Strategy provides maximum flexibility.

The LLM Strategy is **not** suitable when:

- **Resource-constrained targets** — embedded systems, firmware, or
  real-time environments should use the Embedded Strategy.
- **Maximum throughput required** — HPC, data processing, or
  latency-sensitive workloads should use the High Performance Strategy.
- **Fully offline builds** — environments with no network access should
  use the Default Strategy with `Toolchain Policy = Offline`.

## Interaction with Other Strategies

The LLM Strategy is a **development-time strategy** that can be active
during authoring and review, while a different strategy is selected for
production execution. The Schema Provider ensures that code generated
under the LLM Strategy is semantically valid under any target strategy;
the Refactor/Migration Tool can transform code between strategy profiles
automatically.

```
Development (LLM Strategy active)
    │  Schema Provider validates
    │  Static Analyser checks
    │  Code Generator produces
    ▼
Production (Default / Embedded / High Performance Strategy active)
    │  Refactor/Migration Tool transforms
    │  Compiler optimises for target
    ▼
Execution
```

## See Also

- [`IMPLEMENTATION_STRATEGIES.md`](./IMPLEMENTATION_STRATEGIES.md)
- [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
- [`LLM_NATIVE_TOOLCHAIN.md`](../../how/concepts/research/LLM_NATIVE_TOOLCHAIN.md)
- [`ARCHITECTURE.md`](../architecture/ARCHITECTURE.md) § LLM Toolchain
- [`DEFAULT_STRATEGY.md`](./DEFAULT_STRATEGY.md)
- [`EMBEDDED_STRATEGY.md`](./EMBEDDED_STRATEGY.md)
- [`HIGH_PERFORMANCE_STRATEGY.md`](./HIGH_PERFORMANCE_STRATEGY.md)
