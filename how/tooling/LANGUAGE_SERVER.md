# Tooling: Language Server (LSP) — One Service, Many Editors

> **Status:** Open · **Priority:** P1 · **Target:** M3 (Tooling)
> **Source:** [D language design](../../why/DESIGN_INFLUENCES.md) § D — `serve-d` / `code-d` case study

---

## Description

An LSP (Language Server Protocol) server exposing Orthon's language
intelligence — completions, go-to-definition, hover, references,
diagnostics, and incremental re-analysis — over the standard protocol.
Every editor (VSCode, Neovim, Emacs, ...) is served by the same server;
each editor integration is a thin adapter, not a separate tool. D's
`serve-d` server with the `code-d` VSCode extension is the reference
pattern for this model.

## Problem

Without a language server, each editor would need its own Orthon
integration — duplicated parsing, name resolution, and diagnostics
that diverge as the language evolves. Editor support becomes
fragmented and stale. A single LSP server keeps language intelligence
in one place and makes every editor correct by construction.

Orthon's architecture already anticipates this consumer:
[`how/architecture/NAME_RESOLUTION.md`](../architecture/NAME_RESOLUTION.md)
lists an "IDE / LSP" interface contract (fast re-resolution on source
change for completions and go-to-definition), and
[`how/architecture/PARSER.md`](../architecture/PARSER.md) lists an
"IDE / LSP" guarantee (partial AST available even after a syntax
error).

## Spec Impact

### If this tool exists

- One server serves every editor; VSCode/Neovim/etc. get a thin
  adapter instead of a bespoke tool.
- The language service becomes a single backend that both human
  editors and LLM tooling can consume — symbols, references,
  completions, and diagnostics are queryable by the LLM Toolchain
  (Schema Provider, Static Analyser, `orthon describe`) without a
  second, parallel mechanism.
- Incremental analysis exercises the parser's error-recovery and the
  name resolver's incrementality contracts in real use.

### If this tool does NOT exist

- Editor integrations fragment; each reimplements parsing and
  resolution and drifts from the spec.
- The LLM toolchain would need a separate access path to the same
  symbol/type/reference data — two mechanisms for one fact.
- The "REPL with IDE-level intelligence" goal
  ([`DEVELOPER_TOOLING.md`](../concepts/research/deferrable/DEVELOPER_TOOLING.md))
  has no shared engine to build on.

## Language Feature Dependencies

- **Parser error recovery** — partial AST after a syntax error is
  required for live diagnostics and completion. Already specified in
  [`how/architecture/PARSER.md`](../architecture/PARSER.md).
- **Incremental name resolution** — fast re-resolution on source
  change for completions and go-to-definition. Already specified in
  [`how/architecture/NAME_RESOLUTION.md`](../architecture/NAME_RESOLUTION.md).
- **Queryable symbol model** — symbol definitions, references, and
  dependency edges must be resolvable; the same machinery that powers
  [`ORTHON-DESCRIBE.md`](ORTHON-DESCRIBE.md) (AST-aware symbol
  discovery) is the natural foundation for the LSP server.
- **Ecosystem boundary** — classified as **Tooling** (M3), not
  language core. The language defines the contracts; the server is an
  implementation concern (see
  [`REPOSITORY_INTROSPECTION.md`](../concepts/research/REPOSITORY_INTROSPECTION.md)).

## Spec Documents Affected

- [`how/architecture/PARSER.md`](../architecture/PARSER.md) — error
  recovery must be a first-class contract, not a best-effort detail.
- [`how/architecture/NAME_RESOLUTION.md`](../architecture/NAME_RESOLUTION.md) —
  resolution must be incremental and queryable.
- [`how/architecture/IR.md`](../architecture/IR.md) — the symbol/type
  model must support live queries (shared with `orthon describe`).
- [`what/EXECUTION_MODEL.md`](../../what/EXECUTION_MODEL.md) — no
  direct impact; the server is a static-analysis consumer.

## Notes

- The LLM Toolchain
  ([`LLM_NATIVE_TOOLCHAIN.md`](../concepts/research/deferrable/LLM_NATIVE_TOOLCHAIN.md))
  is a sibling consumer: a single "language service" layer should feed
  both LSP (human editors) and LLM tooling. Recording this now avoids
  building two parallel access mechanisms in M3.
- D's `serve-d` + `code-d` is the reference case study: a
  community-built server with thin per-editor adapters.
- [`ORTHON-DESCRIBE.md`](ORTHON-DESCRIBE.md) notes it is a "potential
  precursor to a full LSP implementation" — both tooling requirements
  share the same queryable-symbol foundation.
