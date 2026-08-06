# LSP as Orthon's Semantic Interface — Hypothesis

> **Status:** Exploratory hypothesis — no design decision made; relevant to M3 (Build System & Tooling).
> **Created:** 2026-08-06
> **Related:** [`../how/tooling/LANGUAGE_SERVER.md`](../how/tooling/LANGUAGE_SERVER.md) (tooling requirement, P1),
> [`../how/architecture/PARSER.md`](../how/architecture/PARSER.md),
> [`../how/architecture/NAME_RESOLUTION.md`](../how/architecture/NAME_RESOLUTION.md),
> [`../when/ROADMAP.md`](../when/ROADMAP.md) (§ M3 — Build System & Tooling)
>
> This note is the exploratory hypothesis behind the existing LSP tooling
> requirement. It is kept in `notes/` because it is research, not a decision:
> the requirement is the decision-track artifact; this note is the reasoning.

---

## Hypothesis

If Orthon's language server is built on top of the **same semantic core as the
compiler** — parser, resolver, type checker, and effect checker — then it stops
being a conventional autocomplete engine and becomes the language's **interactive
semantic interface**: a single backend that exposes semantic contracts (effects,
invariants, type-inference explanations, module and architecture rules) to both
human editors and AI agents.

The testable claim, restated: *"A semantic-core-backed LSP can show meaning, not
just text — and that semantic visibility is what makes Orthon's tooling stage
deliver its LLM-readiness goal."*

## Rationale

Orthon is designed around semantics that most languages keep implicit:

- **Orthogonal semantics** — each construct solves one problem and composes freely.
- **A strong type system** — types are explicit and inspectable.
- **Effects** — side effects are part of the type-level model.
- **Modules** — capability-based access control (`CONTEXT_LIMITED_MODULES`).
- **Declarative contracts** — functions declare what they guarantee.

Because these are first-class, the LSP can display far more than completion:
it can answer *what a program means* at each point. In a language where these
facts are not modeled, the LSP has nothing to show; in Orthon they are already
in the semantic core, waiting to be surfaced.

## What an LSP Is

An LSP (Language Server Protocol) server is a separate process that makes a
language's intelligence available to editors over a standard protocol.

> **The compiler knows the language → the LSP makes that knowledge available to
> the editor.**

One LSP server serves every editor at once — VS Code, Neovim, Emacs, JetBrains
(via plugins), Helix, Zed — because each integration is a thin adapter, not a
separate tool.

### Standard feature set

Almost all modern IDE features run through the LSP:

- autocompletion
- go-to-definition
- find all references
- rename symbol
- hover documentation
- error highlighting (diagnostics)
- semantic highlighting
- inlay hints
- code actions ("fix automatically")
- refactoring
- folding
- document outline

### Architecture

```
            IDE

         VSCode
         Neovim
         Zed
         Helix
            │
            │ LSP
            ▼
      Orthon Language Server
            │
      ┌─────┴───────────┐
      │                 │
   parser           type checker
      │                 │
      └───────┬─────────┘
              │
          compiler core
```

The strong preference: the LSP **reuses the same parser and type checker as the
compiler**. It must not duplicate compiler logic — it runs on top of the single
semantic core.

## How Orthon Semantics Amplify the LSP

Nine capabilities that become natural when the language itself models semantics:

### 1. Display the program's semantics

```orthon
readFile(path)
```

Hover:

```
readFile

effects:
    filesystem.read

may fail:
    FileNotFound
    PermissionDenied

complexity:
    O(file_size)
```

The IDE shows the function's **semantic contract**, not just a description.

### 2. Visualize effects

```orthon
processOrder()
```

The LSP can highlight:

```
Effects:

✓ Database.Write
✓ Network
✓ Logging
✗ Filesystem
```

Hard in most existing languages; natural when effects are part of the language.

### 3. Show data flow

```orthon
user
    .validate()
    .normalize()
    .save()
```

Hover on `save()`:

```
Input guarantees:

✓ validated
✓ normalized
✓ authenticated
```

The LSP shows **accumulated invariants** along the pipeline.

### 4. Explain type inference

```orthon
value := expr()
```

Hover:

```
Type inferred as:

Result<User, ValidationError>

Reason:

expr()
returns Result<T,E>

T resolved to User
E resolved to ValidationError
```

Invaluable for understanding complex inference results.

### 5. Visualize module dependencies

With capability-limited modules (`CONTEXT_LIMITED_MODULES`), the LSP can:

- show *why* a symbol is accessible;
- show the import chain;
- highlight unnecessary dependencies;
- propose adding a missing `use` automatically.

### 6. Check architectural rules

```orthon
UI
 ↓
Service
 ↓
Repository
```

If `Repository use UI`, the LSP flags the architecture violation before the
project is ever built.

### 7. Check purity live

```orthon
pure func calculate()
```

Typing `print(...)` inside it yields an immediate diagnostic:

```
Error

Pure function cannot perform IO.
```

### 8. Navigate effects

Typed effects enable precise editor commands:

```
Show Effect Graph
Who writes Database?
Who allocates memory?
```

### 9. Power the AI assistant

For a maximally formalized language, the LSP is the ideal context source for an
LLM. Instead of passing the whole file, provide the model:

