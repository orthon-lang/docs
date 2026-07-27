---
phase: quick-260727-flc
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [how/decision_records/architecture/EDR-011-llm-generability-gate.md, how/decision_records/architecture/EDR-014-llm-generability-gate.md, how/gates/DECISION_VALIDATION.md, how/gates/_language-design.md, how/architecture/FITNESS_FUNCTIONS.md, how/decision_records/architecture/EDR-013-semantic-model.md, notes/llm-generability-gate.md, TODO.md, how/decision_records/INDEX.md]
autonomous: true
requirements: [DEBT-13]

must_haves:
  truths:
    - "The orphaned architecture record formerly misfiled as 'EDR-011' now carries the unique ID EDR-014 and is discoverable in INDEX.md (both the All Records table and the Architecture category sub-table)."
    - "EDR-011 refers unambiguously, everywhere in the live documentation (excluding .planning/ historical artifacts), to the Process-category 'Process Inventory' record."
    - "No live markdown file cites 'EDR-011' when referring to the LLM Generability Gate content or links to a file named 'EDR-011-llm-generability-gate.md'."
    - "EDR-013's 'Relationship to Other Records' table cites EDR-014, resolving the ambiguity flagged by 02-VERIFICATION.md (lines 79-91)."
  artifacts:
    - how/decision_records/architecture/EDR-014-llm-generability-gate.md
    - how/decision_records/INDEX.md
  key_links:
    - "how/decision_records/INDEX.md All Records + Architecture category tables <-> how/decision_records/architecture/EDR-014-llm-generability-gate.md"
    - "how/architecture/FITNESS_FUNCTIONS.md Source link <-> renamed EDR-014 file path"
    - "how/decision_records/architecture/EDR-013-semantic-model.md Relationship-to-Other-Records table <-> EDR-014 (not stale EDR-011 citation)"
    - "how/gates/DECISION_VALIDATION.md and how/gates/_language-design.md See-also/Decision-Journal citations <-> EDR-014-llm-generability-gate"
---

<objective>
Resolve a duplicate EDR-ID collision: two files both claim "EDR-011" —
`how/decision_records/process/EDR-011-process-inventory.md` (canonical, correctly
indexed) and `how/decision_records/architecture/EDR-011-llm-generability-gate.md`
(an orphaned duplicate never listed in INDEX.md, predating Phase 2). This was
surfaced by `02-VERIFICATION.md` because EDR-013's "Relationship to Other Records"
table cites "EDR-011 (LLM Generability Gate)" ambiguously against the authoritative
journal. The next free EDR number per INDEX.md is EDR-014.

Purpose: A specification whose own decision journal has two records sharing one ID
violates the project's "all cross-references valid" Core Value and undermines the
EDR system's traceability guarantee (EDR-001).

Output: The architecture record renamed to `EDR-014-llm-generability-gate.md` with a
matching heading, every live cross-reference to it repointed to EDR-014, and
INDEX.md updated to list EDR-014 as a first-class entry (All Records table,
Architecture category sub-table, Status Summary count, footer date).

