---
date: "2026-10-17 08:22"
promoted: false
---

# Spec-Level Complexity Management: Gap Analysis

> **Exploratory note.** Maps problems that appear as a codebase grows (cascading changes, boundary quality, linguistic conflicts, late-integration collapse, traceability, ownership) onto the design of the Orthon specification itself, not onto programs written in Orthon. Evidence was collected on 2026-10-17 at commit `406dae7`.

## Premise

The specification is itself a growing system: 92 EDRs, 58 accepted concepts, more than 130 research documents, 2571 relative cross-links. The cost of changing it follows the same law as software: cost ≈ cost of cascading changes ≈ coupling between decisions. The Core Value ("every concept accepted, all cross-references valid, final review confirming coherence") is an end-of-project integration step, which is exactly where integration collapse usually happens (the "80-90% readiness" effect).

## Problem Mapping

Relevance: **High** = applies to the spec now; **Medium** = applies partially or later; **Low** = defer.

| # | Problem (program level) | Spec-level analog | Relevance | Evidence in repo | Existing mechanism | Gap |
|---|---|---|---|---|---|---|
| 1 | Cascading changes (coupling) | A change to one EDR or concept propagates into other concepts, GLOSSARY, INDEX, syntax and strategy docs | High | 11 of 92 EDRs superseded; EDR-095 propagation into 3+ documents done by hand; a rough link check found 416 of 2571 relative links broken (~16%, some may be false positives in code examples), mostly after files moved into `research/{tier}/` | Supersede chains in EDRs; FITNESS_FUNCTIONS "Layered Isolation" (manual impact analysis per EDR) and "Concept Stability" (manual change count) | No machine-readable dependency graph between EDRs, concepts and glossary terms; no impact analysis ("what must change if EDR-X changes"); no fan-in / fan-out / instability metric per concept |
| 2 | Complexity considered in isolation (boundary quality) | Concept boundaries and subsystem clustering of the spec | High | graphify reports 723 communities, most of them unnamed; research tiers (essential / important / deferrable) express priority, not subsystem | Golden Circle layers (why / how / what / when); Semantic Dependency Architecture Levels 0-3; research tier table | No named spec subsystems (type system, memory, concurrency, modules, execution) with owned concepts and limits on cross-subsystem dependencies |
| 3 | Linguistic conflicts (Ubiquitous Language drift) | Overloaded or drifting terms in the spec vocabulary | High | "Tier" means EDR decision weight (Tier 1-4), invariant class (Tier 1-3) and research priority tier; there is no GLOSSARY entry for it. "Level 3" appears as "Standard Library", "Toolchain" and "Stdlib". DELEGATION vs DELEGATE is disambiguated by a manual note | GLOSSARY.md as single source of truth; AGENTS.md §6 | No deterministic glossary lint: undefined terms in use, one term with several meanings, one meaning under several terms |
| 4 | Collapse at 80-90% readiness (cross-defects at integration) | The final coherence review finds cross-concept contradictions late | High | Concepts are reviewed one at a time; the global coherence check is deferred to the final review | Per-concept gates; SEMANTIC_MODEL cross-dimension table (15 pairwise interactions); FITNESS_FUNCTIONS "Orthogonality" | The "composability matrix" that the Orthogonality fitness function measures does not exist as an artifact (only mentioned in FITNESS_FUNCTIONS, notes and planning); pairwise interaction checks exist only inside the Semantic Model; no continuous spec-wide integration |
| 5 | Requirements traceability | GOAL → principle → gate → EDR → concept → glossary term → (later) conformance test | High | The "Traceability" fitness function checks only concept → EDR | EDRs link goals and principles in prose; Decision History sections; DESIGN_PRINCIPLES § Transparency | No generated traceability matrix; no orphan report (goal without EDR, EDR without goal, term without source, concept without gate trail) |
| 6 | One owner per entity | One normative home per rule or concept | High | 53 of 58 accepted concepts have a second copy in `how/concepts/research/`; several documents still link to the research copy (e.g. CONSTRAINED_TYPES links the research CONTRACTS.md) | `what/concepts/README.md`: research copies kept "as traceable provenance" | No rule separating the normative copy from the provenance copy; no check that links point to the normative home; risk of divergent statements |
| 7 | Data-schema-first instead of event-first design | Designing the spec from a list of concepts instead of from programmer scenarios | Medium | WORKING_BACKWARDS.md and the Working Backwards gate method exist | Working Backwards method in gates | Scenarios are not used as spec-wide acceptance tests: no end-to-end example programs that cross several concepts |
| 8 | Formal model ↔ code conformance | Spec ↔ future implementation; code examples ↔ current syntax decisions | Medium now, High at M10 | Code examples are "illustrative only" and use abstract syntax until Phase 5 | "Deterministic Behavior" fitness function (needs an implementation) | No conformance-suite skeleton; examples are not checked against accepted syntax decisions |
| 9 | Formal verification | Mechanised model of the core (primitive minimality, type-system soundness) | Low for v0.1 | Minimality re-proof done by reasoning (DECISION_LOG, B2) | Gates with the Logical Analysis method | No mechanised model (Alloy / Lean); defer |
| 10 | Illegal states unrepresentable | Illegal document states: Accepted without EDR, missing template sections, invalid Status values | Medium | 0 accepted concepts still carry a DRAFT banner (good); Human Sign-off coverage is checked by hand | Templates; status banners; "Human Sign-off Coverage" fitness function | No schema lint for document headers and required template sections |

