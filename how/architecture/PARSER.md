# Parser

> **Last updated:** 2026-07-20

## Overview

The Parser transforms Orthon source code into an Abstract Syntax Tree (AST) conforming to Core Language semantics. Its responsibility ends at syntactic analysis: it produces a well-formed AST with no semantic interpretation, name resolution, or type checking. This separation ensures that parsing is independent of the semantic pipeline and can be tested, extended, or replaced independently.

## Role in Architecture

Per [ARCHITECTURE.md](ARCHITECTURE.md), the Parser is part of the Syntax layer. It consumes raw source text and produces an AST that the downstream pipeline (name resolver → type checker → IR construction) consumes. The Parser:

- Defines the concrete syntax of Orthon
- Produces an AST that preserves source location information for error reporting
- Implements error recovery so that a single syntax error does not abort parsing
- Is strategy-independent — the AST it produces contains no Implementation Strategy concepts

## Parsing Invariants

1. **No semantic analysis** — The parser must not perform name resolution, type checking, or any semantic interpretation. These are the responsibility of downstream pipeline stages.
2. **Error recovery** — A syntax error in one construct must not prevent parsing of subsequent constructs. The parser produces a partial AST with error nodes for recovery.
3. **Context-free grammar** — The Orthon grammar must be context-free (or very nearly so), enabling efficient parsing without unbounded lookahead or backtracking. Context-sensitive constructs are validated downstream.
4. **Determinism** — The same source text always produces the same AST. No parsing ambiguity is permitted.
5. **Source fidelity** — The AST preserves source positions (line, column, offset) for every token and node, enabling accurate error messages and source mapping.

## Interface Contracts

| Consumer | Guarantee | Violation |
|----------|-----------|-----------|
| Name resolver | AST with all identifiers preserved, no syntactic ambiguity | Ambiguous parse → parser error |
| Type checker | Well-formed AST; all nodes follow grammatical rules | Malformed AST → type checker rejection |
| Error reporter | Source locations on every node for error attribution | Missing location → degraded error messages |
| IDE / LSP | Partial AST available even after syntax error | Aborted parse → no editor feedback |

## Key Design Decisions

1. **Recursive descent as default strategy** — A hand-written recursive descent parser is the default. It is the most flexible for language prototyping, produces clear error messages, and requires no external parser generator dependency. If performance demands it, a generated parser can replace the hand-written one without changing the AST interface.
2. **Lexer as a separate pass** — Tokenization is a separate phase before parsing. This simplifies the grammar (no tokenization rules mixed with parse rules), enables independent testing, and allows the lexer to be replaced (e.g., for whitespace-significant syntax) without changing the parser.
3. **Whitespace significance deferred to Phase 3** — Whether Orthon uses significant indentation (like Python) or delimiter-based syntax (like Rust/C) is a Phase 3 concept decision. The parser architecture supports both: the lexer can produce INDENT/DEDENT tokens or ignore whitespace as configured.
4. **Support for incremental parsing** — The parser is designed to support re-parsing of individual modules or top-level declarations when source changes, enabling fast IDE feedback and incremental compilation.
5. **Explicit syntax for all constructs** — Every Core Language construct has a distinct syntactic form. No syntactic ambiguity between constructs is permitted.

## Relationships

| Document | Relationship |
|----------|-------------|
| [NAME_RESOLUTION.md](NAME_RESOLUTION.md) | Parser feeds unresolved AST to name resolver |
| [TYPE_SYSTEM.md](TYPE_SYSTEM.md) | Parser AST is consumed after name resolution for type checking |
| [IR.md](IR.md) | Parser AST is the raw material from which typed IR is constructed |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Parser is the Syntax layer component |
| Core Language concepts | Parser syntax conforms to [Core Concepts](../what/CORE_CONCEPTS.md) semantics |

## Open Questions

1. Should Orthon use significant indentation (Python-style) or explicit delimiters (Rust/C-style)? Deferred to Phase 3.
2. Should there be a separate lexer specification document, or is the lexer described sufficiently within PARSER.md?
3. What is the exact error recovery strategy — panic mode, error productions, or something else?
