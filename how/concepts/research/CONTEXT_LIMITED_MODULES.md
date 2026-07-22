# Context-Limited Modules

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

How does a module system help an LLM (or a human) understand a module's behaviour without loading the entire codebase into context?

Traditional approaches:

- **File-scoped modules** (C, Go packages) — simple but no visibility control beyond file boundaries. Understanding one module means understanding all its transitive dependencies.
- **Explicit interface files** (Modula-2, OCaml `.mli`, C++ header files) — separate signature from implementation. The signature is the contract; the implementation can be ignored. However, `.mli` files are manually maintained and drift from implementation.
- **Package managers with dependency graphs** (Rust `cargo`, Go modules) — dependency resolution but no language-level isolation. A module's transitive dependencies are visible unless explicitly hidden.
- **Effect-tracking modules** (Koka, Eff) — effect types visible in signatures. The most expressive isolation but requires effect polymorphism, adding complexity.

The core problem: **to understand what a module does, you currently need to load its entire transitive closure into context**. For an LLM with a finite attention window, this is a hard limit on the complexity of code that can be generated or verified in a single pass.

Three specific gaps motivate context-limited modules:

1. **LLM attention window** — An LLM can hold approximately N tokens in context. If a module's effective surface (its own signature + the signatures of its direct dependencies) fits in that window, the LLM can reason about it completely. If not, it must make simplifying assumptions or risk errors.

2. **Human cognitive load** — The same principle applies to human readers. A module that requires understanding of 20 transitive dependencies is harder to reason about than one that requires understanding of 3.

3. **Independent reasoning** — A module with isolated effects can be tested, verified, and optimised independently. Side effects that leak across module boundaries make local reasoning impossible.

## Principles

1. **Explicit public API** — Every module declares a public API that is the *only* way to interact with it. Private symbols, types, and effects are not visible outside the module.

2. **API is the contract** — The public API is the module's contract. It lists all exported functions, types, constants, and effects. Changes to the API are versioned; breaking changes are detectable.

3. **Declared dependencies** — Every module lists its dependencies explicitly. No undeclared transitive dependencies are accessible at the language level. This gives the reader (human or LLM) a complete picture of what the module interacts with.

4. **Isolated effects** — Side effects (I/O, mutation, allocation strategy) are declared in the module's header. A module that does not declare an effect type cannot perform that effect. This makes it possible to reason about a module's impact without reading its body.

5. **No effect leakage from dependencies** — A module does not inherit the effect types of its dependencies. If module A depends on module B (which does I/O), module A is not marked as performing I/O unless it explicitly calls B's I/O functions. Effect types track what the module *actually does*, not what its dependencies *can do*.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Module Visibility Policy | Determines what symbols are public, protected, or private within a module |
| Dependency Declaration Policy | Governs how dependencies are declared, versioned, and resolved |
| Effect Isolation Policy | Controls which effect types are tracked at the module boundary |
| Module Compilation Policy | Specifies whether modules are compiled independently (separate compilation) or together |

## Model (What)

### Module Declaration

A module declares its public API and dependencies in a header section:

```orthon
module http_client

# Declared dependencies — only these modules are accessible
use std::net::TcpStream
use std::time::Duration
use std::result::Result

# Effects this module actually performs
effects: io, alloc

# Public API — everything below is the interface contract
pub type Connection
pub type Request
pub type Response

pub fn connect(host: String, port: Int) -> Result<Connection, Error>
pub fn send(request: Request) -> Result<Response, Error>
pub fn close(conn: Connection) -> Result<Void, Error>

# Private — not visible outside the module
type InternalBuffer
fn parse_response(raw: Bytes) -> Result<Response, Error>
```

### Dependency Declaration

Dependencies are declared at the module level, not imported inline:

```orthon
module data_processor

use std::collections::List      # explicit: this module uses List
use std::collections::Map       # explicit: this module uses Map
use std::sort::quick_sort       # explicit: this module uses quick_sort

# Does NOT have direct access to std::net or std::fs
# even if its dependencies use them
```

### Effect Isolation

Effects are declared as a set. The compiler verifies that the module's body does not perform undeclared effects:

