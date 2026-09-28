# AGENTS.md — Agent Guide for Orthon

> Instructions for AI agents contributing to the Orthon language design project.

---

## 1. Project Identity

Orthon is a **programming language design project**. This repository contains *only* design documentation — no implementation code, no build system, no runtime. The entire project exists in the `docs/` directory.

The language is designed using the same SOLID engineering principles it encourages in its users: a small, orthogonal core with layered abstraction, explicit semantics, and implementation-independent evolution.

> **⚠ Language Rule:** All file content (documents, code snippets, comments, commit messages) MUST be in English. Chat responses to the user may be in the user's language. See §10.9 for full enforcement.

---

## 2. Documentation Framework

All documentation follows the **Why → How → What** (Golden Circle) framework:

| Layer | Question | Documents |
|-------|----------|-----------|
| **Why** | Why does Orthon exist? What do we believe? What are we trying to achieve? | `why/VISION.md`, `why/MANIFESTO.md`, `why/ZEN.md`, `why/GOALS.md` |
| **How** | How is Orthon designed and structured? | `how/DESIGN_PRINCIPLES.md`, `how/architecture/ARCHITECTURE.md`, `how/strategies/IMPLEMENTATION_STRATEGIES.md`, `how/IMPLEMENTATION_POLICIES.md` |
| **What** | What is Orthon concretely? | `what/CORE_CONCEPTS.md` (accepted concept registry — currently empty), `how/concepts/research/` (in-progress research) |
| **How** | How are concepts designed? | `how/concepts/research/` (research inbox), `how/concepts/README.md` (pipeline) |

An agent must **always** anchor new content to the correct layer. A "Why" argument must not be smuggled into a "What" document. If a new piece of documentation spans layers, split it or add cross-references.

---

## 3. Document Map

