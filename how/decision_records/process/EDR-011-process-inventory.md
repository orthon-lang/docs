# EDR-011: Process Inventory

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

**Supersedes:** TDR-010

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

The inventory includes for each tool: name and type, EDR reference
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

### Consequences

- **Positive:**
  - Single entry point for understanding the process
  - "What do we lose?" column forces clarity about each tool's value
  - Dependency graph reveals implicit relationships
- **Negative:**
  - Must be kept in sync as tools are added/deprecated
  - Duplicates some information from individual EDRs

### Evolution

The inventory is updated whenever a new EDR is created or an existing
one is deprecated.

### Compliance

The inventory's accuracy is verified by cross-referencing with the
EDR INDEX.md.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| No inventory — rely on file listing | Not discoverable; no relationships visible |
| Auto-generated from EDRs | Premature automation for a small catalogue |
| Embed in ROADMAP | ROADMAP is about milestones, not tool catalogue |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-001 through EDR-010 | Each tool's full rationale is in its EDR |
| EDR-001 (EDR System) | Process Inventory is the aggregated view of all Process-category EDRs |
| INDEX.md | Cross-reference for completeness |
