# Codebase Concerns

**Analysis Date:** 2026-07-20

---

## Critical Issues

### Empty Placeholder Files (Blocking Architecture)

**Files:** 
- `docs/how/architecture/IR.md` (0 lines)
- `docs/how/architecture/NAME_RESOLUTION.md` (0 lines)
- `docs/how/architecture/PARSER.md` (0 lines)
- `docs/how/architecture/TYPE_SYSTEM.md` (0 lines)

**Problem:** These four core architecture components are referenced in `AGENTS.md` (Document Map, §3) as planned specifications but contain zero content. They represent essential implementation subsystems but block architectural completeness.

**Impact:** 
- Cannot draft implementation strategies without IR, parser, type system specs
- Post-Freeze milestones (8-10: Standard Library, Build System, Implementation) cannot start until these are filled
- Architectural gaps may force design rework late in the project

**Risk Level:** **HIGH** — Blocks downstream milestones

**Fix Approach:** 
1. Explicitly defer to post-Freeze (Milestone 8-10 scope) or plan completion before Milestone 7
2. If pre-Freeze, schedule concept design for these subsystems
3. Update `when/ROADMAP.md` to clarify assignment and timeline
4. Add to `TODO.md` with explicit ownership and target completion date

---

## Tech Debt

### Deprecated Template Still in Repository

**File:** `docs/how/templates/_adr.md`

**Issue:** Marked as `DEPRECATED` with clear supersession note, yet remains in the repository. Old ADR template uses outdated EDR system numbering (ADR-NNN) but references are only in this file.

**Impact:** 
- Agents/humans unfamiliar with project may find and use deprecated template
- Creates cognitive burden: must explain why two template systems exist
- Maintenance: two competing documentation patterns
- Migration burden: unclear if old ADRs (if any exist) need conversion to EDR

**Fix Approach:**
1. Migrate file to `.archived/` or remove after confirming no external references
2. Update AGENTS.md Document Map to remove _adr.md reference if exists
3. Verify no ADR-*.md files exist in decision_records (appears to be EDR-only now)

---

### Superseded Concept Still Referenced

**File:** `docs/what/concepts/EXECUTION_IMAGE.md`

**Issue:** Marked as SUPERSEDED (replaced by EXECUTION_PROGRAM.md), but still actively referenced in research notes:
- `docs/notes/imperative-crutch-lazy-sequences.md` — "See also: concepts `GENERATORS.md`, `EXECUTION_IMAGE.md`, `EXECUTION_PROGRAM.md`"
- `docs/notes/imperative-crutch-resource-management.md` — "See also: concepts `OWNERSHIP.md`, `EXECUTION_IMAGE.md`"

**Impact:**
- Readers following cross-references land on superseded concept first
- Conceptual confusion: two similar documents with unclear relationship
- Maintenance: updating one may not update the other
- Will be removed "after Concept Design Review (Milestone 2)" but no target completion date set

**Fix Approach:**
1. Remove EXECUTION_IMAGE references from notes, update to point to EXECUTION_PROGRAM only
2. Move EXECUTION_IMAGE.md to `.archived/` with version of EXECUTION_PROGRAM it was replaced by
3. Set explicit removal date or trigger (e.g., "remove on Milestone 2 completion")
4. Add to milestone checklist: "Clean up superseded concepts post-review"

---

### No Acceptance Gate for DRAFT Concepts

**Files:** 18 out of 22 concept documents marked `⚠️ DRAFT`

**Drafts:** `CORE_CONCEPTS.md`, `DATA_MODEL.md`, `EQUALITY.md`, `ALLOCATION.md`, `MUTABILITY.md`, `EXECUTION_PROGRAM.md`, `GENERATORS.md`, `ERROR_HANDLING.md`, `LLM_NATIVE_TOOLCHAIN.md`, `GENERICS.md`, `FUNCTIONS.md`, `SPAN.md`, `UNPACKING.md`, `ASYNC_AWAIT.md`, `SORTING.md`, `OWNERSHIP.md`, `LITERATE_PROGRAMMING.md`, `METAOBJECTS.md`