| File | Layer | Purpose |
|------|-------|---------|
| `why/VISION.md` | Why | Core philosophy, Principle of Least Astonishment, orthogonality, Execution Program model |
| `why/POSITIONING.md` | Why | Strategic positioning — the problem being solved, why it is central, the chosen approach, and explicit trade-offs |
| `why/DESIGN_INFLUENCES.md` | Why | External language influences — Python, Java, and what Orthon learns from them |
| `why/GOALS.md` | Why | Concrete aims derived from the vision — six goals with criteria and non-goals |
| `why/MANIFESTO.md` | Why | Explicit principles — consistency over legacy, minimal core, composition over exceptions |
| `why/ZEN.md` | Why | Aphorisms capturing the language's spirit |
| `how/DESIGN_PRINCIPLES.md` | How | Orthogonality, simplicity, explicitness, consistency, execution model principles |
| `how/architecture/ARCHITECTURE.md` | How | Layered architecture — Core Language → Syntax → Standard Library → Implementation Strategy (with Policies) |
| `how/architecture/FITNESS_FUNCTIONS.md` | How | Architectural fitness functions — catalogue of measurable checks guarding against design decay |
| `how/strategies/IMPLEMENTATION_STRATEGIES.md` | How | Strategy = named set of Policies; declarative profiles and aspect mapping |
| `how/IMPLEMENTATION_POLICIES.md` | How | Policy-level decisions for implementation work |
| `how/architecture/PARSER.md` | How | Source code parsing and lexing |
| `how/architecture/TYPE_SYSTEM.md` | How | Type checking and type inference |
| `how/architecture/NAME_RESOLUTION.md` | How | Symbol resolution and scope management |
| `how/architecture/IR.md` | How | Intermediate representation and code generation |
| `how/strategies/DEFAULT_STRATEGY.md` | How | Default implementation strategy |
| `how/strategies/EMBEDDED_STRATEGY.md` | How | Strategy for embedded / resource-constrained targets |
| `how/strategies/HIGH_PERFORMANCE_STRATEGY.md` | How | Strategy for performance-optimized targets |
| `what/CORE_CONCEPTS.md` | What | Registry of accepted Orthon concepts — currently empty (see `how/concepts/research/` for in-progress research) |
| `what/THESES.md` | What | Distilled, verifiable understanding anchors about language semantics — status-tracked, routed to canonical destinations |
| `what/concepts/README.md` | What | Describes the acceptance pipeline for concept drafts |
| `how/concepts/README.md` | How | Concept design pipeline: research → design → spec |
| `how/concepts/research/DATA_MODEL.md` | How | Concept research: formal data model analysis |
| `how/concepts/research/EQUALITY.md` | How | Concept research: equality semantics |
| `how/concepts/research/ALLOCATION.md` | How | Concept research: memory allocation model |
| `how/concepts/research/OWNERSHIP.md` | How | Concept research: ownership model |
| `how/concepts/research/MUTABILITY.md` | How | Concept research: mutability model |
| `how/concepts/research/FUNCTIONS.md` | How | Concept research: function model |
| `how/concepts/research/EXECUTION_PROGRAM.md` | How | Concept research: execution program model |
| `how/concepts/research/` (additional files) | How | Concept research inbox (20+ draft analyses) |
| `AGENTS.md` | Meta | This file — instructions for AI agents |
| `what/GLOSSARY.md` | Meta | Unified terminology reference with cross-document links |
| `how/templates/_edr.md` | Meta | EDR base template (fill-in form) |
| `how/templates/_edr-architecture.md` | Meta | EDR Architecture category template |
| `how/templates/_edr-process.md` | Meta | EDR Process category template |
| `how/templates/_design-review.md` | Meta | Design review template (fill-in form) |
| `how/decision_records/{category}/EDR-*.md` | Meta | Engineering Decision Records — logged per-decision |
| `how/decision_records/INDEX.md` | Meta | Unified EDR journal — master index of all decisions |
| `how/gates/_language-design.md` | Meta | Quality gate checklist for language design decisions |
| `how/gates/DECISION_VALIDATION.md` | How | Six independent validation gates for language design decisions |
| `how/process/DECISION_PROCESS.md` | How | One-page decision authority — who decides what, using which criteria |
| `how/process/DECISION_PIPELINE.md` | How | 10-question pipeline — pre-filter before detailed concept design |
| `how/EXPLORING_THE_DOCS.md` | How | Using `graphify` to build a queryable knowledge graph of this repository for orientation and research |
| `what/SEMANTIC_MODEL.md` | What | Unified semantic model — identity, ownership, mutation, evaluation, visibility, lifetime |
| `what/PRIMITIVE_BLOCKS.md` | What | Minimal orthogonal primitive blocks — all features decompose to these |
| `what/LIBRARY_BOUNDARY.md` | What | Language vs. standard library vs. external classification |
| `what/SYNTAX.md` | What | Syntax hub — principles + pointers to `what/syntax/` (EDR-087) |
| `what/syntax/README.md` | What | Accepted syntax records — index, one file per construct (EDR-087) |
| `how/SYNTAX_PIPELINE.md` | How | Syntax acceptance pipeline — hypothesis to accepted syntax (EDR-087) |
| `how/syntax/README.md` | How | Syntax hypothesis inbox + decision queue — open Phase 5 decisions (EDR-087) |
| `what/CROSS_CUTTING.md` | What | Interaction matrix — pair-wise concept interaction analysis |
| `what/CONFLICT_REGISTRY.md` | What | Concept-boundary conflict tracking and resolution |
| `what/EXECUTION_MODEL.md` | What | Execution semantics — what the language guarantees about execution |
| `what/OPTIMIZATION_MODEL.md` | What | Semantics vs. optimisation boundary |
| `how/EVOLUTION_MODEL.md` | How | Versioning, deprecation, experimental features, feature gates |
| `how/tooling/README.md` | How | Tooling Requirement artifact type — forward-looking tooling, ecosystem, and LLM agent UX requirements (implemented in M3/M4) |
| `how/tooling/{NAME}.md` | How | Individual Tooling Requirement documents, one per requirement, status-tracked (open / deferred / incorporated) |

Convention: **one file, one coherent topic**. Do not create a file titled "Miscellaneous" or "Various."

---

## 4. Language & Style

### 4.1 Tone

- **Precise** over poetic. Avoid metaphor where direct language suffices.
- **Concise** over verbose. Prefer a short sentence to a long paragraph.
- **Authoritative** over speculative. Design documents state decisions, not opinions.
- **Consistent** terminology (see §6 Terminology).

### 4.2 Code Examples

Every code example in documentation should:

1. Be syntactically valid Orthon (or clearly marked as pseudocode).
2. Demonstrate exactly one concept.
3. Include **all canonical forms** when documenting a feature (see the *Show All Canonical Forms* principle in `how/DESIGN_PRINCIPLES.md`).
4. Precede semantic explanation rather than follow it.

### 4.3 File Structure

