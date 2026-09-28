---
phase: quick-260928-fcr
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - how/syntax/GENERATOR_EXPRESSION_SYNTAX.md          # new (Stages 1–6b reasoning trail)
  - what/syntax/GENERATOR_EXPRESSION_SYNTAX.md         # new (Stage 8 accepted record)
  - what/SYNTAX.md                                     # edit (Stage 9 hub pointer)
  - what/syntax/README.md                              # edit (Stage 9 § Records)
  - how/syntax/README.md                               # edit (Stage 9 decision queue → Resolved)
  - how/architecture/PARSER.md                         # edit (Stage 10 grammar production)
  - how/decision_records/architecture/EDR-092-generator-expression-syntax.md  # edit (Stage 7 validation-trail cite)
autonomous: true
requirements: [EDR-092, EDR-087]

estimate:
  tokens: 60000
  raw_tokens: 40000
  tasks: 3
  confidence: low

must_haves:
  truths:
    - "A reader can open how/syntax/GENERATOR_EXPRESSION_SYNTAX.md and follow the full reasoning trail — Decision Pipeline Run, Coupling & Overload Check, the 5 Syntax Principles, the 4 required Syntax Acceptance Gates, the relevant Language Design Gate items, and the referenced (not new) Human Sign-off — with the doc marked Resolved."
    - "A reader can open what/syntax/GENERATOR_EXPRESSION_SYNTAX.md and find the canonical gen(...) accepted record (single form, nested gen(v for s in subs for v in s), the reserved-production rule, the emit-only / no-yield-from note), with a header declaring EDR-092 and the concept it renders (GENERATORS.md)."
    - "The provenance chain is complete: what/SYNTAX.md hub lists the record, what/syntax/README.md § Records lists it, and how/syntax/README.md decision queue marks the hypothesis Resolved."
    - "how/architecture/PARSER.md contains a concrete grammar production for the gen(...) generator expression consistent with a labeled grammar section."
    - "EDR-092 cites the how/syntax/GENERATOR_EXPRESSION_SYNTAX.md hypothesis trail as its validation trail (one added reference; decision, sign-off, and supersession fields unchanged)."
    - "Every new or edited cross-reference resolves to an existing file at the correct ../ depth (AGENTS §10.8)."
  artifacts:
    - how/syntax/GENERATOR_EXPRESSION_SYNTAX.md
    - what/syntax/GENERATOR_EXPRESSION_SYNTAX.md
    - what/SYNTAX.md
    - what/syntax/README.md
    - how/syntax/README.md
    - how/architecture/PARSER.md
    - how/decision_records/architecture/EDR-092-generator-expression-syntax.md
  key_links:
    - "how/syntax trail ↔ EDR-092 ↔ what/syntax record ↔ what/SYNTAX.md hub ↔ how/syntax queue — the five must reference each other consistently."
    - "Relative-link depth: what/syntax/ → EDR uses ../../how/decision_records/... and → concept uses ../concepts/...; how/syntax/ → what uses ../../what/... and → how siblings use ../... — depth errors are the most likely breakage."
---

<objective>
Back-fill the Syntax Pipeline (how/SYNTAX_PIPELINE.md, EDR-087) artifacts that the prior GSD run skipped for the already-LOCKED `gen(...)` generator-expression surface form (EDR-092). The DECISION is settled and unchanged — this plan produces the missing pipeline artifacts (Stages 1–10), it does NOT re-decide, does NOT create a new Human Sign-off, and does NOT re-run the `fn`→`fun` or `[T]`→`<T>` migrations (already committed).

Purpose: give `gen(...)` the same complete, traceable pipeline trail that RANGE (EDR-083), GENERICS (EDR-086), and INVOCATION (EDR-085) already have, so the v0.1 spec is self-consistent and every cross-reference resolves.

Output: one new how/syntax reasoning-trail doc, one new what/syntax accepted record, hub + queue provenance updates, a PARSER.md grammar production, and a one-line EDR-092 validation-trail citation — grouped into three atomic `docs:` commits.
</objective>

<execution_context>
@.claude/gsd-core/workflows/execute-plan.md
@.claude/gsd-core/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@AGENTS.md
@.claude/CLAUDE.md

