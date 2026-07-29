# Module Header File (`.orthon`)

> **⚠️ DRAFT — Exploratory hypothesis.**
> Proposes a directory-level module header file (`.orthon`) as the semantic
> index of an Orthon module — analogous to Python's `__init__.py` but as a
> language-level construct, not a runtime initialization hook.
>
> **Last updated:** 2026-07-29

## Problem

How does an LLM (or human) discover a module's dependencies, effects, and
public API **before opening any implementation file**?

Current approaches:

| Approach | Cost |
|----------|------|
| Inline module header (EDR-072) | Header lives inside an implementation file. LLM must open at least one file to find it. |
| Filesystem-as-namespace (BAML) | Eliminates imports but sacrifices encapsulation and capability checks. |
| Repository introspection tools (`orthon describe`) | Requires tooling to be installed and invoked. Not available at filesystem-browse time. |

The gap: when an LLM browses a project's directory tree, it sees file and
directory names. It cannot see module dependencies, effects, or API surface
until it reads implementation files. For projects with many modules, this
means opening multiple files before understanding the structure.

## Proposal

Each module directory **must** contain a `.orthon` file that declares the
module header — namespace, dependencies, effects, and public API symbols.

```
src/
├── net/
│   ├── .orthon                    # module net; use socket, tls, dns; effects: io
│   ├── http/
│   │   ├── .orthon                # module net.http; use json; effects: io
│   │   ├── client.orthon
│   │   ├── server.orthon
│   │   └── types.orthon
│   └── socket/
│       ├── .orthon                # module net.socket; use buffer; effects: io
│       └── impl.orthon
└── db/
    ├── .orthon                    # module db; use net; effects: io, alloc
    ├── postgres.orthon
    └── cache.orthon
```

The `.orthon` file is the **single source of truth** for the module's contract:

```orthon
# net/.orthon
module net
  use socket
  use tls
  effects: io
---

# Public API — signatures only (no implementations)
pub fn resolve(host: String) -> Result<IpAddress, Error>
pub fn connect(addr: IpAddress) -> Result<Connection, Error>
```

Sibling `.orthon` files in the same directory contribute to the same module.
Child directories are sub-modules, automatically namespaced by path.

### Key properties

1. **Discoverable by convention** — Every module directory has a `.orthon` file
   at its root. LLM scans directory listing, finds `.orthon`, reads the contract.
2. **No implementation in header** — Unlike inline headers, `.orthon` contains
   only declarations (signatures, type definitions, exported symbols). No
   function bodies, no expressions.
3. **Compiler-enforced correspondence** — The compiler verifies that every `pub`
   declaration in `.orthon` has a matching implementation in the module's files,
   and that no undeclared `pub` symbols exist in implementation files.
4. **Optional for leaf files** — A single-file module (no directory) can still
   use an inline header. `.orthon` is required only when a module spans multiple
   files in a directory.

## Effect on Semantic Model

| Dimension | Impact |
|-----------|--------|
| **Module Visibility** | No change. Visibility rules (`priv`, module-default, `pub`) remain as defined in VISIBILITY_AND_ENCAPSULATION.md. `.orthon` exports `pub` symbols only. |
| **Dependency Declaration** | Dependencies move from inline header to `.orthon` file. The `use` declaration is still the source of truth; only its location changes. |
| **Effect Isolation** | Effects declared in `.orthon`. Compiler verifies implementation files against the declared effect set, as in EDR-072. |
| **Name Resolution** | Namespace is inferred from directory path by default, but can be overridden in `.orthon`. The compiler resolves `use net::socket` by finding the nearest `net/.orthon` in the project tree. |
| **Compilation Unit** | The module is the compilation unit. `.orthon` defines the module boundary. All `.orthon` files in a directory tree form the module hierarchy. |

## Effect on Building Blocks

| Building Block | Impact |
|----------------|--------|
| **scope** | `.orthon` defines a named scope. All files under its directory contribute to that scope. |
| **identifier** | Symbols defined in `.orthon` become the module's public identifiers. Implementation files may use private identifiers not listed in `.orthon`. |
| **visibility** | `.orthon` explicitly exports `pub` symbols. Non-`pub` symbols in implementation files are invisible outside the module — same as current model. |

## Implementation Level

