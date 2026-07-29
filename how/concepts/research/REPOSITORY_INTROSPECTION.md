# Repository Introspection

> **⚠️ DRAFT — This document is exploratory research.**
> It describes tooling that belongs in the implementation repository, not in the language spec.
> The language spec defines the module header format that makes introspection possible.
>
> **Last updated:** 2026-07-29

## Issue (Why)

How does an LLM (or human) discover a project's module structure, dependencies, and public API without reading every file into context?

Traditional approaches rely on the LLM navigating the filesystem tree, guessing import paths, and grepping for symbols. This is error-prone: agents add wrong imports, miss needed ones, and spend context budget on "side quests" across the codebase.

BAML solves this by eliminating imports entirely (filesystem-as-namespace, FQN-only references). But this comes at the cost of encapsulation and capability-based access control — values Orthon explicitly prioritises.

**Orthon's approach:** Instead of changing the language, provide **repository introspection tools** that export the semantic module map — the same information a compiler already has — in a format LLMs can consume directly.

## Principles

1. **Language unchanged** — Introspection tools are an implementation concern, not a language extension. The language defines the module header format; tools export its content.
2. **Module header as source of truth** — Every tool reads the module header (`module ... use ... effects ...`) and the public API section. No inference from file paths.
3. **Semantic, not syntactic** — Tools understand the module structure, dependency graph, effect propagation, and type signatures — not just file listings.
4. **LLM-first output** — Output is designed for direct consumption by an LLM agent: structured, token-efficient, and self-contained (no follow-up queries required).
5. **Composable** — Tools can be chained: `orthon describe` on a module, `orthon deps` to trace its dependencies, `orthon symbols` to find specific declarations.

## Tools

### `orthon describe [module]`

Returns the complete semantic profile of a module. This is the primary entry point for LLM orientation.

```
$ orthon describe http.client

Module: http.client
Path: src/net/http/client.orthon

Dependencies:
  socket          (direct)
  tls             (direct)
  dns             (direct)

Effects: io, alloc
Context budget: 340 tokens (header + direct deps)

Public API:
  type Connection
  type Request
  type Response
  fn connect(host: String, port: Int) -> Result<Connection, Error>
  fn send(request: Request) -> Result<Response, Error>
  fn close(conn: Connection) -> Result<Void, Error>
```

### `orthon deps [module]`

Returns the dependency graph — direct and transitive — with effect propagation.

```
$ orthon deps http.client

http.client
├── socket          (direct)     effects: io
│   └── buffer      (transitive) effects: alloc
├── tls             (direct)     effects: io, crypto
└── dns             (direct)     effects: io

Transitive total: 4 modules
Cumulative effects: io, alloc, crypto
```

### `orthon symbols [module]`

Returns all exported symbols with their locations, types, and documentation.

```
$ orthon symbols http.client

http.client::Connection       type     src/net/http/client.orthon:12
http.client::Request          type     src/net/http/client.orthon:13
http.client::Response         type     src/net/http/client.orthon:14
http.client::connect          fn       src/net/http/client.orthon:16
  └─ Signature: (host: String, port: Int) -> Result<Connection, Error>
http.client::send             fn       src/net/http/client.orthon:18
  └─ Signature: (request: Request) -> Result<Response, Error>
http.client::close            fn       src/net/http/client.orthon:20
  └─ Signature: (conn: Connection) -> Result<Void, Error>
```

### `orthon graph [--format <text|dot|json>]`

Renders the full module dependency graph for the project.

```
$ orthon graph

http.client ──┬── socket ── buffer
              ├── tls
              └── dns

server ──┬── http.client
         ├── db ── cache
         └── auth
```

The JSON format is designed for LLM ingestion — a structured adjacency list with effect annotations:

```json
{
  "nodes": [
    {"id": "http.client", "effects": ["io", "alloc"]},
    {"id": "socket", "effects": ["io"]}
  ],
  "edges": [
    {"from": "http.client", "to": "socket", "kind": "direct"},
    {"from": "http.client", "to": "tls", "kind": "direct"}
  ]
}
```

## LLM Integration Pattern

The recommended workflow for an LLM agent working with an Orthon project:

1. **`orthon graph`** — Get the full module map (one call, complete picture).
2. **`orthon describe <module>`** — For relevant modules, get headers and API.
3. **`orthon deps <module>`** — For modules of interest, trace effect propagation.
4. Read the actual implementation files only where needed.

This replaces the current pattern of:
- Listing directory structure
- Grepping for import statements
- Reading files to discover dependencies
- Guessing which imports are missing

## Relationship to EDR-072

Repository introspection tools are the **tooling complement** to `CONTEXT_LIMITED_MODULES` (EDR-072). The module header format defined by EDR-072 provides the structured data that these tools export.

| EDR-072 | Introspection tools |
|---------|---------------------|
| Defines module header structure | Reads and exports header content |
| Compiler verifies declarations | Presents declarations to LLM |
| Effect isolation (compile-time) | Effect visibility (LLM-readable) |
| Context budget (compiler diagnostic) | Context budget (explicit in output) |

## Alternatives Considered

| Strategy | Why Not Chosen |
|----------|----------------|
| Filesystem-as-namespace (BAML) | Sacrifices encapsulation and capability checks. Contradicts EDR-072. |
| LSP protocol for LLMs | Standard LSP is designed for IDE UI, not token-bounded LLM context. Over-fetching problem. |
| Generated module index file | Static file goes out of sync. Tools always reflect current state. |
| AI-specific compiler output flag | Same effect, but coupled to compilation. Tools work pre-compilation. |

## Open Questions

1. Should `orthon describe` include context budget in tokens or a simpler metric (module count, dependency depth)?
2. Should tools support a "watch mode" that outputs incremental diffs as the LLM edits files?
3. Should `orthon graph` support subgraph queries (`--subgraph http.client` showing only relevant transitive closure)?
4. How does this interact with the Execution Program model — does `orthon graph` show execution programs as nodes?

## Decision History

| Date | Decision | Source |
|------|----------|--------|
| 2026-07-29 | Repository introspection is an implementation tooling concern, not a language extension. Tools read the module header format defined by EDR-072. | BAML gap analysis + architectural analysis |

---

### Cross-References

- [CONTEXT_LIMITED_MODULES.md](../../what/concepts/CONTEXT_LIMITED_MODULES.md) — Module header format that tools export
- [NAMESPACES.md](essential/NAMESPACES.md) — Namespace vs. filesystem decision (complementary analysis)
- [baml-concepts-orthon-gap-analysis.md](../../notes/baml-concepts-orthon-gap-analysis.md) — BAML comparison motivating introspection tooling
- [AGENTS.md](../../AGENTS.md) §5.1.6 — Agent workflow: read module headers first
