# Information Architecture Plan

> **Last updated:** 2026-07-20

## Current State

| Directory | File Count | Description |
|-----------|------------|-------------|
| `docs/what/concepts/` | 22 | Language concept documents |
| `docs/how/architecture/` | 6 | Architecture specifications |
| `docs/how/strategies/` | 4 | Implementation strategies |
| `docs/how/decision_records/` | 15 categories | Engineering Decision Records |
| `docs/how/gates/` | 2+ | Design validation gates |
| `docs/notes/` | 13 | Research notes and analyses |

## Scaling Strategy

### Short-Term (Phase 3 completion, ~30-35 concept files)

Keep `docs/what/concepts/` as a flat directory. A flat structure is
navigable up to ~40 files and avoids the complexity of nested subdirectories.
At 30-35 files, the single directory is still scannable.

### Medium-Term (Post-Freeze, ~50+ files)

If `what/concepts/` exceeds 40 files, consider subdirectories by category:

```
what/concepts/
├── core/           # Core language concepts (CORE_CONCEPTS, DATA_MODEL, etc.)
├── control/        # Control flow concepts (FUNCTIONS, PATTERN_MATCHING, etc.)
├── data/           # Data-related concepts (ALLOCATION, OWNERSHIP, etc.)
└── meta/           # Meta-programming concepts (METAOBJECTS, LLM_TOOLCHAIN, etc.)
```

Subdirectory organization should be decided by usage patterns — the goal
is to reduce scanning time, not to achieve perfect taxonomy.

### New Top-Level Directories

| Directory | Purpose | When |
|-----------|---------|------|
| `docs/what/stdlib/` | Standard Library specifications | Post-Freeze (Milestone 8) |
| `docs/what/tooling/` | Build system and developer tooling specs | Post-Freeze (Milestone 9) |
| `docs/what/llm/` | LLM toolchain specifications (Schema Provider, Code Completer, Code Generator, Static Analyser) | Post-Freeze (Milestone 8) |

## File Naming

- **Content documents:** `UPPER_CASE.md` — one file, one coherent topic
- **Templates:** `_kebab-case.md` with underscore prefix
- **Decision records:** `EDR-NNN-title-with-dashes.md`
- **No "Miscellaneous" or "Various" files**

## Directory Index Files

When any top-level directory exceeds 10 files, maintain an `INDEX.md`
listing all files with brief descriptions:

```markdown
# Directory Index: concepts/

| File | Description |
|------|-------------|
| CORE_CONCEPTS.md | Core language abstractions: Data and Data Modifiers |
| DATA_MODEL.md | Formal data model: types and representations |
| ... | ... |
```

Pattern sourced from `how/decision_records/INDEX.md`.

## Search Strategy

For directories exceeding 50 files, prefer a directory-level INDEX.md
rather than nesting. INDEX.md should be generated or updated automatically
at phase boundaries.

## Cross-Reference Growth

As the directory count grows:
- Each top-level directory should maintain its own INDEX.md listing all files
- Cross-directory references should use relative paths from the source document's location
- The Glossary (`what/GLOSSARY.md`) serves as the unified entry point for term discovery
