# Requirements: Orthon Language Specification

**Defined:** 2026-07-20
**Core Value:** A complete, self-consistent Orthon v0.1 specification — every concept accepted (no DRAFT), core architecture specs filled in, all cross-references valid, and a final review confirming the result coheres as a real language specification.

## v1 Requirements

Requirements for the v0.1 specification. Each maps to roadmap phases (Milestones 0-7).

### Foundations & Process (resolves CONCERNS.md governance/process gaps)

- [ ] **PROC-01**: Concept acceptance gate defined — documents the DRAFT → Accepted transition (owner, criteria, EDR linkage)
- [ ] **PROC-02**: Concept Design Review process fully documented — checklist, review criteria, and accept/defer/reject outcomes (`docs/how/concept-design-review.md`)
- [ ] **PROC-03**: Milestone/TODO tracking model defined for solo authorship — every TODO.md item converted into a tracked GSD requirement with explicit status
- [ ] **PROC-04**: Template-compliance process defined for the 7-section concept template (`_concept.md`) — audit checklist or equivalent

### Concerns Remediation (CONCERNS.md blockers & tech debt)

- [ ] **DEBT-01**: `docs/how/architecture/IR.md` filled with intermediate representation design
- [ ] **DEBT-02**: `docs/how/architecture/PARSER.md` filled with parsing strategy / grammar design
- [ ] **DEBT-03**: `docs/how/architecture/TYPE_SYSTEM.md` filled with type inference/checking design
- [ ] **DEBT-04**: `docs/how/architecture/NAME_RESOLUTION.md` filled with scoping/symbol resolution design
- [ ] **DEBT-05**: Deprecated `docs/how/templates/_adr.md` archived or removed; references cleaned up
- [ ] **DEBT-06**: Superseded `EXECUTION_IMAGE.md` references updated to point to `EXECUTION_PROGRAM.md`; concept archived
- [ ] **DEBT-07**: Date-stamp / freshness metadata added to concept and architecture documents
- [ ] **DEBT-08**: EDR numbering gap (008-009) and TDR→EDR migration rationale documented
- [ ] **DEBT-09**: `docs/how/architecture/FITNESS_FUNCTIONS.md` filled with a full fitness-functions catalogue
- [ ] **DEBT-10**: `docs/how/DOCUMENTATION_PRINCIPLES.md` expanded beyond the current template stub

### Milestone 0 — Vision

- [ ] **VISION-01**: Manifesto, Vision, and Zen reviewed against Working Backwards rationale
- [ ] **VISION-02**: Criteria-based algorithm selection idea validated (`docs/notes/criteria-based-algorithm-selection.md`)

### Milestone 1 — Language Inventory (13 concepts)

- [ ] **CONCEPT-01**: Pattern Matching concept designed
- [ ] **CONCEPT-02**: Error Handling concept designed
- [ ] **CONCEPT-03**: Ownership concept designed
- [ ] **CONCEPT-04**: Generics / Traits concept designed
- [ ] **CONCEPT-05**: Async/Await concept designed
- [ ] **CONCEPT-06**: Object Initialization concept designed
- [ ] **CONCEPT-07**: Concurrency / Actors concept designed
- [ ] **CONCEPT-08**: Literate Programming concept designed
- [ ] **CONCEPT-09**: Sorting Stability concept designed
- [ ] **CONCEPT-10**: Unpacking / Destructuring concept designed
- [ ] **CONCEPT-11**: Generators / Yield concept designed
- [ ] **CONCEPT-12**: Metaobjects concept designed
- [ ] **CONCEPT-13**: Span / Memory View concept designed

### Milestone 2 — Anti-pattern & Declarative Design Analysis

