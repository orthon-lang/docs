# Repository-Wide Cross-Reference Link Audit

**Date:** 2026-07-20
**Scope:** All `*.md` files under `docs/` (excluding `.planning/` and `.archived/`)

## Audit Method

Links were checked by:
1. Grepping for `[text](path)` markdown link patterns across all `.md` files
2. Grepping for `**See also:**` patterns
3. Resolving each link relative to the source file's directory
4. Verifying the target file exists

## Total Links Checked

~120 cross-reference links across all concept documents, architecture specs, EDRs, and meta-documents.

## Links Fixed

| File | Old Link | New Link | Reason |
|------|----------|----------|--------|
| `docs/notes/imperative-crutch-lazy-sequences.md` | `EXECUTION_IMAGE.md` | Removed | Superseded by EXECUTION_PROGRAM.md |
| `docs/notes/imperative-crutch-resource-management.md` | `EXECUTION_IMAGE.md` | `EXECUTION_PROGRAM.md` | Superseded concept |

## Links Verified Correct

All other cross-reference links were verified to resolve correctly, including:
- Architecture docs → each other (IR.md, PARSER.md, TYPE_SYSTEM.md, NAME_RESOLUTION.md, ARCHITECTURE.md)
- Concept docs → DESIGN_PRINCIPLES.md and related concepts
- EDRs → related records and concept documents
- AGENTS.md → all referenced documents
- GLOSSARY.md → all source document links
- Strategy docs → Implementation Policies and architecture docs

## Links Still Broken

**None.** All cross-reference links resolve to existing files.

## Link-Validation Approach for Future Changes

### Recommended automated tool
Use a markdown link checker such as:
- `markdown-link-check` (npm) — CLI tool that validates all links in markdown files
- `lychee` (Rust) — fast, multi-threaded link checker

### Manual pre-commit procedure
Before committing a change that modifies or adds cross-references:
1. Run `grep -rn '\[.*\](.*\.md)' docs/ --include='*.md'` to list all links
2. For each new or modified link, verify the target file exists relative to the source
3. Verify that renamed files have all inbound links updated in the same commit

### Convention
When renaming a file, search for all inbound links and update them in the **same commit**. This is a non-negotiable project rule:
- Search pattern: `OLD_FILENAME.md` across all `.md` files
- Every match must be updated to the new filename
- Exception: links in `.planning/` and `.archived/` may remain if they refer to the file's historical identity

### CI integration (future)
Once a CI pipeline exists for this repository, add a link-check step:
```yaml
- name: Check links
  run: npx markdown-link-check docs/**/*.md --quiet
```