**Issue:** All concepts are marked as preliminary drafts "created during Milestone 0 (Foundation)". No clear process documented for:
- Transitioning from DRAFT to stable/accepted
- Who approves a concept (owner, leadership, gate)
- What "acceptance via EDR" means in practice (EDR-010 covers only architecture layer)
- How concept drafts interact with Milestone 1-2 design process

**Impact:**
- Uncertainty about document stability — readers cannot tell if content is reliable
- Downstream milestones (1-2) reference drafts, creating circular dependency
- Concept Design Review (Milestone 2) will need to formally accept/reject each — no tracking of this process yet
- EDR index shows only 1 Architecture decision (EDR-010 Layered Architecture); no concept-acceptance EDRs exist

**Risk Level:** **MEDIUM** — Risk of rework if drafts diverge significantly during Milestone 2

**Fix Approach:**
1. Define "concept acceptance" in `how/gates/DECISION_VALIDATION.md` or create new gate file `how/gates/CONCEPT_ACCEPTANCE.md`
2. Clarify: Is acceptance via Architecture EDR or separate Concept EDR category?
3. Document transition: DRAFT → Milestone 2 Review → Accepted/Deferred/Rejected (with EDR or other tracking)
4. Add "Remove DRAFT headers" to post-Milestone 2 checklist
5. Consider adding DRAFT-removal date or gate to roadmap

---

## Missing Critical Infrastructure

### Incomplete Architecture Specifications

**Deliverables promised in AGENTS.md but content missing/minimal:**

| File | Size | Status | Issue |
|------|------|--------|-------|
| `docs/how/architecture/FITNESS_FUNCTIONS.md` | 35 lines | Exists but sparse | Lists only concept template + 2 examples, no fitness functions catalogue |
| `docs/how/architecture/PARSER.md` | 0 lines | Empty | No parsing strategy, lexer design, or grammar constraints documented |
| `docs/how/architecture/TYPE_SYSTEM.md` | 0 lines | Empty | No type inference, type checking, or constraint resolution documented |
| `docs/how/architecture/NAME_RESOLUTION.md` | 0 lines | Empty | No scoping rules, symbol tables, or import semantics documented |
| `docs/how/architecture/IR.md` | 0 lines | Empty | No intermediate representation format or code generation strategy documented |
| `docs/how/DOCUMENTATION_PRINCIPLES.md` | 25 lines | Minimal | Only defines template structure, lacks enforcement or detailed principles |

**Impact:**
- Implementation strategies (Milestones 8-10) cannot be drafted without these
- Type system, scoping, and parsing rules are typically pre-Freeze (Milestone 5-7 scope)
- Fitness functions catalogue incomplete; cannot measure architectural health
- Documentation principles too brief — enforcement and consistency checking unclear

**Fix Approach:**
1. Prioritize TYPE_SYSTEM, PARSER, NAME_RESOLUTION — these are critical for Milestone 5 (Language Specification)
2. Fill FITNESS_FUNCTIONS.md with full catalogue (currently just 2 examples)
3. Expand DOCUMENTATION_PRINCIPLES.md or reference full policy from IMPLEMENTATION_POLICIES.md
4. Add to TODO.md Milestone 4-5 with explicit ownership

---

## Document Quality & Consistency Issues

### Extreme Variation in Concept Document Depth

**Analysis by concept size:**

| Size Range | Count | Examples |
|-----------|-------|----------|
| 13-17 lines (stubs) | 2 | EXECUTION_IMAGE (superseded), DATA_MODEL (incomplete draft) |
| 25-35 lines (minimal) | 4 | EQUALITY, FUNCTIONS, MUTABILITY, ALLOCATION |
| 70-90 lines (complete) | 11 | ERROR_HANDLING, GENERICS, OWNERSHIP, SPAN, METAOBJECTS |
| 150+ lines (comprehensive) | 2 | CORE_CONCEPTS (151), EXECUTION_PROGRAM (502) |

**Issue:** Concept template defined in `docs/how/templates/_concept.md` specifies uniform 7-section structure, but actual documents vary wildly (13–502 lines). No clear enforcement; some concepts deeply under-developed.

