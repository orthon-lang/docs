---
phase: quick-261101-ntd
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - how/gates/methods/NYQUIST_VALIDATION_METHOD.md
  - how/gates/DECISION_VALIDATION.md
  - how/concept-design-review.md
  - how/PROCESS_INVENTORY.md
  - how/process/DECISION_PIPELINE.md
  - what/GLOSSARY.md
  - how/decision_records/process/EDR-099-nyquist-validation-method.md
  - how/decision_records/INDEX.md
autonomous: true
must_haves:
  truths:
    - Nyquist Validation is a documented Orthon process method for auditing whether significant semantic invariant violations are detectable.
    - The method gives a reproducible sequence from invariant to violation scenario, observation, verdict, and explicit coverage limit.
    - Orthon still has seven validation gates; the method audits sufficiency across gates and fitness functions rather than adding a gate.
    - The method is distinguished from GSD's requirement-to-automated-test coverage mechanism and makes no mathematical sampling guarantee.
    - The process decision and its alternatives are traceable through an indexed Process EDR.
  artifacts:
    - how/gates/methods/NYQUIST_VALIDATION_METHOD.md
    - how/decision_records/process/EDR-099-nyquist-validation-method.md
  key_links:
    - Decision Validation and Concept Design Review link to the method at its actual relative path.
    - Process Inventory and Glossary point to the canonical method and EDR.
    - EDR-099 appears in both Index tables.
---

<objective>
Add Nyquist Validation as an additional, cross-gate process method for finding blind spots in Orthon's design checks.

Purpose: A gate pass or documented example can miss a semantic failure at a concept boundary if no selected observation would reveal it.
Output: An operational method, process integration, glossary entry, and indexed Process EDR.
</objective>

<context>
@AGENTS.md
@.planning/STATE.md
@how/process/DECISION_PIPELINE.md
@how/gates/DECISION_VALIDATION.md
@how/gates/_language-design.md
@how/concept-design-review.md
@how/PROCESS_INVENTORY.md
@how/decision_records/process/EDR-003-validation-methods.md
@how/templates/_edr-process.md
@/Users/mniedre/Documents/cane-wiki/Wiki/Engineering/Process/Nyquist-Validation.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Define the method and its limits</name>
  <files>how/gates/methods/NYQUIST_VALIDATION_METHOD.md, what/GLOSSARY.md</files>
  <action>Create an English-only method document matching the structure of existing method files. State its question, applicability to design documentation without a compiler, and steps: name a semantic invariant; enumerate meaningful violations including interactions and event/order boundaries; select discriminating evidence or observation in the specification, example, gate trail, or future executable check; compare expected with observed outcome; record uncovered cases and the scope of the verdict. Provide one concrete Orthon semantic example using an existing accepted invariant and clearly distinguish an illustrative scenario from executable test evidence. Explain that the Nyquist-Shannon theorem is only an analogy: no sampling frequency or completeness guarantee is claimed. Distinguish the method from GSD's requirement-to-test mapping. Add an alphabetically placed Glossary entry with Source and See also links to canonical repo files, following glossary style.</action>
  <verify><automated>test -f how/gates/methods/NYQUIST_VALIDATION_METHOD.md &amp;&amp; rg -n 'Nyquist Validation|invariant|violation|observation|GSD|analogy' how/gates/methods/NYQUIST_VALIDATION_METHOD.md what/GLOSSARY.md</automated></verify>
  <done>The method is actionable for a documentation-only repository and the new term is defined in the Glossary.</done>
</task>

<task type="auto">
  <name>Task 2: Connect the method to existing review flow</name>
  <files>how/gates/DECISION_VALIDATION.md, how/concept-design-review.md, how/PROCESS_INVENTORY.md, how/process/DECISION_PIPELINE.md</files>
  <action>Add a distinct supplemental method entry near the seven gate methods in Decision Validation while preserving each gate's current method and the seven-gate count. Specify when to apply the audit: after problem framing and while preparing or reviewing validation evidence, especially for cross-concept, order-dependent, or strategy-dependent invariants. Add a concise actionable reference in Concept Design Review's validation step, an inventory row, and a post-filter routing note in Decision Pipeline. Keep the Pipeline's ten questions and quick-reject purpose intact. Match existing heading and table styles, and use resolving relative links.</action>
  <verify><automated>rg -n 'Nyquist Validation|seven|10 questions|10-question' how/gates/DECISION_VALIDATION.md how/concept-design-review.md how/PROCESS_INVENTORY.md how/process/DECISION_PIPELINE.md</automated></verify>
  <done>A reader can find the method from the Pipeline, validation overview, review procedure, and process inventory without mistaking it for an eighth gate.</done>
</task>

<task type="auto">
  <name>Task 3: Record the process decision</name>
  <files>how/decision_records/process/EDR-099-nyquist-validation-method.md, how/decision_records/INDEX.md</files>
  <action>Verify EDR-099 remains the next sequential unused ID immediately before writing. Create an English Process EDR using the process template and recording the problem, decision, scope, costs, alternatives (eighth gate, GSD-style test mapping only, no added method), compliance, and relationship to EDR-003 and existing validation. This is a Tier 3 process decision; do not claim human sign-off for a Tier 1-2 language design decision. Add EDR-099 to both All Records and By-Category Process tables in INDEX.md, using unique surrounding context. Keep the index's last-updated note accurate and concise.</action>
  <verify><automated>test -f how/decision_records/process/EDR-099-nyquist-validation-method.md &amp;&amp; test "$(rg -c '^\| EDR-099 \|' how/decision_records/INDEX.md)" -eq 2 &amp;&amp; git diff --check</automated></verify>
  <done>The process choice is documented and indexed twice, and all new or changed Markdown links resolve.</done>
</task>

</tasks>

<verification>
Check English-only new text, existing heading anchors, relative Markdown link targets, the seven-gate count, EDR numbering and dual-index entries, and git diff --check. Do not edit the unrelated worktree changes.
</verification>

<success_criteria>
The user can use Nyquist Validation as a separate, bounded process method and trace its rationale without changing Orthon language semantics.
</success_criteria>

<output>
Create .planning/quick/261101-ntd-add-nyquist-validation-as-an-additional-/261101-ntd-SUMMARY.md when done.
</output>
