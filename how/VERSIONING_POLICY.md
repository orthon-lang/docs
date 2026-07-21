# Versioning & Change-Log Policy

> **Last updated:** 2026-07-20

## Scope

This policy applies to large monolithic documents in the Orthon specification:

| Document | Path |
|----------|------|
| Execution Program | `docs/how/concepts/research/EXECUTION_PROGRAM.md` |
| Glossary | `docs/what/GLOSSARY.md` |
| Architecture | `docs/how/architecture/ARCHITECTURE.md` |
| Design Principles | `docs/what/DESIGN_PRINCIPLES.md` |
| Agent Guide | `docs/AGENTS.md` |

## Change-Log Format

Each large document carries a change-log section at the bottom tracking:

```markdown
## Change Log

| Date | Change | Affected Sections |
|------|--------|-------------------|
| YYYY-MM-DD | Brief description of the change | §§ Section names |
```

## Revision Levels

| Level | When | Action Required |
|-------|------|-----------------|
| **Major revision** | Semantic change to content (rule added, definition changed, policy updated) | Add change-log entry + update `> **Last updated:**` date stamp |
| **Minor revision** | Typo fix, formatting improvement, clarification without semantic change | Update `> **Last updated:**` date stamp only; no change-log entry required |
| **Structural revision** | Document reorganisation, section addition/removal, significant expansion | Add change-log entry referencing the EDR or phase that drove the change + update date stamp |

## Freshness Policy

Any large document with `> **Last updated:**` more than **90 days** in the past
should be flagged for review at the next phase boundary. The responsible
author reviews whether the document remains accurate and up to date.

## Enforcement

- Date-stamp freshness is tracked by the **Document Freshness** fitness function
- Change-log entries are verified during phase completion checklists
- Documents without a change-log section should have one added at the next major revision
