# Exploring the Docs with graphify

> **Last updated:** 2026-07-26

## Issue (Why)

This repository is large and cross-referenced: 91+ Markdown files spread
across `why/`, `how/`, `what/`, and `when/`, linked by relative Markdown
links, shared terminology (`what/GLOSSARY.md`), and Engineering Decision
Records that reference concepts, principles, and each other. A new
contributor — human or AI agent — orienting for the first time, or an
agent trying to answer "which documents touch mutability?" or "what
depends on the Execution Program concept?", has no fast way to see the
whole shape of the corpus without reading most of it.

`graphify` (`~/.claude/skills/graphify/SKILL.md`) turns any folder of
files into a queryable knowledge graph — nodes for concepts, documents,
and terms; edges for the relationships between them; community detection
that surfaces clusters of related documents. Run against this repository,
it gives a navigable map of the specification instead of a flat file
list.

This document explains how to use it against this docs repository
specifically. It does not replace `AGENTS.md` (the document map and
agent contribution protocol) — it complements it as an exploration tool
for questions the static document map cannot answer.

## Principles

- **Exploration tool, not a source of truth.** The graph is a derived,
  regenerable artifact. The Markdown files in `why/`, `how/`, `what/`,
  and `when/` remain the only authoritative specification. If the graph
  and the documents disagree, the documents win — rebuild the graph.
- **Not committed.** `/graphify-out/` is already listed in
  [`.gitignore`](../.gitignore). Generated graphs, reports, and vault
  exports stay local; they are not project deliverables and are not
  reviewed like specification content.
- **Read-only with respect to the specification.** Running graphify
  against this repository never edits `why/`, `how/`, `what/`, or
  `when/` content — it only reads those files to build the graph. Any
  actual documentation change discovered while exploring still goes
  through the normal `AGENTS.md` §5 Agent Workflow (Orient → Design →
  Gate → Write) and the commit-prefix conventions in `AGENTS.md` §10.10.

## Model (What)

### Building the graph

From the repository root (`docs/`):

```
/graphify .
```

This scans the whole repository (Markdown only — there is no code here),
extracts entities and relationships from each document, clusters them
into communities, and writes three outputs to `graphify-out/` (already
git-ignored):

- `graph.html` — interactive graph, open in a browser
- `GRAPH_REPORT.md` — plain-language audit report (god nodes, surprising
  connections, suggested questions)
- `graph.json` — raw graph data used by `query` / `path` / `explain`

To scope the graph to a single layer instead of the whole repository —
useful when iterating on one area, e.g. concept research — pass a
subpath:

```
/graphify how/concepts/research
```

### Querying the graph

Once `graphify-out/graph.json` exists, ask questions directly instead of
rebuilding:

```
/graphify query "which documents discuss mutability?"
/graphify path "Ownership" "Execution Program"
/graphify explain "Data Modifier"
```

- `query` does a broad traversal — good for "what touches X?" questions.
- `path` finds the shortest connection between two named concepts —
  good for "how does X relate to Y?" questions, e.g. tracing an EDR back
  to the Manifesto principle it satisfies.
- `explain` gives a plain-language summary of a single node and what
  connects to it.

### Refreshing after edits

The docs change frequently (new concept research, new EDRs, glossary
updates). Re-run incrementally instead of a full rebuild:

```
/graphify . --update
```

This re-extracts only new or changed files and merges them into the
existing graph.

### Example questions for this repository

- "Which concept research documents are tagged essential tier but have
  no corresponding EDR yet?"
- "What connects `why/MANIFESTO.md` to `how/DESIGN_PRINCIPLES.md`?"
- "Which documents reference the Execution Program model?"
- "What are the most-referenced (god) nodes in `how/concepts/research/`?"

## Default Strategy

Run `/graphify .` once per exploration session (or after a batch of
documentation changes), then use `query` / `path` / `explain` against
the cached graph for follow-up questions instead of rebuilding each
time. Use `--update` to keep the graph current without a full rebuild.

## Alternative Strategies

- **Scoped subgraphs.** Build against a single layer directory
  (`how/concepts/research/`, `how/decision_records/`) when exploring one
  area in depth — faster to build, easier to read the community
  breakdown.
- **Obsidian vault export.** Pass `--obsidian` to generate a
  browsable Obsidian vault (one file per node) instead of, or alongside,
  the default HTML/JSON outputs, if a persistent local knowledge base is
  preferred over a one-off graph.

## Open Questions

- Whether a periodic (e.g. milestone-boundary) graphify run should be
  added as a recommended step in `AGENTS.md` §5 Agent Workflow, or
  remain purely opt-in exploration tooling as described here.

## Decision History

None yet — this document introduces the practice; no prior alternative
was rejected.

---

**Affected documents checklist:**

- [x] `README.md` — links to this guide from "For AI Agents"
- [x] `AGENTS.md` — document map entry added
- [ ] `what/GLOSSARY.md` — no new terminology introduced
- [ ] `how/DESIGN_PRINCIPLES.md` — not applicable (tooling guide, not a
  language design principle)
