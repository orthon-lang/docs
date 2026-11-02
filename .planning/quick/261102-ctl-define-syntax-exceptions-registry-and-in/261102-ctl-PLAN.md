---
phase: quick-261102-ctl
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - how/SYNTAX_EXCEPTIONS.md
  - how/decision_records/process/EDR-100-syntax-exceptions-registry.md
  - how/decision_records/INDEX.md
  - how/SYNTAX_PIPELINE.md
  - how/syntax/README.md
  - how/PROCESS_INVENTORY.md
autonomous: true
requirements: [EDR-100, EDR-087]

must_haves:
  truths:
    - "A syntax reviewer can distinguish an accepted exception, an open conflict, and a resolved or rejected collision without treating the registry as the canonical language specification."
    - "A new syntax decision cannot silently retain a context-dependent role: Stage 3 requires a registry check and routes an unregistered collision to redesign or an explicit exception decision."
    - "An accepted syntax decision that depends on an exception leaves reciprocal provenance in its Architecture EDR, its accepted syntax record, and the registry."
    - "The project process catalog and both EDR index tables make the registry mechanism discoverable."
  artifacts:
    - how/SYNTAX_EXCEPTIONS.md
    - how/decision_records/process/EDR-100-syntax-exceptions-registry.md
    - how/SYNTAX_PIPELINE.md
    - how/syntax/README.md
    - how/PROCESS_INVENTORY.md
    - how/decision_records/INDEX.md
  key_links:
    - "EDR-100 defines the registry process; SYNTAX_EXCEPTIONS.md carries the controlled entries; SYNTAX_PIPELINE.md makes the registry check mandatory at Stage 3."
    - "Every registry entry cites canonical syntax or concept sources and the deciding EDR; it never replaces them."
---

<objective>
Define the Syntax Exceptions Registry and make it a required control in the Syntax Pipeline. Record the project-process decision as EDR-100 and seed the registry from documented current exceptions, unresolved conflicts, and historical negative decisions.

Purpose: preserve the Manifesto's one-syntactic-element/one-responsibility rule as a default that cannot be bypassed invisibly, while retaining traceable, narrowly-scoped exceptions.

Output: the new How-layer registry, Process EDR-100, pipeline and inventory integration, and EDR index updates.
</objective>

<execution_context>
@/Users/mniedre/.codex/gsd-core/workflows/execute-plan.md
@/Users/mniedre/.codex/gsd-core/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@AGENTS.md
@why/MANIFESTO.md
@how/DESIGN_PRINCIPLES.md
@how/SYNTAX_PIPELINE.md
@how/syntax/README.md
@how/PROCESS_INVENTORY.md
@how/decision_records/process/EDR-087-syntax-acceptance-process.md
@how/decision_records/process/EDR-099-nyquist-validation-method.md
@how/decision_records/INDEX.md
@how/templates/_edr-process.md
@what/SYNTAX.md
@what/concepts/UNION_INTERSECTION_TYPES.md
@what/concepts/UNPACKING.md
@how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create the registry and record the governing process decision</name>
  <files>how/SYNTAX_EXCEPTIONS.md, how/decision_records/process/EDR-100-syntax-exceptions-registry.md, how/decision_records/INDEX.md</files>
  <action>
Create `how/SYNTAX_EXCEPTIONS.md` as the authoritative process-control registry, not as a second syntax specification. Start with a clear scope statement: canonical meaning remains in the linked `what/syntax/`, `what/concepts/`, and deciding EDR documents; the registry records only the justification, boundary, status, provenance, and review trigger for deviations from Syntax Principle 2.

Define stable entry statuses: `ACCEPTED EXCEPTION` for a deliberately retained context-sensitive or mirrored role; `OPEN CONFLICT` for unresolved collision risk that blocks acceptance of the proposed additional role; and `RESOLVED` or `REJECTED` for anti-memory. Define a required row schema with the syntactic element, roles or contention, classification/status, permitted scope or decision rule, rationale, canonical sources, deciding EDR, and review/sunset condition. State the governing invariant exactly: an unregistered exception is a design defect, not an implicit allowance.

