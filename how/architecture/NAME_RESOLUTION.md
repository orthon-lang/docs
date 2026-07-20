# Name Resolution

> **Last updated:** 2026-07-20

## Overview

Name Resolution is the process of mapping every identifier occurrence in a program to its corresponding declaration — determining what each name refers to. It establishes the scoping rules of the language, detects shadowing and ambiguity, and produces the resolved AST that the type checker consumes. Name resolution runs between parsing and type checking: the parser produces an unresolved AST with identifier strings; the name resolver produces a resolved AST where every identifier is linked to its declaration.

## Role in Architecture

The Name Resolver sits between the parser and the type checker:
- **Input:** Unresolved AST from the parser (identifiers as strings)
- **Output:** Resolved AST (identifier references → declaration nodes)
- **Guarantee:** Every identifier resolves to exactly one declaration, or produces a clear resolution error
- **Constraint:** Resolution is deterministic and does not depend on type information (it precedes type checking)

## Name Resolution Invariants

1. **Hygienic scoping** — No accidental capture of identifiers across scope boundaries. Identifiers introduced in one scope do not leak into unrelated scopes unless explicitly exported.
2. **Lexical (static) scoping by default** — An identifier's meaning is determined by its position in the source code, not by runtime control flow. Inner scopes may shadow outer declarations.
3. **Explicit shadowing with warning** — Shadowing is permitted but must be explicit (e.g., a `let` or `fn` keyword in the inner scope). The compiler issues a warning when shadowing occurs, as it is a common source of confusion.
4. **No implicit globals** — All identifiers must be declared before use (or be in scope). There are no implicit global variables or automatic hoisting.
5. **Determinism** — The same program always produces the same resolution. No non-determinism from import order or compilation unit ordering.

## Interface Contracts

| Consumer | Guarantee | Violation |
|----------|-----------|-----------|
| Type checker | Every identifier reference maps to a declaration node | Unresolved reference → compilation error with diagnostic |
| IR construction | Resolved names available for typed IR nodes | Incomplete resolution → incomplete IR |
| Error reporter | Specific error messages for ambiguous or unresolved references | Ambiguity error → clear suggestion for disambiguation |
| IDE / LSP | Fast re-resolution on source change for completions and go-to-definition | Slow resolution → poor editor experience |

## Key Design Decisions

1. **Lexical (static) scoping as default** — Scope is determined by the syntactic structure of the program. Function bodies, blocks, and module bodies each create a new scope. This is the most predictable scoping model and aligns with programmer intuition.
2. **Module-level explicit imports** — All names from other modules must be explicitly imported. No implicit visibility across module boundaries. This ensures that the origin of every name used in a module is clear from the import declarations.
3. **No implicit globals** — Top-level declarations are accessible within their module, but are not automatically visible in child scopes or other modules without explicit import. The language does not have a global namespace that accumulates declarations.
4. **Module system design deferred to Phase 3** — The full module system (import semantics, visibility modifiers, cyclic dependency handling) is deferred to Phase 3 concept design. This specification covers the scoping and resolution rules that apply within a single module.
5. **Shadowing is explicit** — A declaration in an inner scope shadows an outer declaration only when introduced with an explicit keyword (`let`, `fn`, etc.). Silent shadowing (e.g., a parameter name matching a global) produces a warning.

## Relationships

| Document | Relationship |
|----------|-------------|
| [PARSER.md](PARSER.md) | Name resolver consumes the AST produced by the parser |
| [TYPE_SYSTEM.md](TYPE_SYSTEM.md) | Resolved names feed the type checker for type inference |
| [IR.md](IR.md) | IR nodes reference resolved symbols |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Name resolution sits between Syntax and Core Language layers |
| [IMPLEMENTATION_POLICIES.md](../IMPLEMENTATION_POLICIES.md) | Scope and visibility rules may be affected by module Policy choices |

## Open Questions

1. Should Orthon support cyclic module dependencies (with forward declaration), or should module graphs be strictly acyclic?
2. What is the exact resolution algorithm — will it use a simple tree-walking scope stack, or something more sophisticated (e.g., lexical address form)?
3. Should there be a distinction between public and private visibility at the declaration level, or is visibility purely module-scoped?