**Impact:**
- Readers expect consistent depth; some concepts seem incomplete
- Concept Design Review (Milestone 2) will need major rewrites for underdeveloped concepts
- Hampers cross-referencing — readers cannot assume all concepts are equally detailed
- Template compliance difficult to audit manually

**Fix Approach:**
1. Audit each concept against `_concept.md` template — mark compliance gaps
2. Prioritize bringing minimal concepts (25-35 line) to at least 70 lines
3. Document minimum line-count guidance or section requirements
4. Add template-compliance check to pre-commit or pre-review gate (if CI/CD exists)

---

### Missing Date Stamps and Freshness Tracking

**Issue:** Only 2 documents in entire codebase have date stamps or "last updated" metadata:
- `docs/how/decision_records/INDEX.md` — "*Last updated: 2026-07-18*"
- One other embedded reference (unclear)

Out of 91 markdown files, no systematic versioning or staleness detection.

**Impact:**
- No way to identify which documents are most/least recent
- Readers cannot assess freshness; "Is this from 2024? 2026?"
- Archival/cleanup decisions require manual file-by-file inspection
- Difficult to spot divergence between related documents (e.g., EXECUTION_IMAGE vs. EXECUTION_PROGRAM sync)

**Risk Level:** **MEDIUM** — Minor risk for pre-release docs, but critical for post-1.0 language

**Fix Approach:**
1. Add YAML frontmatter to all concept/architectural docs: `last_updated: YYYY-MM-DD`
2. Document staleness threshold (e.g., "Notify if >90 days without update")
3. Add automated check (if CI/CD exists) to flag documents >180 days old during review
4. Consider git log analysis as alternative (commit date for file)

---

## Governance & Process Gaps

### Unclear Milestone Ownership & Accountability

**TODO.md shows extensive uncompleted work:**

- **Milestone 0 (Vision):** Review items, acceptance criteria unclear (shows unchecked items)
- **Milestone 1 (Language Inventory):** 13 concept designs — no owner assigned per concept
- **Milestone 2 (Anti-pattern Analysis):** 10 research items — no owner, no estimated effort
- **Milestones 3-7:** High-level deliverables without detail or ownership

**No clear answer to:** "Who is responsible for designing PATTERN_MATCHING? When must it be complete?"

**Impact:**
- No project accountability — milestones stall waiting for volunteers
- Cannot estimate completion dates
- Cannot route issues or PRs to responsible maintainer
- Risk of duplicate work or abandoned concepts

**Fix Approach:**
1. Assign owner (name or role) to each TODO item in `TODO.md`
2. Add target completion date or version (e.g., "Week of Jul 27", "Milestone 1.5")
3. Add effort estimate (small/medium/large) for planning
4. Create separate `GOVERNANCE.md` or governance section in `AGENTS.md` documenting ownership model
5. Consider GitHub Issues or project board if this project uses GitHub

---

### EDR Numbering Gap

**Issue:** EDR-008 and EDR-009 are intentionally skipped (per `how/decision_records/INDEX.md`), with note:
> "EDR-008 and EDR-009 are intentionally skipped. TDR-007 and TDR-008 are superseded by EDR-001, not migrated."

**Problem:**
- TDR (old system) records not migrated to EDR archive/historical reference
- Gap in numbering creates question: "Why skip 8-9? Are there open slots?"
- Historical rationale (TDR → EDR) documented only in INDEX.md, not in EDR-001 itself

**Impact:**
- Potential for confusion when new decisions are added
- Lost traceability to pre-EDR design history (TDR-001 through TDR-010)
- Future maintainers may not understand intentional gaps