- AST;
- types;
- effects;
- call graph;
- active invariants;
- symbols;
- dependencies.

This lets AI perform more precise transformations and explanations, grounded in
semantics rather than raw text.

## Implementation Reality

A typical toolchain is a linear pipeline:

```
Lexer
      ↓
Parser
      ↓
AST
      ↓
Resolver
      ↓
Type checker
      ↓
Effect checker
      ↓
Compiler
```

The LSP consumes almost all of it:

```
LSP

completion()
hover()
definition()
references()
rename()
diagnostics()
semanticTokens()
codeActions()
```

**The LSP must not duplicate compiler logic.** It operates over the same
semantic core (parser, resolver, type checker, effect checker) that the
compiler reuses — one source of truth for language facts.

## Proposed Feature Tiers

For Orthon, the LSP feature set divides into three levels.

### Tier 1 — MVP (mandatory)

- autocompletion;
- go-to-definition and find references;
- live compilation-error highlighting;
- hover with types and documentation;
- rename symbol;
- automatic `use` add/remove.

### Tier 2 — Language strengths

- function effect display;
- type-inference explanation;
- invariant and contract display;
- symbol provenance and module dependency tracing;
- architecture-rule checks based on language rules.

### Tier 3 — Unique Orthon capabilities

- interactive effect and dependency graph;
- semantic program overview (not merely syntactic);
- LLM integration via structured context (AST, types, effects, contracts),
  enabling far more precise AI refactoring and code generation.

If one of Orthon's goals is code that is simultaneously understandable by
humans, compilers, and AI, the LSP stops being an autocomplete aid and becomes
the **interactive semantic interface of the language** — the channel through
which IDEs, analysis tools, and AI assistants access the formal model of a
program.

## Relationship to Existing Artifacts

- [`../how/tooling/LANGUAGE_SERVER.md`](../how/tooling/LANGUAGE_SERVER.md) —
  the P1 tooling requirement this hypothesis underpins (source: D `serve-d` /
  `code-d` case study). This note is its exploratory enrichment; the requirement
  is the decision-track record.
- [`../how/architecture/PARSER.md`](../how/architecture/PARSER.md) — parser
  error recovery must be a first-class contract so a partial AST is available
  for live diagnostics and completion after a syntax error.
- [`../how/architecture/NAME_RESOLUTION.md`](../how/architecture/NAME_RESOLUTION.md) —
  resolution must be incremental and queryable for completion and
  go-to-definition latency.
- [`../how/tooling/ORTHON-DESCRIBE.md`](../how/tooling/ORTHON-DESCRIBE.md) —
  AST-aware symbol discovery is the natural foundation for the LSP's
  queryable symbol model.
- **LLM Toolchain** (Schema Provider, Static Analyser, `orthon describe`) —
  the same symbol/type/effect data serves both human editors and LLM tooling,
  avoiding a second parallel mechanism.
- [`../when/ROADMAP.md`](../when/ROADMAP.md) § M3 (item 9.2) — "Formatter,
  linter, LSP (Language Server Protocol)"; [`../how/tooling/README.md`](../how/tooling/README.md)
  is the catalogue input to M3 planning.

## Testability / Open Questions

The hypothesis is validated at M3, not at spec-freeze time. To make that
validation possible:

1. **How is "richer than autocomplete" measured?**
   - Conformance tests per feature tier (MVP / strengths / unique).
   - LLM-context quality metrics — e.g., does structured context
     (AST + types + effects + invariants) reduce generation errors versus
     raw-text context?
2. **Incrementality and latency** — completion and diagnostics must feel
   interactive; which incremental-reanalysis contracts must `PARSER.md` and
   `NAME_RESOLUTION.md` guarantee?
3. **Which spec contracts must exist first?** The LSP depends on a queryable
   semantic core — `TYPE_SYSTEM.md` (types inspectable), an effect checker,
   and a symbol model. Do the architecture docs currently commit to these?
4. **Ecosystem boundary** — the LSP is classified as Tooling (M3), not
   language core. The language defines the contracts; the server is an
   implementation concern.

## See Also

- [`../how/tooling/LANGUAGE_SERVER.md`](../how/tooling/LANGUAGE_SERVER.md) — the tooling requirement
- [`../how/tooling/README.md`](../how/tooling/README.md) — tooling requirements catalogue
- [`../how/architecture/PARSER.md`](../how/architecture/PARSER.md), [`../how/architecture/NAME_RESOLUTION.md`](../how/architecture/NAME_RESOLUTION.md) — semantic-core contracts
- [`../how/concepts/research/important/CONTEXT_LIMITED_MODULES.md`](../how/concepts/research/important/CONTEXT_LIMITED_MODULES.md) — capability-limited modules
- [`../how/concepts/research/deferrable/DEVELOPER_TOOLING.md`](../how/concepts/research/deferrable/DEVELOPER_TOOLING.md) — "REPL with IDE-level intelligence" goal
- [`./llm-native-concept-shortlist.md`](./llm-native-concept-shortlist.md) — LLM-generability framing
- [`../when/ROADMAP.md`](../when/ROADMAP.md) § M3 — Build System & Tooling