- [ ] **ANTIPAT-01**: Collections & Loops anti-pattern research completed
- [ ] **ANTIPAT-02**: Null handling anti-pattern research completed
- [ ] **ANTIPAT-03**: Resource management anti-pattern research completed
- [ ] **ANTIPAT-04**: Collection initialization anti-pattern research completed
- [ ] **ANTIPAT-05**: Type checking & casting anti-pattern research completed
- [ ] **ANTIPAT-06**: Date/time API anti-pattern research completed
- [ ] **ANTIPAT-07**: Lazy sequences & generators anti-pattern research completed
- [ ] **ANTIPAT-08**: Multi-key sorting anti-pattern research completed
- [ ] **ANTIPAT-09**: Metaprogramming & reflection anti-pattern research completed
- [ ] **ANTIPAT-10**: Serialization anti-pattern research completed
- [ ] **ANTIPAT-11**: All 22 concept documents pass Concept Design Review — DRAFT headers removed, each transitions to Accepted/Deferred/Rejected with documented rationale (EDR or equivalent)

### Milestone 3 — Cross-cutting Review

- [ ] **XCUT-01**: Interaction Matrix format investigated and finalized

### Milestone 6 — Naming

- [ ] **NAME-01**: "Orthon" verified as the final name, or replacement decided

### Milestone 7 — Freeze

- [ ] **FREEZE-01**: All validation issues resolved or explicitly deferred with documented rationale
- [ ] **FREEZE-02**: Specification verified self-consistent and complete
- [ ] **FREEZE-03**: All cross-references ("See also" links etc.) validated — no dangling links
- [ ] **FREEZE-04**: Dedicated final review confirms Orthon v0.1 coheres as a complete language specification
- [ ] **FREEZE-05**: Version tagged (e.g., "Orthon 0.1 Specification")
- [ ] **FREEZE-06**: Whitepaper strategy revisited; whitepaper(s) commissioned/drafted
- [ ] **FREEZE-07**: Launch/announcement strategy defined (audience, channels, timing)

## v2 Requirements

Deferred to post-Freeze (Milestones 8-9 and LLM tooling). Tracked but not in current roadmap.

### Milestone 8 — Standard Library & FFI

- **STDLIB-01**: Standard library architecture designed (collections, I/O, formatting)
- **STDLIB-02**: FFI & interoperability designed (C ABI, embedding API)
- **STDLIB-03**: Standard library whitepaper written, if stdlib design introduces novel concepts

### Milestone 9 — Build System & Tooling

- **BUILD-01**: Build system & package manager designed
- **BUILD-02**: Developer tooling designed (formatter, linter, LSP)
- **BUILD-03**: Build-system whitepaper benefit evaluated

### Orthon for LLM (beyond the already-shipped LLM Generability Gate, EDR-011)

- **LLMTOOL-01**: LLM Toolchain implemented (Schema Provider, Code Completer, Code Generator, Static Analyser)
- **LLMTOOL-02**: ML-friendly grammar research (tokenizer alignment, ambiguity metrics)
- **LLMTOOL-03**: Common LLM error-pattern research across languages
- **LLMTOOL-04**: Literate Programming as LLM-native interface designed
- **LLMTOOL-05**: Opinionated Code Generation mode decided (lint-level constraints for LLM output)
- **LLMTOOL-06**: Structured error format for LLM consumption decided (JSON schema)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Milestone 8 — Standard Library & FFI implementation/design | Deferred until v0.1 spec is locked at Freeze |
| Milestone 9 — Build System & Tooling design | Deferred until v0.1 spec is locked at Freeze |
| Milestone 10 — Compiler/runtime implementation | Happens in a separate repository; this project is documentation-only |
| Full LLM Toolchain implementation | Design/decisions only pre-Freeze; implementation deferred to post-Freeze (Milestone 8 area) |

## Traceability

To be populated during roadmap creation (`/gsd-new-project` roadmap step).

**Coverage:**
- v1 requirements: 49 total
- Mapped to phases: 0 (pending roadmap)
- Unmapped: 49 ⚠️ (expected until roadmap phase runs)

---
*Requirements defined: 2026-07-20*
*Last updated: 2026-07-20 after initial definition*