**Fix Approach:**
1. Create `.archived/decision_records/TDR/` with stub files or summary of TDR-001 through TDR-010 and their mapping to EDRs
2. Document in EDR-001 why TDRs were abandoned (link to conversion rationale)
3. Add comment in INDEX.md explaining: "EDR-008 and EDR-009 reserved for future process decisions" (if that's the plan)
4. Update TODO.md or roadmap to clarify if EDR-008 and EDR-009 should eventually be filled

---

## Fragile & High-Maintenance Areas

### Cross-Reference Network at Risk

**Pattern identified:** Research notes reference concept documents, which reference research notes. Example:
- `docs/notes/imperative-crutch-collections-loops.md` → "See also: concepts `GENERATORS.md`, `UNPACKING.md`, `SORTING.md`"
- Concept docs themselves reference back to notes for examples

**Risk:** If any concept is renamed, moved, or deleted, orphaned links will break silently (no validation tool apparent).

**Impact:**
- Broken links create reader friction
- Links to superseded EXECUTION_IMAGE will confuse readers
- Hard to audit which concepts are most critical (by reference count)

**Fix Approach:**
1. Add link-checking to CI/CD (if available) — e.g., `markdown-link-check`
2. Audit all "See also" sections for dead links
3. Consider using reference-style links ([1]: path) for easier maintenance
4. Document the concept reference network in a "Dependency Graph" section (future STRUCTURE.md)

---

### Concept Design Review Process Undefined

**Milestone 2 (Concept Design Review) is described in `when/ROADMAP.md` as:**
> "Critical review of concepts for philosophical consistency, LLM generability, and implementation feasibility."

**But no gate, checklist, or acceptance criteria documented for reviewing a concept.**

**Issues:**
- How do reviewers evaluate whether a concept passes review?
- What makes a concept "ready for language specification"?
- If a concept fails review, what then? (Deferred, redesigned, removed?)
- No EDR category for Concept decisions yet

**Impact:**
- Milestone 2 will stall on process clarification
- Concepts may be blocked indefinitely with no clear path forward
- No way to appeal or escalate rejected concepts

**Fix Approach:**
1. Create `docs/how/concept-design-review.md` (stub exists, needs completion) with full process
2. Add concept acceptance gate to `how/gates/DECISION_VALIDATION.md` or create separate gate file
3. Define "Accepted Concept" state and transition rules in process doc
4. Link from AGENTS.md Document Map
5. Add checklist template for reviewers

---

## Maintenance & Evolution Risks

### Large Monolithic Documents Difficult to Maintain

**Complex files at risk of drift:**

| File | Size | Risk |
|------|------|------|
| `docs/what/concepts/EXECUTION_PROGRAM.md` | 502 lines | Conceptual core; likely to evolve; needs version tracking |
| `docs/what/GLOSSARY.md` | 489 lines | Central terminology; must stay in sync with concept docs |
| `docs/how/architecture/ARCHITECTURE.md` | 385 lines | Architectural backbone; high coupling to multiple systems |
| `docs/what/DESIGN_PRINCIPLES.md` | 390 lines | Policy document; deviations need careful review |
| `docs/AGENTS.md` | 370 lines | Agent instructions; critical for newcomer onboarding |

**Issue:** No explicit versioning strategy for these documents. When a major change is needed (e.g., concept redefinition), no process to:
- Announce breaking changes
- Update dependent documents systematically
- Maintain backward compatibility or migration guide

**Impact:**
- Large document edits risk introducing inconsistencies
- Readers may rely on outdated mental model if changes aren't propagated
- Post-Freeze language evolution (Milestones 8-10) will require careful versioning

**Fix Approach:**
1. Add versioning section to top of large docs (>300 lines): "Last Revised: YYYY-MM-DD | Version: X.Y"
2. Document breaking change policy (e.g., "EXECUTION_PROGRAM concept redefined in v0.2; see migration guide [link]")
3. Create migration/change-log section for EXECUTION_PROGRAM, GLOSSARY, DESIGN_PRINCIPLES
4. Consider splitting EXECUTION_PROGRAM into multiple documents if it continues to grow

---

## Gaps in Enforcement & Validation

### Template Compliance Not Enforced

**Condition:** Concept documents have 7-section template defined in `_concept.md`:
1. **Issue (Why)**
2. **Principles**
3. **Model (What)**
4. **Default Strategy**
5. **Alternative Strategies**
6. **Open Questions**
7. **Decision History**

**Reality:** No auditable way to confirm all concepts follow this structure. No checklist, no linter rule, no review gate.

**Impact:**
- Concepts may be missing critical sections (e.g., "Decision History" explaining rejected alternatives)
- Readers cannot rely on consistent structure
- Concept Design Review will need to check manually

**Fix Approach:**
1. Add template-compliance section to concept-design-review process
2. Consider creating a simple shell script/Python linter to check for required section headers
3. Add linting to CI/CD if available
4. Document expected section order in DOCUMENTATION_PRINCIPLES.md

---

### Glossary Maintenance Burden

**File:** `docs/what/GLOSSARY.md` (489 lines)

**Issue:** 
- Central terminology reference created and currently linked from concept docs
- No process documented for adding new terms
- AGENTS.md §6 says: "When introducing a new term, add it to `what/GLOSSARY.md` and cross-reference the source document"
- But no verification mechanism ensures terms are added when new concepts are created

**Impact:**
- Glossary likely to fall out of sync with concept documents during Milestone 1-2
- Readers may miss unified terminology reference if it's incomplete
- Post-Freeze language documentation will require glossary as single source of truth

**Fix Approach:**
1. Add pre-review checklist item: "New concepts must register all terms in GLOSSARY.md"
2. Audit existing concepts for unmapped terms
3. Document glossary maintenance workflow (e.g., periodic sync with concept files)

---

## Scaling Bottlenecks (Pre-Freeze)

### Documentation Growth Trajectory

**Current state:**
- 91 markdown files
- Major growth expected in Milestones 1-2 (concepts and research)
- Post-Freeze growth slower but steady (stdlib, tooling specs)

**Concern:** No clear information architecture for organizing growth. Example:
- Where do Milestone 1 concept design decisions live? (In concept files themselves? Or separate?)
- How are Milestone 3-5 cross-cutting decisions stored? (New EDR categories needed?)
- Will concepts continue to live in `what/concepts/` or migrate somewhere during freeze?

**Impact:**
- Risk of "docs/ as monolithic dump" pattern
- Navigation difficulty as document count grows
- Unclear ownership of new files

**Fix Approach:**
1. Document in AGENTS.md §3 Document Map: "Concepts created in Milestone 1 must be added to `what/concepts/` with EDR in `how/decision_records/architecture/`"
2. Define EDR category expansion plan if new categories (e.g., Data, Tooling) needed
3. Consider versioned language specs (e.g., `specs/Orthon-0.1.md`, `specs/Orthon-0.2.md`) post-Freeze
4. Create namespace strategy for future (e.g., `stdlib/`, `tooling/`) to avoid root-level bloat

---

## Summary Table: Severity & Priority

| Issue | Severity | Impact | Owner Action |
|-------|----------|--------|--------------|
| Empty architecture files (IR, Parser, Type System, Name Resolution) | 🔴 Critical | Blocks Milestones 5-10 | Assign owners, set completion target |
| Deprecated _adr.md template still in repo | 🟡 Medium | Maintenance burden, confusion | Archive or remove post-verify |
| Superseded EXECUTION_IMAGE still referenced | 🟡 Medium | Reader confusion, sync drift | Update refs, archive concept |
| DRAFT concept acceptance gate undefined | 🟠 High | Rework risk, milestone stall | Define acceptance process + gate |
| Concept depth inconsistency (13–502 lines) | 🟠 High | Uneven quality, rework needed | Audit templates, prioritize underdeveloped |
| No date stamps/staleness tracking | 🟡 Medium | Reader uncertainty, cleanup blind spot | Add frontmatter timestamps |
| Milestone ownership undefined | 🔴 Critical | Accountability gap, stalling | Assign owners to TODO items |
| EDR numbering gap (008-009) not explained | 🟢 Low | Historical confusion, minor | Document rationale |
| Cross-reference links at risk | 🟡 Medium | Silent breakage, reader friction | Add link validation to CI/CD |
| Concept Design Review process undefined | 🔴 Critical | Milestone 2 will stall | Create review process + gate |
| Template compliance not enforced | 🟠 High | Inconsistent quality | Add linting/checklist |
| Glossary sync not automated | 🟡 Medium | Incomplete reference, drift | Add pre-review validation |

---

*Concerns audit: 2026-07-20*