```
docs/
├── AGENTS.md                 # Meta — agent instructions
├── README.md                 # Project root reference
├── LICENSE
├── why/                      # WHY — purpose, vision, philosophy
│   ├── VISION.md
│   ├── DESIGN_INFLUENCES.md
│   ├── GOALS.md
│   ├── MANIFESTO.md
│   └── ZEN.md
├── what/                     # WHAT — language design & reference
│   ├── CORE_CONCEPTS.md      # Accepted concept registry (currently empty; see how/concepts/research/)
│   ├── GLOSSARY.md
│   ├── THESES.md             # Distilled understanding anchors (status-tracked)
│   ├── SEMANTIC_MODEL.md     # Unified semantic model (Phase 2)
│   ├── PRIMITIVE_BLOCKS.md   # Minimal orthogonal primitive blocks (Phase 3)
│   ├── LIBRARY_BOUNDARY.md   # Language vs stdlib vs external (Phase 4)
│   ├── SYNTAX.md             # Syntax hub — principles + pointers (Phase 5)
│   ├── syntax/               # Accepted syntax records (one per construct)
│   ├── CROSS_CUTTING.md      # Interaction matrix (Phase 6)
│   ├── CONFLICT_REGISTRY.md  # Concept-boundary conflicts (Phase 6)
│   ├── EXECUTION_MODEL.md    # Execution semantics (Phase 7)
│   ├── OPTIMIZATION_MODEL.md # Semantics vs optimisation (Phase 7)
│   └── concepts/             # Accepted Orthon concept drafts (see README.md)
│       └── README.md
├── how/                      # HOW — implementation & process
│   ├── DESIGN_PRINCIPLES.md  # 27 design rules organized in 3 groups
│   ├── process/              # Design process documents
│   │   ├── DECISION_PROCESS.md   # One-page decision authority (Phase 1)
│   │   └── DECISION_PIPELINE.md  # 10-question feature pipeline (Phase 4)
│   ├── EVOLUTION_MODEL.md    # Versioning, deprecation, feature gates (Phase 8)
│   ├── SYNTAX_PIPELINE.md    # Syntax acceptance pipeline (Phase 5, EDR-087)
│   ├── syntax/               # Syntax hypothesis inbox + decision queue (Phase 5)
│   ├── concepts/             # Concept design pipeline
│   │   ├── README.md
│   │   ├── research/         # Concept research inbox (raw analyses, triaged by tier)
│   │   │   ├── README.md
│   │   │   ├── essential/    # Must-have concepts — semantic bedrock
│   │   │   ├── important/    # Important concepts — usability & expressiveness
│   │   │   ├── deferrable/   # Nice-to-have — deferrable to v0.2/v0.3
│   │   │   ├── reject/       # Contradicts principles — rejection candidates
│   │   │   └── ... (110+ research files across tiers)
│   ├── IMPLEMENTATION_POLICIES.md
│   ├── architecture/         # Compiler architecture
│   │   ├── ARCHITECTURE.md
│   │   ├── PARSER.md
│   │   ├── TYPE_SYSTEM.md
│   │   ├── NAME_RESOLUTION.md
│   │   └── IR.md
│   ├── strategies/           # Implementation strategies
│   │   ├── IMPLEMENTATION_STRATEGIES.md
│   │   ├── DEFAULT_STRATEGY.md
│   │   ├── EMBEDDED_STRATEGY.md
│   │   └── HIGH_PERFORMANCE_STRATEGY.md
│   ├── decision_records/    # Engineering Decision Records
│   │   ├── INDEX.md          #   Unified EDR journal
│   │   ├── architecture/     #   Architecture-category EDRs
│   │   ├── process/          #   Process-category EDRs
│   │   ├── quality/          #   Quality-category EDRs
│   │   ├── technology/       #   Technology-category EDRs
│   │   ├── tedr.md
│       ├── _edr-architecture.md
│       ├── _edr-process.md
│       ├── _design-review.md
│       └── _concept    #   Delivery-category EDRs
│   │   ├── operations/       #   Operations-category EDRs
│   │   ├── security/         #   Security-category EDRs
│   │   ├── governance/       #   Governance-category EDRs
│   │   ├── data/             #   Data-category EDRs
│   │   ├── ai/               #   AI-category EDRs
│   │   ├── documentation/    #   Documentation-category EDRs
│   │   ├── knowledge/        #   Knowledge-category EDRs
│   │   ├── collaboration/    #   Collaboration-category EDRs
│   │   └── product/          #   Product-category EDRs
│   ├── gates/
│   │   ├── DECISION_VALIDATION.md
│   │   └── _language-design.md
│   └── templates/
│       ├── _edr.md
│       ├── _edr-architecture.md
│       ├── _edr-process.md
│       ├── _design-review.md
│       └── _concept.md
└── when/                     # WHEN — roadmap, milestones
```

