---
date: "2026-07-20 12:00"
promoted: false
---

## Orthon's LLM-Native Design — Complete Picture

### Architecture
Six-component LLM Toolchain layer alongside Language + Execution layers:

1. **Schema Provider** — machine-readable grammar, type system, stdlib contracts, and active Strategy schema. Eliminates hallucination.
2. **Code Completer** — context-aware completion understanding schema + active Policies.
3. **Code Generator** — generates Orthon from natural language, validates via Schema Provider.
4. **Static Analyser** — LLM's internal feedback loop; catches hallucinated APIs, wrong lifetimes, policy violations before compilation.
5. **Documentation Generator** — RAG-indexable, LLM-parseable docs in sync with schema.
6. **Refactor/Migration Tool** — safe, semantics-preserving transformations across strategy boundaries.

### Schema Provider (language model schemas)
Central source of truth: Grammar Schema + Type System Schema + Standard Library Schema + Strategy Schema. Versioned, deterministic, strategy-parametric.

### MCP Server
Not explicitly specced yet. Schema Provider is the conceptual equivalent — a canonical machine-readable interface. An MCP server wrapping it would be a natural implementation.

### LLM Generability Gate (EDR-011)
7th validation gate for every new language construct. Criteria: schema-serializable, ≥90% predictable generation, no hallucination surface, strategy-aware default, self-correctable by Static Analyser.

### LLM Strategy (Implementation Strategy profile)
Policies tuned for LLM collaboration: Lenient syntax, LLMOptimized style, Full toolchain, LLM-formatted errors, LLM-assisted inference.

### Language-Level Design for LLMs (VISION.md)
- Small surface, consistent rules (orthogonal design)
- Intent over Implementation (LLM says *what*; compiler/Policies handle *how*)
- Explicit Semantics (no invisible behavior to hallucinate)
- Execution Program model (LLM produces fully-defined, reproducible artifacts)
- Extensibility via libraries/Strategies (core language stays stable)