## Proposed Tasks

| ID | Priority | Task | Closes | Done when |
|---|---|---|---|---|
| T1 | P1 | Turn "Cross-Reference Validity" into an executable check, triage the 416 findings, fix real breaks | #1, Core Value | Checker runs on demand; zero real broken links; false-positive rules documented |
| T2 | P1 | Decide the policy for research copies of accepted concepts (stub with redirect, archive, or provenance-only marker) and repoint links to the normative home | #6 | Every concept has exactly one normative home; no links to provenance copies except from Decision History |
| T3 | P1 | Glossary lint and resolution of overloaded terms: "Tier" (decision weight vs invariant class vs research priority), "Level 3" naming | #3 | GLOSSARY entries exist; each meaning has a distinct term; lint reports zero conflicts |
| T4 | P2 | Machine-readable dependency metadata per EDR / concept (depends_on, supersedes, defines_terms, layer, subsystem) plus impact analysis and coupling metrics (fan-in, fan-out, instability) as a fitness function | #1, #2 | "What changes if EDR-X changes" is answered by a query; coupling metrics reported per concept |
| T5 | P2 | Traceability matrix generator (goal, principle, gate, EDR, concept, terms, status) with an orphan report; upgrade the "Traceability" fitness function | #5 | Matrix is generated, not hand-written; orphan report is empty or every orphan is justified |
| T6 | P2 | Create the composability matrix (concept × concept) that the Orthogonality fitness function refers to; review it on every concept acceptance instead of only at the final review | #4 | Matrix exists; every new concept fills its row and column; forbidden pairs trigger review |
| T7 | P3 | Define spec subsystems with owned concepts and allowed cross-subsystem edges; use graphify communities as input | #2 | Every concept belongs to one subsystem; cross-subsystem edges are counted and bounded |
| T8 | P3 | Document schema lint (Status enum, required sections per template, EDR fields) | #10 | Lint passes on all EDRs and accepted concepts |
| T9 | P4 | End-to-end scenario programs that cross several concepts, used as spec acceptance tests and as the seed of a conformance suite | #7, #8 | Scenario set exists and is re-checked when an affected concept changes |
| T10 | Defer | Mechanised formalisation of the primitive set (Alloy or Lean) | #9 | Revisit after v0.1 |

## Phase Assignment (decided 2026-10-17)

| Task | Placement |
|---|---|
| T1, T2, T3, T4, T5, T7, T8 | Phase 04.3: Specification Integrity Infrastructure (inserted after 04.2, before Phase 5) |
| T6 | Input to Phase 6 (owns `what/CROSS_CUTTING.md`; the composability matrix duplicates its Interaction Matrix) |
| T9 | After Phase 5 (needs concrete syntax); recorded as a Phase 6 input |
| T10 | Deferred past v0.1 |

## Where Design Complexity Can Actually Be Controlled

1. **Dependency structure between decisions.** This is the only lever on cascade cost. Superseding is cheap only when the fan-in of the superseded record is known and small.
2. **Vocabulary.** The glossary is the contract between documents. An overloaded term is a hidden coupling: changing one meaning silently changes every document that uses the other one.
3. **One normative home per rule.** Duplicated statements are two owners of one entity; divergence is a matter of time.
4. **Executable fitness functions.** Most fitness functions are checklists performed by an agent. The measurable ones (links, glossary, traceability, coupling) become cheap and continuous once scripted.
5. **WIP limit on open hypotheses.** "Concepts In Flight" already measures it. Fewer concepts open at once means fewer pairwise interactions left unchecked until the final review.

## Open Questions

1. Validation scripts conflict with the stated constraint "no build tools, no external tooling dependencies". Where should they live (GSD tooling, a `tools/` directory, the future implementation repository), and does introducing them require a Process EDR?
2. Should T3 rename "Tier" for EDR decision weight (e.g. "Decision Weight 1-4") or rename invariant tiers instead? DESIGN_PRINCIPLES.md and DECISION_PROCESS.md use "Tier 1 EDR"; the invariant taxonomy lives in DRAFT research documents, so renaming the invariant side is the cheaper cascade.
3. Can graphify (already enabled for this repository) serve as the dependency graph for T4 and T7, or does it need explicit metadata to avoid inferred edges?