### 4.4 Document Naming Conventions

All files in `docs/` follow a consistent naming pattern:

| Convention | Rule | Examples |
|------------|------|----------|
| **Content documents** | UPPER_CASE, single topic per file | `VISION.md`, `CORE_CONCEPTS.md`, `DESIGN_PRINCIPLES.md` |
| **Category directories** | Lowercase, plural | `why/`, `what/`, `how/`, `when/`, `decision_records/`, `templates/`, `gates/` |
| **Fill-in templates** | `_` prefix + kebab-case inside `templates/` | `how/templates/_edr.md`, `how/templates/_design-review.md` |
| **Gate checklists** | `_` prefix + kebab-case inside `gates/` | `how/gates/_language-design.md` |
| **EDR records** | `EDR-NNN-title-with-dashes.md` inside `decision_records/{category}/` | `how/decision_records/architecture/EDR-042-event-sourcing.md` |
| **Glossary / reference** | UPPER_CASE, `GLOSSARY.md` | `what/GLOSSARY.md` |
| **Syntax hypothesis / record** | UPPER_CASE with `_SYNTAX` suffix, in `how/syntax/` (hypotheses) or `what/syntax/` (accepted) | `TYPE_ANNOTATION_SYNTAX.md`, `RANGE_SYNTAX.md` |

**Rules:**

1. **Content docs** live inside their layer directory (`why/`, `what/`, `how/`), use `UPPER_CASE.md`.
2. **`AGENTS.md`** stays at `docs/` root as the single entry-point for agent instructions.
3. **Templates** (fill-in forms) use the `_` prefix so they sort first in directory listings. Place them inside `how/templates/`.
4. **Category directories** (`why/`, `what/`, `how/`, `when/`, `decision_records/`, `templates/`, `gates/`) are lowercase, plural nouns.
5. **Actual records** (filled-in EDRs, completed gates) drop the `_` prefix — they are content, not forms.
6. **Do not nest directories deeper than `docs/{layer}/{category}/`** unless explicitly justified.
7. **When adding a new document type**, create a matching category directory and template, then update this section and §3 Document Map.

---

## 5. Agent Workflow

When assigned a task in this project, follow this protocol:

### 5.1 Orient

1. **Assert language.** Before any other step, assert: *"All file content I produce will be in English."* Chat responses to the user may match the user's language. The project language is English (§10.9).
2. **Read the relevant layer first.** If the task is about a concrete feature, start with `how/concepts/research/` (concept research). `what/CORE_CONCEPTS.md` is the acceptance destination but is currently empty — no concepts have been accepted yet. If it is about a principle decision, start with `why/VISION.md` and `how/DESIGN_PRINCIPLES.md`.

2a. **Read module headers first.** Before reading any implementation file or searching for import statements, read the module header (`module ... use ... effects ...`). The header is the semantic index of the module — it declares dependencies, effects, and public API. Do not grep for imports across files; the header is the single source of truth for a module's dependencies.

2b. **Place new concept research in the correct tier directory.** When adding a
    new concept research document to `how/concepts/research/`, do NOT place it
    directly in the root. Instead, put it in the appropriate tier subdirectory
    (`essential/`, `important/`, `deferrable/`) based on how foundational the
    concept is to Orthon's semantic identity. Meta-files (anti-pattern analyses,
    language comparisons, reference indices) stay in the root.
3. **Check cross-references.** A design decision in one document may affect documents in other layers.
4. **Run the Decision Pipeline** (`how/process/DECISION_PIPELINE.md`) before designing any new feature — 10 questions determine whether the feature should exist and at what level.
5. **Check `how/gates/_language-design.md`** if the task involves making a design decision — the gate defines the acceptance criteria.

### 5.2 Design