# The pipeline being satisfied and its conventions
@how/SYNTAX_PIPELINE.md
@how/syntax/README.md
@what/syntax/README.md

# Precedents to mirror — DO read these before authoring
@what/syntax/RANGE_SYNTAX.md
@what/syntax/GENERICS_SYNTAX.md
@what/syntax/INVOCATION_SYNTAX.md
@how/syntax/TYPE_ANNOTATION_SYNTAX.md
@how/concepts/research/important/RANGE_STEP.md

# The decided record, the concept, the hub, the gates, the grammar target
@how/decision_records/architecture/EDR-092-generator-expression-syntax.md
@what/concepts/GENERATORS.md
@what/SYNTAX.md
@how/gates/DECISION_VALIDATION.md
@how/gates/_language-design.md
@how/process/DECISION_PIPELINE.md
@how/architecture/PARSER.md
</context>

<tasks>

<task type="tracer">
  <name>Task 1: Create how/syntax/GENERATOR_EXPRESSION_SYNTAX.md — the Stage 1–6b reasoning trail (Commit 1)</name>
  <files>how/syntax/GENERATOR_EXPRESSION_SYNTAX.md</files>
  <action>
Create the hypothesis / reasoning-trail doc using UPPER_SNAKE_SYNTAX.md naming, mirroring the structure of how/syntax/TYPE_ANNOTATION_SYNTAX.md and the "resolved reasoning trail" marker precedent of how/concepts/research/important/RANGE_STEP.md. This is the thin end-to-end tracer for the whole pipeline: it wires Stages 1 through 6b in one document so the remaining tasks expand from a proven trail shape.

Header blockquote: mark the doc Resolved per the RANGE_STEP.md precedent — a leading "Resolved via EDR-092 (2026-09-28)" line stating the accepted specification lives in what/syntax/GENERATOR_EXPRESSION_SYNTAX.md and the what/SYNTAX.md hub, and that this document is the reasoning trail (Syntax Pipeline Stages 1–6b). Include a See-also list. This is a forward reference to the what/syntax record created in Task 2 (same plan) — expected; the full-resolution link check runs in Task 3.

Sections, in order:
- "## Issue (Why)" — the bare parenthesised form carried no explicit production marker (indistinguishable at its opening delimiter from a parenthesised expression or tuple, per EDR-092 Context); an explicit reserved `gen(` marker improves learnability and LLM generability without adding semantics. State plainly that semantics are settled (EDR-021 `emit`, EDR-050 generator expressions, EDR-032 combinators) and only the surface form is chosen here.
- "## Decision Pipeline Run" (Stage 2) — reproduce the 10-question table format from TYPE_ANNOTATION_SYNTAX.md § Decision Pipeline Run. Verdict to record: this is SYNTACTIC SUGAR over already-accepted generator-expression semantics (EDR-050 / EDR-092); it adds no new semantics; it is not a library problem and not an optimisation. Answer each Q consistent with that verdict (Q2 Language; Q5 no new semantics — surface form only; Q7 yes, sugar over emit / stdlib combinators; Q8 not an optimisation). Close with a "Verdict: Pipeline PASS (sugar over settled constructs)" line. Frame it as provenance, not a new binding decision.
- "## Coupling & Overload Check" (Stage 3) — record the `gen` collision sweep. IMPORTANT: do NOT hardcode a predetermined "only Code gen" claim. Run the sweep during execution (see verify) and RECORD THE ACTUAL findings: (a) the `gen(` production form appears only in the generator docs (what/concepts/GENERATORS.md, what/concepts/LAZY_SEQUENCE_GENERATORS.md, what/CORE_CONCEPTS.md) — all the same construct; (b) the prose word `gen` also occurs in non-accepted layers as "Code gen" in how/concepts/research/deferrable/REFLECTION_ALTERNATIVES.md and as an `id + gen` generation-counter in notes/primitive-blocks-discussion.md. State the verdict: CLEAN — no `gen(` production ever carried another meaning; the prose occurrences are informal words in non-accepted (research / notes) layers, not the reserved production, so one-symbol-one-meaning holds for `gen(`.
- "## Syntax Principles" (Stage 4) — one-line verdict against each of the 5 principles from what/SYNTAX.md (one concept→one syntax; one symbol→one meaning; no significant whitespace; named before symbolic; syntax derived from semantics). All pass; note the named-before-symbolic principle is satisfied because `gen` is itself a named keyword marker.
- "## Syntax Acceptance Gates" (Stage 5) — the 4 REQUIRED gates from DECISION_VALIDATION.md § Gate Selection "Syntax change" row, one-line PASS verdict each: LOGICAL_CONSISTENCY, CONCEPTUAL_SIMPLICITY, ARCHITECTURAL_INTEGRITY, LLM_GENERABILITY. Add USER_VALUE as the applicable optional gate (purely syntactic sugar) with a one-line verdict. Note (per EDR-092 Gate Validation) that a full seven-gate table is not reproduced for a locked sugar decision over settled constructs.
- "## Language Design Gate (relevant items)" (Stage 6) — one-line verdict on the syntax-relevant _language-design.md items: named equivalence, all canonical forms documented, explicitness, orthogonality, LLM generability.
- "## Human Sign-off (Stage 6b)" — reference the EXISTING LOCKED sign-off VERBATIM: `Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED` (as recorded in EDR-092). State explicitly that this sign-off is pre-existing and is NOT re-created here (agents must not self-certify Stage 6b).
- "## Cross-References" — links to: how/syntax/README.md (queue), what/syntax/GENERATOR_EXPRESSION_SYNTAX.md (accepted record), what/SYNTAX.md (hub), what/concepts/GENERATORS.md (concept), the EDR-092 record, how/SYNTAX_PIPELINE.md, how/process/DECISION_PIPELINE.md, how/gates/DECISION_VALIDATION.md.

