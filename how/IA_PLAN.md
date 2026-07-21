# Information Architecture Plan

> **Last updated:** 2026-07-20

## Current State

| Directory | File Count | Description |
|-----------|------------|-------------|
| `docs/how/concepts/research/` | 30+ | Concept research analyses (drafts) |
| `docs/how/architecture/` | 6 | Architecture specifications |
| `docs/how/strategies/` | 4 | Implementation strategies |
| `docs/how/decision_records/` | 15 categories | Engineering Decision Records |
| `docs/how/gates/` | 2+ | Design validation gates |
| `docs/notes/` | 13 | Research notes and analyses |

## Scaling Strategy

### Short-Term (Phase 3 completion, ~30-35 research files)

Keep `docs/how/concepts/research/` as a flat directory. A flat structure is
navigable up to ~40 files and avoids the complexity of nested subdirectories.
At 30-35 files, the single directory is still scannable.

### Medium-Term (Post-Freeze, ~15-20 accepted concepts in what/concepts/)

If `what/concepts/` exceeds 20 files, consider subdirectories by category
for accepted concepts only. Research stays flat in `how/concepts/research/`:

```
what/concepts/
├── core/           # Core language concepts
├── control/        # Control flow concepts
├── data/           # Data-related concepts
└── meta/           # Meta-programming concepts
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
| FOUNDATIONAL_ABSTRACTIONS.md | Foundational abstractions hypothesis: Data and Data Modifiers |
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