Seed the registry only with evidence-supported entries from existing records. Include the accepted mirror-symmetry treatment of prefix `*` (collect in receiving positions, spread in sending positions, with multiplication still open); the distinct ADT-variant and union-type uses of `|`; the generic-bound and explicit-label uses of `as`; the existing `!` contention across error union, forced unwrap, and proposed negative bounds; bracket literal/indexing contention as a syntax-stage concern; and the unresolved `->`, `:`, `{}`, and `<>` collision/disambiguation cases where the current sources establish them. Record withdrawn `..` rest syntax and `?` ignore syntax as rejected historical collisions. Do not invent semantics, mark speculative notation as accepted, or recast same-responsibility delimiters as violations without source evidence.

Create `how/decision_records/process/EDR-100-syntax-exceptions-registry.md` from the process template with `Accepted` status and date `2026-11-02`. The decision must establish the registry, make it the Stage 3 input and output, require an explicit registry update whenever a decision accepts, changes, resolves, rejects, or deprecates an exception, and require accepted syntax records plus their Architecture EDRs to link to applicable entries. Cover the failure mode (silent dilution of the syntax principle), proportional overhead, lifecycle/review criteria, compliance checks, alternatives, and the four applicable Process EDR gate verdicts. Make clear that this is Tier 3 and does not alter language semantics or create another validation gate.

Add EDR-100 to both the All Records and Process tables in `how/decision_records/INDEX.md`; update the status count, the last-updated narrative, and any immediately affected index text while preserving historical entries. Use the exact next sequential ID and valid relative links. All written text, headings, table cells, and code comments must be English.

Commit this coherent registry-and-EDR unit with a conventional documentation commit message.
  </action>
  <verify>
    <automated>test -f how/SYNTAX_EXCEPTIONS.md && test -f how/decision_records/process/EDR-100-syntax-exceptions-registry.md && rg -n 'unregistered exception.*design defect|ACCEPTED EXCEPTION|OPEN CONFLICT|RESOLVED|REJECTED' how/SYNTAX_EXCEPTIONS.md && rg -n 'Stage 3|Tier 3|does not.*gate|does not.*semantics' how/decision_records/process/EDR-100-syntax-exceptions-registry.md</automated>
    <automated>rg -n 'EDR-100.*Syntax Exceptions Registry|syntax-exceptions-registry' how/decision_records/INDEX.md | wc -l | awk '$1 >= 2 { exit 0 } { exit 1 }'</automated>
  </verify>
  <done>The registry exists with defined statuses, entry schema, evidence-backed seed entries, and an explicit unregistered-exception defect rule; EDR-100 makes the mechanism a bounded Tier 3 process decision; both EDR index tables and their summary metadata include EDR-100.</done>
</task>

<task type="auto">
  <name>Task 2: Make registry consultation a mandatory Syntax Pipeline control</name>
  <files>how/SYNTAX_PIPELINE.md, how/syntax/README.md, how/PROCESS_INVENTORY.md</files>
  <action>
Amend `how/SYNTAX_PIPELINE.md` without creating a parallel gate. In the overview and Stage 3, rename or clearly expand the existing Coupling & Overload Check to include a Syntax Exceptions Registry Check. Require every syntax proposal to search the registry as well as accepted concepts, then select exactly one outcome: no collision; an existing accepted exception applies within its documented boundary; an open conflict requires resolution; or a new exception must be justified, human-approved through the existing acceptance path, and entered before the syntax is accepted. State that redesign is preferred; an exception may not be assumed from context. Require Stage 7 Architecture EDR, Stage 8 accepted syntax record, and the registry to carry reciprocal links when an exception applies. Extend Phase 5 exit criteria so open registry conflicts are resolved, rejected, or explicitly deferred, and every accepted exception has complete provenance.

Update `how/syntax/README.md` only where it describes the Coupling & Overload Check or syntax acceptance conventions. Point readers to `how/SYNTAX_EXCEPTIONS.md`, state that collision sweeps include the registry, and preserve the existing Decision Queue as the inventory of hypotheses rather than duplicating it. Make the distinction explicit: the queue tracks decisions; the registry tracks exceptions and collision history.