Relative-link depth from how/syntax/ (verify each): how siblings use `../` (e.g. `../SYNTAX_PIPELINE.md`, `../process/DECISION_PIPELINE.md`, `../gates/DECISION_VALIDATION.md`, `../concepts/research/deferrable/REFLECTION_ALTERNATIVES.md`); the what layer uses `../../what/...` (e.g. `../../what/SYNTAX.md`, `../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`, `../../what/concepts/GENERATORS.md`); decision records match the how/syntax/README.md precedent `../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md`. All content in English.

Then commit (Commit 1): `docs(quick-260928-fcr): add gen(...) syntax reasoning trail (pipeline Stages 1–6b)`.
  </action>
  <verify>
    <automated>test -f /home/user/docs/how/syntax/GENERATOR_EXPRESSION_SYNTAX.md && for m in "Decision Pipeline Run" "Coupling & Overload" "Syntax Principles" "Syntax Acceptance Gates" "Human Sign-off" "Verdict: LOCKED" "mniedre"; do grep -qF "$m" /home/user/docs/how/syntax/GENERATOR_EXPRESSION_SYNTAX.md || { echo "MISSING SECTION: $m"; exit 1; }; done; echo OK</automated>
    <automated>echo "record actual sweep:"; grep -rn 'gen(' /home/user/docs/what /home/user/docs/how /home/user/docs/notes 2>/dev/null; grep -rnw 'gen' /home/user/docs/what /home/user/docs/how /home/user/docs/notes 2>/dev/null</automated>
  </verify>
  <done>how/syntax/GENERATOR_EXPRESSION_SYNTAX.md exists with all Stage 1–6b sections; the Coupling & Overload Check records the actual grep sweep (both `gen(` production sites and prose-`gen` occurrences) and reaches a CLEAN verdict; the Human Sign-off is the pre-existing LOCKED line quoted verbatim, not a new sign-off; doc marked Resolved; Commit 1 made.</done>
</task>

<task type="auto">
  <name>Task 2: Create what/syntax/GENERATOR_EXPRESSION_SYNTAX.md accepted record + hub/queue provenance (Stages 8–9, Commit 2)</name>
  <files>what/syntax/GENERATOR_EXPRESSION_SYNTAX.md, what/SYNTAX.md, what/syntax/README.md, how/syntax/README.md</files>
  <action>
Create the accepted syntax record what/syntax/GENERATOR_EXPRESSION_SYNTAX.md (Stage 8), mirroring the structure of what/syntax/RANGE_SYNTAX.md, what/syntax/GENERICS_SYNTAX.md, and what/syntax/INVOCATION_SYNTAX.md exactly. Use CURRENT conventions throughout: `fun` (not `fn`) and `<T>` (not `[T]`), e.g. `fun () -> Iterator<Int>`.