```orthon
module pure_math
effects: none                 # no side effects at all

pub fn factorial(n: Int) -> Int
pub fn fibonacci(n: Int) -> Int

# compiler error if body tries I/O, allocation, or mutation
```

```orthon
module logger
effects: io, alloc

pub fn log(level: Level, message: String)
pub fn set_log_level(level: Level)
```

### Context Window Budget

A module's "attention cost" is the total token count of:

1. Its own public API header
2. The public API headers of its direct dependencies (recursively, but only public signatures, not implementations)

A toolchain diagnostic can report this cost:

```
Module "http_client" context budget: 340 tokens
  Direct dependencies: 2 (120 tokens)
  Transitive (resolved, not loaded): 0
  Total surface: 460 tokens — fits in ~1K window
```

## Core Inclusion Filter

Before entering formal validation, this concept is run through the Core Inclusion Filter procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 1 (Primitive Operation)

Library test:   Can Context-Limited Modules be implemented as a Standard
                Library function using existing Core constructs?

                The module system is a fundamental language organisation
                construct. It affects name resolution, compilation units,
                and visibility — none of which can be expressed as a library
                function. A library cannot introduce new scoping rules or
                effect tracking at module boundaries.

                → FAIL (cannot be a library).

Pattern test:   Can Context-Limited Modules be expressed as a composition
                of existing Primitive Operations and Data Model constructs?

                The module system requires new primitives:
                - Module declaration (a construct that groups symbols and
                  controls their visibility).
                - Effect declaration (a construct that tags a scope with
                  permitted effect types).
                - Explicit dependency declaration (a construct that lists
                  external modules and restricts access to them).

                None of these exist in the current primitive set (identifier,
                literal, assignment, function, call, attribute access, scope,
                reference, pack, unpack, operator definition).

                → FAIL (cannot be composed from existing primitives).

Primitive test: Is Context-Limited Modules an atomic operation on data
                that cannot be decomposed?

                The module system introduces new semantic mechanisms:
                scoped visibility groups, effect isolation at the module
                boundary, and explicit dependency management. These are
                atomic — they cannot be decomposed into simpler primitive
                operations because they define the organisational structure
                of the language itself.

                → PASS — belongs in Primitive Operations (Level 1).

Filter result: Level 1 (Primitive Operation).
```

## Default Strategy

By default, all modules use explicit public API declaration with private-by-default visibility. Effects are inferred where possible but must be declared if the module performs I/O, allocation, or mutation. Dependencies are declared at the module level.

The compiler produces a "context budget" diagnostic for each module, showing the token count of its public API surface. This is displayed as a suggestion, not an error.

## Alternative Strategies

| Strategy | Description |
|---|---|
| **Flat modules** | No nested modules. All public symbols are at the top level. Simplest model, but may produce large public APIs. |
| **Hierarchical modules** | Modules can contain submodules. Visibility can be scoped to the parent module. More expressive but increases context surface. |
| **No effect isolation** | Effects are tracked internally by the compiler but not declared in the module header. Simpler syntax but loses the reasoning benefit. |
| **Open modules** | Modules can re-export symbols from dependencies, making them part of the public API. Useful for facade modules but increases context cost. |
| **Signature-only modules** | Like OCaml `.mli` — separate interface and implementation files. Interface can be distributed without implementation. |

## Open Questions

1. Should effect isolation be mandatory or optional? Mandatory provides stronger reasoning guarantees but adds declaration overhead to every module.
2. Should the "context budget" be a hard compiler limit or a warning? A hard limit enforces LLM-compatibility but may be too restrictive for large modules.
3. How do context-limited modules interact with the package manager? A package's public API is the union of its modules' public APIs.
4. Can modules declare "friend" modules that have access to private symbols (like C++ `friend`)?
5. Should the effect isolation allow "escape hatches" for unsafe optimisations?
6. How does module versioning interact with the explicit dependency declaration — should the dependency pin a version range?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/architecture/NAME_RESOLUTION.md`
- [ ] `how/architecture/ARCHITECTURE.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [x] `how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- [ ] Other: `what/SYNTAX.md` (module declaration syntax)