Explicitly out of scope (per calling context): anything under `.planning/`,
including `.planning/PROJECT.md`, `.planning/REQUIREMENTS.md`, and
`.planning/codebase/STRUCTURE.md` — these are historical planning-session records,
analogous to git history, and are not touched even where they happen to mention
"EDR-011" (e.g. `.planning/REQUIREMENTS.md`'s "Orthon for LLM" section heading).
`.planning/STATE.md`'s own "Quick Tasks Completed" table update is handled
separately by the orchestrator at the end of the quick workflow.
</objective>

<execution_context>
@$HOME/.claude/gsd-core/workflows/execute-plan.md
@$HOME/.claude/gsd-core/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@how/decision_records/INDEX.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Renumber the orphaned architecture EDR from EDR-011 to EDR-014</name>
  <files>how/decision_records/architecture/EDR-011-llm-generability-gate.md, how/decision_records/architecture/EDR-014-llm-generability-gate.md</files>
  <action>
    Rename the file with `git mv how/decision_records/architecture/EDR-011-llm-generability-gate.md how/decision_records/architecture/EDR-014-llm-generability-gate.md` (preserves git history via rename detection — do not delete and recreate).

    Then edit line 1 of the renamed file: change the heading
    "# EDR-011: LLM Generability Gate" to "# EDR-014: LLM Generability Gate".
    This is the file's only internal self-reference to its own ID — the body text
    (Context, Decision, Consequences, Alternatives, See-also sections) does not
    reference "EDR-011" anywhere else, so no further edits are needed inside this
    file.
  </action>
  <verify>
    <automated>test -f how/decision_records/architecture/EDR-014-llm-generability-gate.md && test ! -f how/decision_records/architecture/EDR-011-llm-generability-gate.md && head -1 how/decision_records/architecture/EDR-014-llm-generability-gate.md | grep -q "^# EDR-014: LLM Generability Gate$" && test $(grep -c "EDR-011" how/decision_records/architecture/EDR-014-llm-generability-gate.md) -eq 0 && echo PASS</automated>
  </verify>
  <done>File renamed via `git mv` (history preserved as a rename, not delete+create); the old path no longer exists; the new file's heading reads "# EDR-014: LLM Generability Gate"; zero remaining occurrences of "EDR-011" inside the file.</done>
</task>

<task type="auto">
  <name>Task 2: Repoint every live cross-reference to the renamed EDR-014 file</name>
  <files>how/gates/DECISION_VALIDATION.md, how/gates/_language-design.md, how/architecture/FITNESS_FUNCTIONS.md, how/decision_records/architecture/EDR-013-semantic-model.md, notes/llm-generability-gate.md, TODO.md</files>
  <action>
    Depends on Task 1 having renamed the file first. Make these six single-line edits
    (each file has exactly one occurrence of "EDR-011", confirmed by
    `grep -c "EDR-011"` per file before this task began):

    1. `how/gates/DECISION_VALIDATION.md` line 279: in
       "**See also:** `EDR-011-llm-generability-gate`, `../../how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`,"
       change the backtick-quoted filename `EDR-011-llm-generability-gate` to
       `EDR-014-llm-generability-gate`.

    2. `how/gates/_language-design.md` line 99: in the Decision Journal table row
       "| 2026-07-19 | LLM_GENERABILITY_GATE | Added as 7th gate criterion | EDR-011 — LLM-native pillar requires explicit LLM generability validation for every language construct |"
       change the `EDR-011` cell reference to `EDR-014`.

    3. `how/architecture/FITNESS_FUNCTIONS.md` line 125: in
       "**Source:** [EDR-011](../decision_records/architecture/EDR-011-llm-generability-gate.md); [\_language-design.md](../gates/_language-design.md) — LLM generability criterion"
       change both the link text `[EDR-011]` to `[EDR-014]` and the link target path
       `../decision_records/architecture/EDR-011-llm-generability-gate.md` to
       `../decision_records/architecture/EDR-014-llm-generability-gate.md`.

    4. `how/decision_records/architecture/EDR-013-semantic-model.md` line 235, in the
       "Relationship to Other Records" table: change the row
       "| EDR-011 (LLM Generability Gate) | The `LLM_GENERABILITY_GATE` criteria from that EDR were applied during this model's validation (see Consequences § Negative for the two open Flags). |"
       so its ID cell reads `EDR-014 (LLM Generability Gate)` instead of
       `EDR-011 (LLM Generability Gate)`. This is the exact ambiguity 02-VERIFICATION.md
       flagged.

    5. `notes/llm-generability-gate.md` line 3: in
       "**Status:** *Accepted — see EDR-011 for the formal decision.*"
       change `EDR-011` to `EDR-014`.

    6. `TODO.md` line 214: in the inline reference `` `EDR-011-llm-generability-gate.md`) ``
       change the filename to `` `EDR-014-llm-generability-gate.md`) ``.

    Do not touch any other line in these six files, and do not touch any file under
    `.planning/` even if it also mentions "EDR-011" in this context (e.g.
    `.planning/REQUIREMENTS.md` line 133's "beyond the already-shipped LLM
    Generability Gate, EDR-011" parenthetical — out of scope per the calling
    context).
  </action>
  <verify>
    <automated>for f in how/gates/DECISION_VALIDATION.md how/gates/_language-design.md how/architecture/FITNESS_FUNCTIONS.md how/decision_records/architecture/EDR-013-semantic-model.md notes/llm-generability-gate.md TODO.md; do test $(grep -c "EDR-011" "$f") -eq 0 || { echo "FAIL: $f still has EDR-011"; exit 1; }; test $(grep -c "EDR-014" "$f") -ge 1 || { echo "FAIL: $f missing EDR-014"; exit 1; }; done && echo PASS</automated>
  </verify>
  <done>All six files reference EDR-014 (not EDR-011) for the LLM Generability Gate; each file's other content is unchanged; EDR-013's Relationship-to-Other-Records table no longer cites an ambiguous EDR-011.</done>
</task>