Header blockquote: declare the deciding record and the concept it renders — "Accepted — EDR-092 (Generator Expression Syntax)." followed by "The canonical semantic specification lives in what/concepts/GENERATORS.md; this file records the surface form." (parallel to RANGE_SYNTAX.md's header).

Sections:
- "## Canonical forms" — an orthon code block showing the single form `gen(x * x for x in 1..10)`, the filtered form `gen(x for x in 1..100 if x % 2 == 0)`, the single-delegate form `gen(v for v in sub)`, and the nested / flattening form `gen(v for s in subs for v in s)`.
- "## Rules" — numbered, drawn from EDR-092 Decision items 1–8 and GENERATORS.md § Model: (1) single surface form `gen(...)`; `gen` is a reserved surface grammar production, not an identifier — the parser recognises `gen(` as opening a comprehension, never a call, and it cannot be assigned, shadowed, imported, or passed as a value; (2) the parenthesised contents are comprehension clause-grammar (`expr for pattern in src if cond`), not an argument expression; (3) scope is the comprehension form only, including nested / multi-clause `gen(v for s in subs for v in s)`; (4) pure sugar (Desugaring Policy) — desugars to stdlib combinators (`.filter` / `.map` / `.flat_map`) or an equivalent `emit`-based `fun`, introducing no new core primitive; (5) generators remain emit-only — no `yield`, no `yield from`; delegation is ordinary iteration + `emit` (equivalently a nested `gen(...)` clause); (6) rejected: `gen(sub)` wrapping a bare value (redundant synonym for `.iter()` / identity, breaks the "`gen(` always opens a comprehension" invariant) and the bare parenthesised form without a marker (withdrawn); (7) `emit` never appears inside lambdas / closures — this closes LAZY_SEQUENCE_GENERATORS Open Question 2 negatively. Cite the D-item / EDR reference inline where natural (this record implements EDR-092).
- "## Cross-References" — mirror RANGE_SYNTAX.md's cross-ref block: EDR-092 (deciding record), what/concepts/GENERATORS.md (full semantic spec), what/concepts/LAZY_SEQUENCE_GENERATORS.md (OQ2 closed), what/SYNTAX.md (hub), how/syntax/README.md (decision queue — resolved), how/syntax/GENERATOR_EXPRESSION_SYNTAX.md (reasoning trail).

Relative-link depth from what/syntax/ (verify each): EDR uses `../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md`; concept siblings use `../concepts/GENERATORS.md` and `../concepts/LAZY_SEQUENCE_GENERATORS.md`; hub uses `../SYNTAX.md`; how/syntax targets use `../../how/syntax/...`.

Stage 9 — hub + provenance edits (Edit, not Write — these are existing files):
- what/SYNTAX.md § "Accepted Syntax (hub)": add a table row `| Generator expression gen(...) | [what/syntax/GENERATOR_EXPRESSION_SYNTAX.md](syntax/GENERATOR_EXPRESSION_SYNTAX.md) | EDR-092 |`, matching the existing row format (hub-relative path is `syntax/...`).
- what/syntax/README.md § "Records": add a bullet `- [GENERATOR_EXPRESSION_SYNTAX.md](GENERATOR_EXPRESSION_SYNTAX.md) — generator expression gen(...) (EDR-092).` matching the existing bullet format.
- how/syntax/README.md § "Resolved (accepted — record in what/syntax/)": add a table row mirroring the existing resolved rows: `| Generator expression gen(...) | [GENERATOR_EXPRESSION_SYNTAX.md](../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md) | EDR-092 | Resolved | EDR-092 (accepted) |`.
- OPTIONAL, low-risk tidy in the same commit: in what/concepts/GENERATORS.md flip the compliance checklist item `- [ ] what/SYNTAX.md` to `- [x] what/SYNTAX.md` (the hub now points to this record). If done, add what/concepts/GENERATORS.md to this commit's files; skip if it introduces any ambiguity.

Then commit (Commit 2): `docs(quick-260928-fcr): add gen(...) accepted syntax record + hub/queue provenance`.
  </action>
  <verify>
    <automated>test -f /home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -qF 'EDR-092' /home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -qF 'gen(v for s in subs for v in s)' /home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && ! grep -qE '\bfn\b|\[T\]' /home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && echo OK</automated>
    <automated>grep -qF 'syntax/GENERATOR_EXPRESSION_SYNTAX.md' /home/user/docs/what/SYNTAX.md && grep -qF 'GENERATOR_EXPRESSION_SYNTAX.md' /home/user/docs/what/syntax/README.md && grep -qF '../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md' /home/user/docs/how/syntax/README.md && echo OK</automated>
  </verify>
  <done>what/syntax/GENERATOR_EXPRESSION_SYNTAX.md exists, declares EDR-092 + GENERATORS.md, shows canonical + nested forms, uses only `fun`/`<T>` conventions; the hub (what/SYNTAX.md), the records index (what/syntax/README.md), and the decision queue (how/syntax/README.md § Resolved) all reference the new record; Commit 2 made.</done>
</task>

<task type="auto">
  <name>Task 3: PARSER.md grammar production (Stage 10) + EDR-092 validation-trail cite (Stage 7) + full cross-reference resolution (Commit 3)</name>
  <files>how/architecture/PARSER.md, how/decision_records/architecture/EDR-092-generator-expression-syntax.md</files>
  <action>
PARSER.md currently contains NO grammar productions (it is prose-only), so introduce a clearly labeled grammar section rather than matching a non-existent notation. Add a new "## Grammar" section (place it after "## Key Design Decisions", before "## Relationships") containing the concrete generator-expression production in standard EBNF, consistent, self-describing notation. Target production (comprehension form only, per EDR-092 item 4):

`GeneratorExpr ::= "gen" "(" Expr ForClause ( ForClause | IfClause )* ")"` with `ForClause ::= "for" Pattern "in" Expr` and `IfClause ::= "if" Expr`.

This must accept `gen(x for x in 1..100 if x % 2 == 0)` and the nested `gen(v for s in subs for v in s)`. Add a one-line note that `gen` is a reserved production keyword (parser recognises `gen(` as the opening of a comprehension, never a call — EDR-092), and cross-link the accepted record what/syntax/GENERATOR_EXPRESSION_SYNTAX.md and EDR-092. Keep it minimal — do NOT add grammar for other constructs (out of scope). Do not disturb the existing "## Open Questions" whitespace-significance items.

Relative-link depth from how/architecture/: to what/syntax record use `../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`; to the EDR use `../decision_records/architecture/EDR-092-generator-expression-syntax.md`.

Stage 7 — EDR-092 validation-trail cite: in how/decision_records/architecture/EDR-092-generator-expression-syntax.md, LIGHTLY amend the "### Gate Validation" section to add ONE reference line citing how/syntax/GENERATOR_EXPRESSION_SYNTAX.md as the decision's validation trail (the hypothesis doc's Decision Pipeline Run + gate verdicts are the EDR's validation trail, per SYNTAX_PIPELINE.md Stage 7). Relative path from how/decision_records/architecture/ is `../../syntax/GENERATOR_EXPRESSION_SYNTAX.md`. Do NOT alter the Decision items, the Human Sign-off line, the Partially-supersedes / supersession fields, or any other content — one added line/reference only.

Full cross-reference resolution (AGENTS §10.8): after the edits, verify EVERY markdown link in all touched files resolves to an existing file at the correct depth — the two new docs (how/syntax/ and what/syntax/) and the four edited files (what/SYNTAX.md, what/syntax/README.md, how/syntax/README.md, PARSER.md, EDR-092). Use the link-resolution loop in verify below; fix any BROKEN path before committing.

Then commit (Commit 3): `docs(quick-260928-fcr): add gen(...) parser grammar + cite validation trail in EDR-092`.
  </action>
  <verify>
    <automated>grep -qF 'GeneratorExpr' /home/user/docs/how/architecture/PARSER.md && grep -qF '"gen"' /home/user/docs/how/architecture/PARSER.md && grep -qF '../../syntax/GENERATOR_EXPRESSION_SYNTAX.md' /home/user/docs/how/decision_records/architecture/EDR-092-generator-expression-syntax.md && echo OK</automated>
    <automated>bad=0; for f in how/syntax/GENERATOR_EXPRESSION_SYNTAX.md what/syntax/GENERATOR_EXPRESSION_SYNTAX.md what/SYNTAX.md what/syntax/README.md how/syntax/README.md how/architecture/PARSER.md how/decision_records/architecture/EDR-092-generator-expression-syntax.md; do d=$(dirname "/home/user/docs/$f"); grep -oE '\]\([^)]+\.md[^)]*\)' "/home/user/docs/$f" | sed -E 's/^\]\(//; s/\)$//; s/#.*$//' | while read -r p; do case "$p" in http*|/*) continue;; esac; [ -f "$d/$p" ] || echo "BROKEN in $f -> $p"; done; done | tee /tmp/claude-0/-home-user-docs/*/scratchpad/xref.txt 2>/dev/null; test ! -s /tmp/claude-0/-home-user-docs/*/scratchpad/xref.txt 2>/dev/null && echo "ALL LINKS RESOLVE"</automated>
  </verify>
  <done>PARSER.md has a labeled Grammar section with the GeneratorExpr production accepting single, filtered, and nested forms; EDR-092 Gate Validation cites the hypothesis trail (one added line, decision/sign-off/supersession untouched); every markdown link in all seven touched files resolves at the correct ../ depth; Commit 3 made.</done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| (none) | Documentation-only change. No executable surface, no runtime, no untrusted input, no network, no package installs. |

## STRIDE Threat Register

| Threat ID | Category | Component | Severity | Disposition | Mitigation Plan |
|-----------|----------|-----------|----------|-------------|-----------------|
| T-fcr-01 | Tampering (spec integrity) | cross-references / provenance chain | low | mitigate | Full markdown-link resolution check (Task 3 verify) plus the coupling-sweep record ensures no dangling or contradictory references are introduced. |
| T-fcr-SC | Tampering | dependency installs | n/a | accept | No npm/pip/cargo installs — pure Markdown edits in a documentation repository. No package-legitimacy gate applies. |
</threat_model>

<verification>
Phase-level checks after all three commits:

1. Both new files exist and follow UPPER_SNAKE_SYNTAX.md naming:
   `test -f /home/user/docs/how/syntax/GENERATOR_EXPRESSION_SYNTAX.md && test -f /home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`
2. The provenance chain is complete and bidirectional: the how/syntax trail, EDR-092, the what/syntax record, the what/SYNTAX.md hub, and the how/syntax/README.md queue all reference each other (spot-check with the grep asserts in Tasks 1–3).
3. No stale conventions leaked into the new accepted record:
   `! grep -qE '\bfn\b|\[T\]' /home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`
4. The Human Sign-off appears ONLY as a verbatim reference to the pre-existing LOCKED line — no new sign-off was authored (grep shows `mniedre` / `LOCKED` only in the reference context).
5. Every markdown link in all seven touched files resolves (Task 3 link-resolution loop prints `ALL LINKS RESOLVE`).
6. `git log --oneline` shows exactly three `docs(quick-260928-fcr): ...` commits, each ending with the session attribution trailer.
7. No changes under `.claude/` or `.planning/` (other than this plan file) and no re-run of the `fn`→`fun` / `[T]`→`<T>` migrations.
</verification>

<success_criteria>
- Syntax Pipeline Stages 1–10 are represented for `gen(...)`: reasoning trail (1–6b), EDR validation-trail cite (7), accepted record (8), hub + queue provenance (9), and grammar production (10).
- The decision is UNCHANGED: no new Human Sign-off, no edits to EDR-092 decision/sign-off/supersession fields, no migration re-runs.
- The Coupling & Overload Check records the actual grep sweep result (not a hardcoded claim) and reaches a CLEAN verdict for the `gen(` production.
- All content is English; naming and headers match existing what/syntax/*.md and how/syntax/*.md files; all cross-references resolve at correct depth.
- Three atomic `docs:` commits, each with the session attribution trailer.
</success_criteria>

<output>
Create `.planning/quick/260928-fcr-complete-syntax-pipeline-for-gen-generat/260928-fcr-SUMMARY.md` when done.
</output>