1. **Start from first principles.** Anchor every proposal in the project's existing philosophy. If a proposal contradicts `why/MANIFESTO.md` or `why/ZEN.md`, the proposal must either be rejected or the philosophy documents must be updated (and the change must be intentional, not accidental).
2. **Favor minimal additions.** Ask: *"Can this be expressed through composition of existing concepts?"* before introducing a new concept.
3. **Prefer named over symbolic.** If adding an operator, also define its named function equivalent.
4. **Document alternatives.** When proposing a design choice, briefly note what was considered and why it was rejected.
5. **Preserve orthogonality.** Every new construct must combine freely with existing constructs. No special cases.

### 5.3 Gate

Before finalizing any new or modified design document:

1. **Verify language compliance.** Scan every string of new or modified text. If any content is not in English, translate it before proceeding.
2. **Verify against the `how/gates/_language-design.md`** checklist. If the gate does not exist yet or is empty, propose a gate entry for the decision.

### 5.4 Write

1. Place the document in the correct `docs/` location.
2. Follow the document's existing structure and heading style.
3. Use Markdown (`*.md`).
4. Write in English (see §10.9).
5. Use fenced code blocks with `orthon` as the language tag for Orthon code.
6. Use `text` for non-Orthon structural diagrams.
7. Keep table of contents in sync if the document uses one.

---

## 6. Terminology (Ubiquitous Language)

All project terminology is defined in [`what/GLOSSARY.md`](what/GLOSSARY.md).

Use terms **consistently** and **always** in their defined meaning. When introducing a new term, add it to `what/GLOSSARY.md` and cross-reference the source document.

Key terms every agent must know:

- **Data** — the primary abstraction; values without imposed semantic meaning.
- **Data Modifier** — transforms data between representations.
- **Representation** — a specific view of data (Value, Tuple, Reference, Sequence, Set, Option, Result).
- **Sequence** — values produced over time; describes *what*, not *how*.
- **Orthogonality** — each construct solves one problem and combines freely.

→ See [`what/GLOSSARY.md`](what/GLOSSARY.md) for the complete reference with cross-document links.

---

## 7. Design Decision Protocol

Every design decision follows the **Decision Process** defined in
[`how/process/DECISION_PROCESS.md`](how/process/DECISION_PROCESS.md).
Before detailed design, run through the **Decision Pipeline** in
[`how/process/DECISION_PIPELINE.md`](how/process/DECISION_PIPELINE.md)
to determine whether the feature should exist and at what level.

Substantive design decisions must be recorded. Use one of these mechanisms, ordered by increasing formality:

### 7.1 Inline (for small decisions)

Add a note at the end of the relevant section:

```markdown
> **Decision:** Tuples are immutable by default.
> **Rationale:** Consistency with the data-first model — data does not mutate.
> **Alternatives considered:** Mutable tuples rejected because they conflate identity with value.
```

### 7.2 Gate Entry (for decisions that need review)

If the decision is significant enough to warrant a review pass, add an entry to `how/gates/_language-design.md`:

```markdown
### Gate 003: Sequence Emission Syntax

**Status:** Pending

**Criteria:**
- [ ] `emit` keyword semantics are defined.
- [ ] `return sequence(...)` equivalence is documented.
- [ ] `->` operator equivalence is documented.
- [ ] All three forms compile to the same semantics.
```

### 7.3 EDR (for consequential engineering decisions)

For decisions with long-lasting impact — architectural choices, principle changes,
process decisions, or trade-offs that affect multiple documents — create an
Engineering Decision Record in `docs/how/decision_records/{category}/`.

Use the base template at [`docs/how/templates/_edr.md`](../how/templates/_edr.md)
or a category-specific template (`_edr-architecture.md`, `_edr-process.md`).
Name the file `EDR-NNN-title-with-dashes.md` where `NNN` is the next sequential
number (see [`decision_records/INDEX.md`](../how/decision_records/INDEX.md)).

EDRs are appropriate when:
- The decision affects how future design work is evaluated.
- The rationale needs to be *findable* years later.
- The decision supersedes or deprecates a previous EDR.

### 7.4 Human Sign-off (mandatory before EDR)

Every Tier 1–2 EDR requires a prior **Human Sign-off**: the solo author
explicitly confirms the design is baked — `Reviewed-by: <author> · Date:
YYYY-MM-DD · Verdict: [LOCKED / changes]`. This is a **non-automatable**
gate: agents/GSD flows must not self-certify it. See the Concept Design
Review Step 7 (Convergence Check) and `SYNTAX_PIPELINE.md` Stage 6b. An
EDR may not be filed until the sign-off is populated.

### 7.5 Theses Capture (for distilled understanding anchors)