| Layer | Role |
|-------|------|
| **Core Language** | The `.orthon` file is a language-level construct. The compiler must understand `.orthon` syntax, enforce correspondence with implementation files, and resolve module boundaries from the directory tree. |
| **Stdlib** | Not applicable. The stdlib may use `.orthon` internally but does not define the mechanism. |
| **Build Tool / Framework** | The build tool (`orthon build`) uses `.orthon` files to determine compilation order, module boundaries, and dependency resolution. But the **semantics** of `.orthon` are defined at the language level. |

## Trade-offs

### Positive

- **LLM-first discoverability** — `.orthon` is visible at directory-listing
  time. No files need to be opened to understand module structure.
- **Single source of truth** — Dependencies and effects are declared in one
  place per module, not spread across implementation files.
- **Compiler-enforced contract** — Unlike Python's `__init__.py` (runtime
  initialization, no enforcement), `.orthon` is semantically checked.
- **Namespace convention, not language requirement** — The default namespace
  mapping from directory path to module name is a build convention. A file can
  override it with `module custom.name` in `.orthon`. The compiler never
  requires filesystem → namespace correspondence — it only requires that
  `.orthon` exists for multi-file modules.
- **No import management overhead** — LLM reads `.orthon`, sees all `use`
  declarations. No grep for imports across files.

### Negative

- **Extra file per module** — Every module directory needs a `.orthon` file.
  Adds ceremony for small modules.
- **Split definition** — Module contract (`.orthon`) is separated from
  implementation. Risk of drift if compiler enforcement is weak.
- **New concept** — Requires a new EDR and updates to EDR-072 (which assumes
  inline header).
- **Migration cost** — Existing single-file modules need no change, but any
  module split across files requires adding `.orthon`.

## Related Concepts

- **CONTEXT_LIMITED_MODULES (EDR-072)** — Defines the module header format.
  `.orthon` is a deployment strategy for that format: directory-level instead
  of inline.
- **NAMESPACES.md** — Discusses filesystem-binding vs. logical namespace
  declarations. `.orthon` with overridable `module` declaration is the
  "convention, not requirement" middle ground.
- **REPOSITORY_INTROSPECTION.md** — Tools like `orthon describe` read `.orthon`
  files to produce the semantic project map. `.orthon` provides the machine-
  readable input these tools need.
- **MODULE_HEADER_FORMAT.md** *(proposed)* — Should specify the exact syntax
  of the module header section (TBD in Phase 5).

## Alternatives

| Alternative | Why Not Preferred |
|---|---|
| **Inline header only** (current EDR-072) | LLM must open an implementation file to find the header. No directory-level discoverability. |
| **Python `__init__.py` model** | Runtime initialization hook, not a compile-time contract. No compiler enforcement. |
| **OCaml `.mli` files** | Per-file signature, not per-module. High maintenance — every implementation file needs a matching interface file. |
| **BAML filesystem-as-namespace** | Sacrifices encapsulation and capability checks for import elimination. Contradicts EDR-072. |
| **Generated manifest file** | Static file goes out of sync. `.orthon` is the source of truth, not a generated artifact. |

## Open Questions

1. Should a single-file module be allowed to omit `.orthon` and use an inline
   header? (Current thinking: yes — `.orthon` is required only for multi-file
   modules.)
2. Should `.orthon` include full signatures or just symbol names? (Current
   thinking: full signatures — type information is essential for LLM context.)
3. Should the compiler enforce that every `pub` symbol in implementation files
   is declared in `.orthon`? (Current thinking: yes — this is the key
   enforcement mechanism.)
4. How does `.orthon` interact with the Execution Program model — does each
   execution program get its own `.orthon`?

## Decision History

| Date | Decision | Source |
|------|----------|--------|
| 2026-07-29 | Proposed as exploratory hypothesis. No decision recorded yet. | Module header discussion |

---

### Cross-References

- [CONTEXT_LIMITED_MODULES.md](../../what/concepts/CONTEXT_LIMITED_MODULES.md) — Module header format (EDR-072)
- [NAMESPACES.md](essential/NAMESPACES.md) — Filesystem vs. logical namespace analysis
- [REPOSITORY_INTROSPECTION.md](REPOSITORY_INTROSPECTION.md) — Tooling to export `.orthon` content
- [VISIBILITY_AND_ENCAPSULATION.md](essential/VISIBILITY_AND_ENCAPSULATION.md) — Visibility rules that `.orthon` exports
