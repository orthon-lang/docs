# TDR-010: Process Inventory

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Process

**Milestone:** 1

---

### Context

As the project's process infrastructure grows (gates, methods,
fitness functions, policies, checklists, procedures), it becomes
harder to see the full picture. Each tool is documented in its
own file, but there is no single map of what tools exist, how
they relate, and what each one prevents.

Without a process-lens view, new contributors face high onboarding
friction, and existing contributors may lose sight of tools that
are rarely used but still important.

### Decision

Adopt `PROCESS_INVENTORY.md` — a catalogue of all process tools
and approaches, providing a process-lens counterpart to
`LANGUAGE_INVENTORY.md`.

The inventory includes for each tool: name and type, TDR reference
(for full rationale), what failure mode it prevents, what we lose
without it, and which milestones it applies to.

Additionally, a dependency graph shows how tools relate, and a
coverage map shows which tools are active at each milestone.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Tool blind spots | Medium | Important but rarely-used tools are forgotten |
| Onboarding friction | Medium | New contributors must discover tools by reading all docs |
| Process fragmentation | Low | No single view of how tools fit together |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| All process tools in `docs/how/` | Language concepts (see LANGUAGE_INVENTORY.md) |
| | External tools (git, editor, etc.) |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-001–TDR-009 | Each tool's full rationale is in its TDR |
| LANGUAGE_INVENTORY.md | Process-lens counterpart to language-lens inventory |
| TDR-008 (TDR System) | Process Inventory aggregates all TDRs into one view |

### Consequences

- **Positive:**
  - Single entry point for understanding the process
  - "What do we lose?" column forces clarity about each tool's value
  - Dependency graph reveals implicit relationships
- **Negative:**
  - Must be kept in sync as tools are added/deprecated
  - Duplicates some information from individual TDRs

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| No inventory — rely on file listing | Not discoverable; no relationships visible |
| Auto-generated from TDRs | Premature automation for a small catalogue |
| Embed in ROADMAP | ROADMAP is about milestones, not tool catalogue |

### Evolution

- Inventory is **updated** whenever a tool is added, deprecated,
  or significantly changed.
- Format may **evolve** from Markdown table to a more structured
  format if the catalogue grows beyond ~30 tools.

### Affected Documents

- [x] `PROCESS_INVENTORY.md`
- [x] `ROADMAP.md` (Milestone 1 deliverables)
