# Intermediate Representation (IR)

> **Last updated:** 2026-07-20

## Overview

The Intermediate Representation (IR) is the bridge between parsed source code and code generation. It is the interface through which all downstream consumers — type checker, name resolver, code generator, and strategy selector — communicate about program structure. The IR is strategy-independent: it must represent any valid Orthon program without assuming a specific Implementation Strategy.

## Role in Architecture

Per [ARCHITECTURE.md](ARCHITECTURE.md), the IR sits between the Syntax layer (parsed AST) and the Implementation Strategy layer (code generation). Its role is to:

- Provide a canonical, typed representation of every Core Language construct
- Serve as the single source of truth after semantic analysis (type checking, name resolution) completes
- Enable strategy-agnostic optimization and analysis passes
- Support incremental compilation by preserving dependency structure

The IR does **not** prescribe execution semantics — those are the responsibility of the Implementation Strategy. The same IR must be mappable to any strategy: default, embedded, or high-performance.

## IR Invariants

1. **Strategy-independence** — The IR must not reference any Policy value or strategy-specific concept. A strategy choice (e.g., GC vs. arena allocation) must be representable as a transformation *over* the IR, not baked into it.
2. **Composability** — Every IR node must compose freely: expressions nest, statements sequence, and modules contain without special cases.
3. **Canonical representation** — Each Core Language construct has exactly one IR representation. No construct is representable in multiple ways.
4. **Losslessness** — The IR preserves all information from the source AST after name resolution and type inference. Round-tripping from source → IR → annotated source must not discard semantic information.
5. **Determinism** — The same source program, under the same compilation context, always produces the same IR. No non-determinism from hashing, ordering, or parallelism leaks into the IR.

## IR Interface Contracts

The IR guarantees to its consumers:

| Consumer | Guarantee | Violation |
|----------|-----------|-----------|
| Type checker | IR nodes that carry type annotations are well-typed by construction | Type mismatch error during IR construction |
| Name resolver | Every identifier reference in an IR node resolves to a declaration | Unresolved reference error |
| Code generator | The IR is a complete, typed, resolved program — no further source-level analysis needed | Incomplete IR → code generation failure |
| Strategy selector | Strategy-relevant annotations (e.g., allocation hints) are present as optional metadata, not hard-coded semantics | Missing strategy metadata → default to baseline |

## Key Design Decisions

1. **Tree-graph hybrid, not opaque bytecode** — The IR is an explicit typed tree with edges for control flow and data dependencies. This preserves structure for analysis passes (type checking, optimization) without requiring decompilation from bytecode.
2. **Representation symmetry with source AST** — The IR mirrors the source AST structure where possible. Each source-level construct has a corresponding IR node. This simplifies debugging, source mapping, and LLM-based tooling.
3. **Typed nodes** — All IR nodes carry type information after type checking. The IR itself is the typed representation; there is no separate typed-AST phase.
4. **Support for incremental compilation** — The IR preserves module-level dependency edges so that a change in one module only requires re-analysis of affected downstream modules.
5. **Strategy annotations as metadata** — Implementation Strategy choices are recorded as optional metadata nodes on the IR tree, not as first-class IR constructs. This preserves strategy-independence while allowing the code generator to make informed decisions.

## Relationships

| Document | Relationship |
|----------|-------------|
| [TYPE_SYSTEM.md](TYPE_SYSTEM.md) | IR carries typed nodes; the type checker produces typed IR from untyped AST |
| [NAME_RESOLUTION.md](NAME_RESOLUTION.md) | IR nodes reference resolved symbols; name resolution runs before type checking |
| [PARSER.md](PARSER.md) | Parser produces the untyped AST that the name resolver and type checker transform into typed IR |
| [ARCHITECTURE.md](ARCHITECTURE.md) | IR sits between Syntax and Implementation Strategy layers |

## Open Questions

1. Should the IR support a SSA (Static Single Assignment) form by default, or should SSA construction be an optimization pass?
2. How much syntactic sugar should be desugared in the IR vs. preserved for source mapping?
3. Should the IR be serializable for caching between compilation sessions?
