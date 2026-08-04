# Tooling Requirements

> Captures forward-looking tooling, ecosystem, and LLM Agent UX
> requirements that arise during language design. These are not
> language concepts — they belong in the implementation repo
> (M3 — Build System & Tooling, M4 — Implementation) but must
> be recorded now because they influence the spec and would be
> lost between spec freeze and implementation.

---

## Purpose

During language design, ideas for developer tooling, LLM agent tooling,
and ecosystem infrastructure emerge from:

- **Gap analyses** — comparisons with other languages (BAML, etc.)
- **Concept Design Reviews** — a new language concept often implies a
  tooling counterpart
- **Ad-hoc insights** — during research, writing, or discussion

Each such insight is captured as a **Tooling Requirement** document
in this directory. These documents are:

- **Forward-looking** — they inform the spec but are implemented later
- **Spec-impacting** — they may require specific language features (e.g.,
  queryable AST for `baml describe`-style tools)
- **Status-tracked** — each entry has a status field (open / deferred /
  incorporated) so nothing is forgotten

---

## When to Create a Tooling Requirement

1. **Ad-hoc** — during any language design work, if you think "this
   concept implies a tool", create a `.md` file here.
2. **In Concept Design Review** — after step 5 (EDR), check whether the
   concept has tooling implications and create a file if needed.

---

## Process

```
Insight arises → create tooling-requirement.md → 
  → if spec-impacting → update affected spec documents →
  → status: OPEN →
  → at M3 (Tooling) → reviewed and actioned →
  → status: INCORPORATED or DEFERRED
```

---

## Directory Contents

| File | Status | Priority | Source |
|------|--------|----------|--------|
| [`LANGUAGE_SERVER.md`](LANGUAGE_SERVER.md) | Open | P1 | D serve-d / code-d case study |
| [`ORTHON-DESCRIBE.md`](ORTHON-DESCRIBE.md) | Open | P2 | BAML gap analysis § Tier 2 (#8) |
| [`ORTHON-RUN.md`](ORTHON-RUN.md) | Open | P2 | BAML gap analysis § Tier 2 (#9) |
| [`ORTHON-RUN-E.md`](ORTHON-RUN-E.md) | Open | P3 | BAML gap analysis § Tier 2 (#10) |
| [`ORTHON-PACK.md`](ORTHON-PACK.md) | Open | P3 | BAML gap analysis § Tier 2 (#11) |
| [`ORTHON-EVAL.md`](ORTHON-EVAL.md) | Open | P3 | BAML gap analysis § Tier 2 (#12) |
| [`ORTHON-SCOPE-MOCK.md`](ORTHON-SCOPE-MOCK.md) | Open | P3 | BAML gap analysis § Tier 2 (#13) |

---

## Growth Points

The current design is intentionally minimal. The following extensions
are natural evolution candidates as the project matures:

1. **EDR for the tooling requirements process itself** — if the
   number of entries grows significantly, consider a Process-category
   EDR formalising the collection → review → handoff cycle.
2. **Fitness functions for tooling** — some tooling requirements
   could be expressed as fitness functions for the implementation repo
   (e.g., "IR must support queryable symbol references").
3. **Cross-referencing with LANGUAGE_INVENTORY.md** — a matrix
   mapping tooling requirements to language concepts (which tools
   depend on which concepts) could help identify concept gaps.
4. **Status automation** — if the catalogue grows beyond ~20 entries,
   consider a simple status-tracking table (e.g., in a dedicated
   INDEX.md with YAML frontmatter for each entry).
5. **M3 transition protocol** — define a formal step in the M1→M3
   handoff: "archive frozen spec → review `how/tooling/` → prioritise
   by spec impact → create M3 plan."

## See Also

- [`_tooling-requirement.md`](_tooling-requirement.md) — template for new entries
- [`../../when/ROADMAP.md`](../../when/ROADMAP.md) § M3 — Build System & Tooling
- [`../PROCESS_INVENTORY.md`](../PROCESS_INVENTORY.md) — process tool catalogue
- [`../concept-design-review.md`](../concept-design-review.md) — tooling implications step