When the author asks to save a thesis ("save thesis <Thesis>", or the equivalent
in another language), record it as a **Thesis** in [`what/THESES.md`](what/THESES.md)
— not as specification:

1. Add the entry following the filed format — **Status**, **Source**, **Thesis**,
   plus optional elaboration. `Source` must point to the canonical document
   (concept doc, EDR, or glossary entry).
2. If the thesis introduces or redefines a term, update
   [`what/GLOSSARY.md`](what/GLOSSARY.md) as well (alphabetical; `Source` and
   `See also` links).
3. Default status is `DRAFT`; use `VERIFIED` only once the claim has been checked
   against its canonical source.
4. Incorrect or superseded theses are marked `REJECTED` and kept (anti-memory) —
   never silently deleted.
5. Theses about keywords or syntax (`let`, `var`, literals) MUST carry an `orthon`
   code block after the elaboration — one example per claim.

Theses are distilled, verifiable claims that restore understanding. They are not
specification; canonical truth stays in the linked sources. THESES.md lives in
`what/` (semantics layer), not `how/`.

---

## 8. Proposal Structure

When proposing a new language feature or design change, structure the proposal as follows:

```markdown
## Feature: [Name]

### Layer
Why / How / What

### Problem
What gap or inconsistency does this address?

### Proposal
The concrete design.

### Canonical Forms
All equivalent ways to express the feature.

### Interaction with Existing Concepts
How this composes with Data, Data Modifiers, Sequence, etc.

### Alternatives Considered
What was rejected and why.

### Impact on Existing Documents
Which docs need updating.

### Gate Criteria
Checklist for `how/gates/_language-design.md`.
```

For proposals that warrant formal peer review, use the template at [`docs/how/templates/_design-review.md`](../how/templates/_design-review.md).

---

## 9. Review Checklist

Before finalizing any document or proposal, verify against this checklist. For formal peer reviews, use the full template at [`docs/how/templates/_design-review.md`](../how/templates/_design-review.md).

- [ ] **Consistency:** Does this align with `why/MANIFESTO.md` and `how/DESIGN_PRINCIPLES.md`?
- [ ] **Orthogonality:** Does this combine freely with existing constructs?
- [ ] **Minimality:** Is this adding a new concept when composition would suffice?
- [ ] **Explicitness:** Are semantic changes syntactically visible?
- [ ] **EDR:** Does this decision warrant an EDR in `docs/how/decision_records/`?
- [ ] **All canonical forms:** Are all equivalent forms documented?
- [ ] **English (MANDATORY):** Scan every string of text — headings, body, code comments, captions, table cells. Is every single one in English? If any non-English text exists, this check FAILS. Do not mark pass until all text is English.
- [ ] **Terminology:** Are project terms used consistently with [`what/GLOSSARY.md`](what/GLOSSARY.md)?
- [ ] **Gate:** Does the decision have a gate entry if needed?
- [ ] **EDR:** Does this decision warrant an EDR in `docs/how/decision_records/`?
- [ ] **Cross-references:** Are related documents linked?

---

## 10. AI Agent Constraints

Agents operating in this repository must follow these rules:

1. **Read first.** Never propose a change without reading the existing document(s) that the change affects.
2. **No code generation.** This is a design-only repository. Do not generate compiler, runtime, or tooling implementation code. Code snippets in documentation are for illustration only.
3. **No external dependencies.** Do not introduce dependencies on external frameworks, tools, or standards unless explicitly justified in the proposal.
4. **Preserve document structure.** Do not rearrange `docs/` hierarchy without updating `AGENTS.md` and cross-references.
5. **One intention per commit.** When making changes, group edits by topic, not by file.
6. **Gate before merge.** Any design decision must pass the gate before being considered final.
7. **When in doubt, reference `why/ZEN.md`.** The Zen aphorisms are the shortest path to a guiding principle.
8. **Verify paths.** Every cross-reference (link) added or modified in a
   document must resolve to an existing file in the repository, relative
   to the source document's location. This applies to all referenced
   files — concept docs, gate checklists, strategies, ADRs, templates,
   and any other path — not only those under `what/concepts/`.
   Additionally, cross-references to documents under `what/concepts/`
   must use the `concepts/` prefix (e.g., `concepts/CORE_CONCEPTS.md`,
   not `CORE_CONCEPTS.md`), as these files reside in a subdirectory and
   bare filenames would not resolve. The same rule applies to
   `how/concepts/research/` — use the full relative path from the
   source document.