<task type="auto">
  <name>Task 3: List EDR-014 in INDEX.md — All Records, Architecture category, Status Summary, footer date</name>
  <files>how/decision_records/INDEX.md</files>
  <action>
    Depends on Task 1 having renamed the file first (the link target must resolve).
    Make four edits to `how/decision_records/INDEX.md`:

    1. In the "## All Records" table, append a new row directly after the existing
       EDR-013 row (EDR-013 is dated 2026-07-27, the most recent entry; the table is
       not strictly date-ordered already — e.g. EDR-013 already appears after
       EDR-012 despite EDR-011 being dated earlier — so append EDR-014 last rather
       than inserting it chronologically):
       `| EDR-014 | Architecture | [LLM Generability Gate](architecture/EDR-014-llm-generability-gate.md) | Accepted | 2026-07-19 | — |`

    2. In "## By Category" → "### Architecture" sub-table, add a new row ordered by
       ID after the existing EDR-013 row:
       `| EDR-014 | [LLM Generability Gate](architecture/EDR-014-llm-generability-gate.md) | Accepted | 2026-07-19 |`
       (Architecture category table then reads EDR-010, EDR-012, EDR-013, EDR-014.)

    3. In "## Status Summary", change the "Accepted" row's count from `11` to `12`
       (one previously-unindexed Accepted record is now indexed; all other status
       rows — Proposed 0, Deprecated 0, Superseded 0 — are unaffected).

    4. Check the footer line at the bottom of the file: it should read
       "*Last updated: 2026-07-27*" (today's date). It already does as of plan-write
       time, so ordinarily no edit is needed here — but if execution happens on a
       later date, update the footer line to that later date instead.

    Do not alter the "## Gap Registry" section (EDR-008/EDR-009 remain Skipped /
    Reserved — unrelated to this fix) or any other row in either table.
  </action>
  <verify>
    <automated>grep -qE '^\| EDR-014 \| Architecture \| \[LLM Generability Gate\]\(architecture/EDR-014-llm-generability-gate\.md\) \| Accepted \| 2026-07-19 \| — \|$' how/decision_records/INDEX.md && grep -qE '^\| EDR-014 \| \[LLM Generability Gate\]\(architecture/EDR-014-llm-generability-gate\.md\) \| Accepted \| 2026-07-19 \|$' how/decision_records/INDEX.md && grep -qE '^\| Accepted \| 12 \|$' how/decision_records/INDEX.md && test $(grep -c "EDR-014" how/decision_records/INDEX.md) -eq 2 && test $(grep -c "| EDR-011 |" how/decision_records/INDEX.md) -eq 2 && echo PASS</automated>
  </verify>
  <done>INDEX.md lists EDR-014 in both the All Records table and the Architecture category sub-table, pointing at the renamed file; the Status Summary Accepted count reads 12; the two remaining EDR-011 rows (All Records + Process category) still correctly refer only to Process Inventory; the footer date reflects today.</done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| None | This plan edits eight Markdown documentation/decision-record files and performs one `git mv` rename. No code execution, no external input, no network calls, no package installation occurs. |

## STRIDE Threat Register

No threats identified. This plan has no trust boundary: it is a solo-authored,
offline correction of an ID-numbering collision across Markdown files already
tracked in git, with no user input processed, no code executed, and no dependency
installed. STRIDE categories do not apply to a documentation cross-reference fix of
this shape.
</threat_model>

<verification>
Run, from the repository root:

1. `test -f how/decision_records/architecture/EDR-014-llm-generability-gate.md && test ! -f how/decision_records/architecture/EDR-011-llm-generability-gate.md` — rename completed.
2. `head -1 how/decision_records/architecture/EDR-014-llm-generability-gate.md` returns `# EDR-014: LLM Generability Gate` — heading updated.
3. `for f in how/gates/DECISION_VALIDATION.md how/gates/_language-design.md how/architecture/FITNESS_FUNCTIONS.md how/decision_records/architecture/EDR-013-semantic-model.md notes/llm-generability-gate.md TODO.md; do grep -c "EDR-011" "$f"; done` returns `0` for every file in the list — no stale citations remain.
4. `grep -c "EDR-014" how/decision_records/INDEX.md` returns `2` (All Records + Architecture category rows) and `grep -E '^\| Accepted \| [0-9]+ \|$' how/decision_records/INDEX.md` returns `12`.
5. `grep -c "EDR-011" how/decision_records/process/EDR-011-process-inventory.md` is unchanged and still `>= 1` — the canonical Process EDR-011 was never touched.
6. `git log --follow --oneline how/decision_records/architecture/EDR-014-llm-generability-gate.md | head -5` shows history continuity from the pre-rename file (confirms `git mv` preserved history rather than delete+create).
</verification>

<success_criteria>
- Exactly one file in the repository claims the ID "EDR-011": `how/decision_records/process/EDR-011-process-inventory.md`.
- The formerly-orphaned architecture record is renamed to `EDR-014-llm-generability-gate.md`, heading-updated, and listed in INDEX.md's All Records table, Architecture category sub-table, and Status Summary count.
- Every live (non-`.planning/`) cross-reference to the old EDR-011 architecture file now points at EDR-014: `how/gates/DECISION_VALIDATION.md`, `how/gates/_language-design.md`, `how/architecture/FITNESS_FUNCTIONS.md`, `how/decision_records/architecture/EDR-013-semantic-model.md`, `notes/llm-generability-gate.md`, `TODO.md`.
- EDR-013's "Relationship to Other Records" table cites EDR-014 unambiguously, resolving the exact gap `02-VERIFICATION.md` (lines 79-91) flagged.
- `.planning/` files (including `.planning/REQUIREMENTS.md`'s incidental "EDR-011" mention in its "Orthon for LLM" section) are explicitly untouched by this plan, per the calling context's constraints — they are historical planning-session records, not live cross-references.
- The rename used `git mv`, so `git log --follow` on the new path shows continuous history from the original file rather than an orphaned new file.
</success_criteria>

<output>
Create `.planning/quick/260727-flc-renumber-duplicate-edr-011-architecture-/260727-flc-SUMMARY.md` when done
</output>