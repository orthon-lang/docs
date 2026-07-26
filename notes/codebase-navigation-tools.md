# Codebase Navigation Tools

> Tools and skills for navigating the Orthon docs codebase (~122 files, ~112K words).

---

## 1. Knowledge Graph: `/graphify`

Already built — `graphify-out/` contains 349 nodes, 346 edges, 86 communities.

**Commands:**

| Command | Purpose |
|---------|---------|
| `/graphify query "<question>"` | BFS traversal — broad context |
| `/graphify query "<question>" --dfs` | DFS — trace a specific path |
| `/graphify query "<question>" --budget 1500` | Cap answer at N tokens |
| `/graphify path "NodeA" "NodeB"` | Shortest path between two concepts |
| `/graphify explain "NodeName"` | Plain-language explanation of a node |
| `/graphify <path> --update` | Incremental — re-extract only new/changed files |
| `/graphify <path> --cluster-only` | Re-run clustering on existing graph |

**Viewers:** `graphify-out/graph.html` (interactive), `graphify-out/graph.json` (GraphRAG-ready), `graphify-out/GRAPH_REPORT.md` (summary).

---

## 2. Codebase Intel: `/gsd-map-codebase`

Parallel mapper agents that produce structured analysis in `.planning/codebase/`.

| Flag | Purpose |
|------|---------|
| (bare) | Full codebase map |
| `--fast` | Quick lightweight scan |
| `--query` | Search already-mapped intel |

---

## 3. Universal Context Entry: `/gsd-ns-context`

Dispatches to the right skill based on intent:

| Goal | Routes to |
|------|-----------|
| Full codebase map | `gsd-map-codebase` |
| Quick scan | `gsd-map-codebase --fast` |
| Query mapped intel | `gsd-map-codebase --query` |
| Knowledge graph | `gsd-graphify` |
| Update docs | `gsd-docs-update` |
| Extract learnings from phase | `gsd-extract-learnings` |
| Recall prior decisions | `gsd-mempalace-recall` |
| File artifact into MemPalace | `gsd-mempalace-capture` |

**Usage:** `$gsd-ns-context` + your question/intent.

---

## 4. Decision History: `/gsd-mempalace-recall`

Recall past decisions, patterns, and surprises before starting new work. Essential before planning — avoids re-litigating settled questions across the 110+ research files.

---

## 5. Socratic Exploration: `/gsd-explore`

Think through ideas before committing to plans. Good when unsure which angle to approach a problem from.

---

## 6. Stats & Health

| Tool | Purpose |
|------|---------|
| `gsd-stats` | Project statistics: phases, plans, git metrics, timeline |
| `gsd-health` | Diagnose and repair `.planning/` directory health |

---

## 7. Cross-Project Knowledge Sync

| Tool | Purpose |
|------|---------|
| `wiki-update` | Sync orthon-lang knowledge into Obsidian wiki |
| `wiki-query` | Search compiled wiki from any project |

---

## 8. VS Code Native Tools

| Tool | Purpose |
|------|---------|
| `grep_search` | Regex search across workspace |
| `file_search` | Glob pattern file lookup |
| `get-search-view-results` | Get VS Code Search view results |
| `vscode_listCodeUsages` | Find symbol references across files |

---

## Quick Start Flow

1. **New question about codebase:** `/graphify query "..."` (fast, existing graph)
2. **Before planning a phase:** `gsd-mempalace-recall` + `gsd-map-codebase --fast`
3. **Deep dive into a topic:** `gsd-ns-context` + topic
4. **Need orientation:** Open `graphify-out/graph.html` for visual cluster map