Update `how/PROCESS_INVENTORY.md` with one concise registry row, a dependency-graph relation from the Syntax Pipeline/EDR-100 to the registry, and a See Also link. Describe the prevented failure concretely: unrecorded context-dependent syntax becomes invisible design debt. Do not change the language principle itself, move canonical syntax into the How layer, or add a new validation gate.

Verify every newly added or changed Markdown link resolves relative to its source document, including all links from the new registry and EDR. Commit the pipeline-and-inventory integration as a separate conventional documentation commit.
  </action>
  <verify>
    <automated>rg -n 'Syntax Exceptions Registry Check|SYNTAX_EXCEPTIONS.md|unregistered' how/SYNTAX_PIPELINE.md how/syntax/README.md how/PROCESS_INVENTORY.md</automated>
    <automated>rg -n 'EDR-100|Syntax Exceptions Registry' how/PROCESS_INVENTORY.md how/SYNTAX_PIPELINE.md how/syntax/README.md</automated>
    <automated>node -e "const fs=require('fs'),path=require('path');const files=['how/SYNTAX_EXCEPTIONS.md','how/decision_records/process/EDR-100-syntax-exceptions-registry.md','how/SYNTAX_PIPELINE.md','how/syntax/README.md','how/PROCESS_INVENTORY.md','how/decision_records/INDEX.md'];let bad=[];for(const f of files){const s=fs.readFileSync(f,'utf8');for(const m of s.matchAll(/(?<!\!)\[[^\]]*\]\(([^)]+)\)/g)){let p=m[1].split('#')[0];if(!p||/^[a-z]+:\/\//i.test(p))continue;if(!fs.existsSync(path.resolve(path.dirname(f),p)))bad.push(f+' -> '+p)}}if(bad.length){console.error(bad.join('\n'));process.exit(1)}"</automated>
  </verify>
  <done>Stage 3 makes registry consultation mandatory and defines its four decision outcomes; the EDR/accepted-record/registry provenance loop is explicit; the syntax inbox and the registry have non-overlapping responsibilities; the process inventory and all edited links are current.</done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| Process record to language specification | The registry summarizes exceptions but must not become an alternate source of syntax semantics. |
| Existing syntax decision to new process control | Historical records must retain their meaning while gaining traceable exception provenance. |

## STRIDE Threat Register

| Threat ID | Category | Component | Severity | Disposition | Mitigation Plan |
|-----------|----------|-----------|----------|-------------|-----------------|
| T-ctl-01 | Tampering | Syntax principle provenance | Medium | mitigate | Require canonical-source and deciding-EDR links in each registry entry and verify all added Markdown links. |
| T-ctl-02 | Repudiation | Exception acceptance | Medium | mitigate | Require registry status, rationale, scope, review trigger, Architecture EDR, and accepted-record cross-links before acceptance. |
| T-ctl-03 | Denial of service | Syntax review workflow | Low | accept | Keep the registry as a bounded Stage 3 lookup and reuse existing gates rather than adding a new gate. |
| T-ctl-SC | Tampering | Dependency installs | Low | accept | Documentation-only task; no package installation is involved. |
</threat_model>

<verification>
1. Confirm the six planned documents exist or are updated and only contain English additions.
2. Confirm `EDR-100` occurs in both EDR index tables and points to its Process record.
3. Confirm Stage 3 includes registry lookup, the four outcomes, and update obligations for an exception decision.
4. Resolve all Markdown links introduced in the registry, EDR, pipeline, inbox README, process inventory, and index.
5. Confirm `git log --oneline` contains the two focused documentation commits and no unrelated files are staged.
</verification>

<success_criteria>
- The project has one discoverable registry for syntax exceptions, open conflicts, and anti-memory, with canonical sources retained elsewhere.
- Future syntax acceptance cannot silently create a context-dependent exception: it must redesign or register and document the decision.
- EDR-100 and the existing pipeline, inbox, inventory, and EDR journal describe the same lifecycle without introducing a new gate.
- All new or changed content is English and all added cross-references resolve.
</success_criteria>

<output>
Create `.planning/quick/261102-ctl-define-syntax-exceptions-registry-and-in/261102-ctl-SUMMARY.md` when done.
</output>
