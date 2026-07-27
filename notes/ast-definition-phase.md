# AST Definition — Phase Mapping

> When and where Abstract Syntax Tree (AST) nodes are formally defined
> for each language construct.

## Phasing

| Layer | Where | Phase |
|-------|-------|-------|
| **Semantic primitives** (9 primitives) | `PRIMITIVE_BLOCKS.md` — semantic spec, illustrative syntax only | **Phase 3** ✅ |
| **Derived features (semantics)** | `what/concepts/*.md` + per-feature EDR | **Phase 4** ⬜ |
| **Formal AST nodes** (all constructs) | `SYNTAX.md` + `PARSER.md` — concrete grammar, node types, parser invariants | **Phase 5** ⬜ |
| **Typed IR nodes** (post-semantic) | `IR.md` — typed AST mirroring source structure | **Phase 5** ⬜ |

## Key Distinction

- Phase 3 answers **WHAT** operations exist (semantic primitives) — each
  primitive implicitly implies a corresponding AST node type (e.g.
  `literal` → `LiteralExpr`, `call` → `CallExpr`).
- Phase 5 answers **HOW** those operations are represented in source text
  and AST — formal grammar rules, node type definitions, parser structure.

## Current Status

- `SYNTAX.md` — placeholder (`⚠️ DRAFT — Placeholder for Phase 5`)
- `PARSER.md` — architecture and invariants defined, but no concrete
  grammar or node type list yet
- `IR.md` — principles defined, node types deferred

## References

- `ROADMAP.md` § Phase 3 / Phase 5
- `PRIMITIVE_BLOCKS.md` (EDR-016) — 9 primitive specifications
- `PARSER.md` — parser architecture
- `IR.md` — intermediate representation
