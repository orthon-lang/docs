# Context-Limited Modules

## Issue (Why)

How does a module system help an LLM (or a human) understand a module's behaviour without loading the entire codebase into context? To understand what a module does, you currently need to load its entire transitive closure. For an LLM with a finite attention window, this is a hard limit on the complexity of code that can be generated or verified in a single pass.

Three specific gaps motivate context-limited modules:

1. **LLM attention window** — If a module's effective surface fits in the window, the LLM can reason about it completely.
2. **Human cognitive load** — A module requiring understanding of 20 transitive dependencies is harder to reason about than one requiring 3.
3. **Independent reasoning** — A module with isolated effects can be tested, verified, and optimised independently.

## Principles

1. **Explicit public API** — Every module declares a public API that is the only way to interact with it.
2. **Declared dependencies** — Every module lists its dependencies explicitly. No undeclared transitive dependencies are accessible.
3. **Isolated effects** — Side effects (I/O, mutation, allocation) are declared in the module's header.
4. **No effect leakage** — A module does not inherit the effect types of its dependencies.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Module Visibility Policy | Determines what symbols are public, protected, or private |
| Dependency Declaration Policy | Governs how dependencies are declared and resolved |
| Effect Isolation Policy | Controls which effect types are tracked at the module boundary |
| Module Compilation Policy | Specifies whether modules are compiled independently |

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
```

### Context Window Budget

A module's "attention cost" is the total token count of:
1. Its own public API header
2. The public API headers of its direct dependencies (signatures only, not implementations)

```orthon
# Toolchain diagnostic
Module "http_client" context budget: 340 tokens
  Direct dependencies: 2 (120 tokens)
  Transitive (resolved, not loaded): 0
  Total surface: 460 tokens — fits in ~1K window
```

## Default Strategy

Context-limited modules are a **Language** construct — the module system affects name resolution, compilation units, and visibility. Compiler enforcement of:
- Explicit dependency declarations
- Effect isolation (compiler verifies the module performs only declared effects)
- No undeclared transitive dependency access
- Context window budget diagnostics (toolchain feature)

## Alternative Strategies

| Strategy | Description |
|---|---|
| File-scoped (C/Go) | No visibility control beyond file boundaries. |
| Explicit interface files (OCaml `.mli`) | Separate signature from implementation. Manual maintenance — drift risk. |
| Effect tracking (Koka) | Effect types in signatures. Most expressive but requires effect polymorphism. |

## Open Questions

1. Should effect tracking be mandatory or opt-in?
2. How do context-limited modules interact with incremental compilation?
3. Should the context budget diagnostic be a lint or a hard error?

## Decision History

- **EDR-072:** Context-Limited Modules accepted as **Language** — compiler-enforced capability-based module access. Module visibility, dependency declaration, and effect isolation require compiler support for verification. These are not expressible via library composition.
- **Classification per D-03:** Language. The module system is a fundamental language organisation construct — it affects name resolution, compilation units, and visibility. A library cannot introduce new scoping rules.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/architecture/ARCHITECTURE.md`
- [ ] `how/process/DECISION_PIPELINE.md`