9. **English in files.** All documentation, code snippets, comments, commit messages,
   and any text written to files MUST be in **English**. This is a non-negotiable
   project rule. Chat responses to the user (including explanations, clarifications,
   and suggestions) may be in the user's language.

   **Enforcement:** Before creating or modifying any file, the agent MUST confirm that
   every string of text to be written is in English. Any content found in another
   language must be translated before writing. Chat responses are exempt from this
   check — the agent may answer in the language the user wrote in.
10. **Commit message prefixes.** Every commit message MUST use a conventional prefix
    to indicate the type of change. Use one of:

    | Prefix | When to use |
    |--------|-------------|
    | `docs:` | Documentation-only changes (concept docs, spec, research) |
    | `feat:` | New language feature or concept |
    | `fix:`  | Correction of an error in existing documentation |
    | `chore:` | Maintenance tasks (templates, config, tooling) |
    | `refactor:` | Restructuring without semantic change |
    | `meta:` | Changes to agent instructions or process docs |

    **Format:** `prefix: short description`

    ```
    docs: add gradual typing concept research document
    chore: update EDR template with new fields
    ```

    The body may contain additional detail after a blank line. Multiple
    changes in the same commit should share the same prefix; if they do
    not, prefer separate commits.
11. **Heading anchors keep underscores.** GitHub anchors are generated from the
    heading text verbatim: `### EXECUTION_CONTEXT_INVOCATION` anchors to
    `#execution_context_invocation`, not `#execution-context-invocation`. Verify a
    heading exists before linking to it — do not invent anchors (for example,
    `what/GLOSSARY.md` has no `### Scope` heading).
12. **Index tables are duplicated.** `how/decision_records/INDEX.md` lists EDRs in
    BOTH the main (All Records) table and the By-Category table. A new EDR row must
    be added to both, and because row text repeats, edits need unique surrounding
    context.
13. **Cross-reference depth from research tiers.** Links from
    `how/concepts/research/{tier}/` to decision records use
    `../../../decision_records/...` (matching the `RANGE_SLICE.md` precedent).

---

## 11. Working With the Author

Behavioural expectations for any agent working in this repository. These apply to
_interaction_, not to file content (see §10.9 for file-language rules).

### 11.1 Connected Prose

Formulate questions, points, and review comments as coherent prose — a self-contained
train of thought that opens with the logic, builds with explicit connecting words, and
lands on a conclusion. Avoid lists of weakly connected jargon terms: technical terms
belong inside sentences, not as bare labels. A paragraph of connected reasoning beats a
bullet list of fragments.

### 11.2 One Question at a Time

When reviewing a concept or design document with the author:

1. Restate the author's questions and list them first (numbered).
2. Answer them **one at a time**, never all at once; wait for the author's follow-up
   on the current question.
3. Iterate on that single question until the outcome is **LOCKED** — a document
   clarification, a partial or full concept revision plus EDR, or an explicit
   "not a problem — skip".
4. State the lock explicitly, then ask whether to move to the next question.
5. Compact the context between questions when the session grows long.
6. Expect questions to be added mid-review ("add one more question ...").

Answering everything at once mixes concerns and produces decisions that do not hold.
A review may not fit in one session; leftover questions are reviewed in a separate one.

### 11.3 Uncertainty Before Solution

Do not jump to a solution while uncertainty is open. Widen first (stakeholder
questions, assumptions, metrics, options), then narrow (criteria, plan), then verify
against real data, people, or the system. Generating options is cheap; removing
uncertainty is not. A plausible answer to the wrong problem is the default failure mode.

### 11.4 Close the Branch: Merge and Cleanup

When work on a task branch has been reviewed and the author confirms the change is
correct, do not leave the branch open-ended. Remind the author of the two closing
steps and perform each only on explicit confirmation: first offer to merge the task
branch into `main` and push it, and then, once the merge has landed on `main`, offer
to delete the task branch both locally and on `origin`. The reminder is the agent's
responsibility, but merging into `main` and deleting a branch remain the author's
decisions — never do either without an explicit go-ahead.

---

## 12. Workstation & Research Notes

Environment-specific notes for this workstation — how the tooling here actually behaves,
so that work is not blocked. They are **not** project rules.

→ See [`notes/agent-environment.md`](notes/agent-environment.md) (macOS CLI quirks, paywalled-source research).

