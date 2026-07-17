# AGENTS.md — Agent Guide for Orthon

> Instructions for AI agents contributing to the Orthon language design project.

---

## 1. Project Identity

Orthon is a **programming language design project**. This repository contains *only* design documentation — no implementation code, no build system, no runtime. The entire project exists in the `docs/` directory.

The language is designed using the same SOLID engineering principles it encourages in its users: a small, orthogonal core with layered abstraction, explicit semantics, and implementation-independent evolution.

---

## 2. Documentation Framework

All documentation follows the **Why → How → What** (Golden Circle) framework:

| Layer | Question | Documents |
|-------|----------|-----------|
| **Why** | Why does Orthon exist? What do we believe? | `why/VISION.md`, `why/MANIFESTO.md`, `why/ZEN.md` |
| **How** | How is Orthon designed and structured? | `what/DESIGN_PRINCIPLES.md`, `what/ARCHITECTURE.md`, `how/IMPLEMENTATION_STRATEGIES.md`, `how/IMPLEMENTATION_POLICIES.md` |
| **What** | What is Orthon concretely? | `what/CORE_CONCEPTS.md`, `what/DATA_MODEL.md`, future syntax and type specs |

An agent must **always** anchor new content to the correct layer. A "Why" argument must not be smuggled into a "What" document. If a new piece of documentation spans layers, split it or add cross-references.

---

## 3. Document Map

| File | Layer | Purpose |
|------|-------|---------|
| `why/VISION.md` | Why | Core philosophy, inspiration from Python/Java, Principle of Least Astonishment, orthogonality |
| `why/MANIFESTO.md` | Why | Explicit principles — consistency over legacy, minimal core, composition over exceptions |
| `why/ZEN.md` | Why | Aphorisms capturing the language's spirit |
| `what/DESIGN_PRINCIPLES.md` | How | Orthogonality, simplicity, explicitness, consistency, execution model principles |
| `what/ARCHITECTURE.md` | How | Layered architecture — Core Language → Syntax → Standard Library → Implementation Strategy |
| `how/IMPLEMENTATION_STRATEGIES.md` | How | How strategies provide interchangeable implementations |
| `how/IMPLEMENTATION_POLICIES.md` | How | Policy-level decisions for implementation work |
| `what/CORE_CONCEPTS.md` | What | Data and Data Modifiers — the two fundamental abstractions |
| `what/DATA_MODEL.md` | What | Formal data model specification |
| `AGENTS.md` | Meta | This file — instructions for AI agents |
| `what/GLOSSARY.md` | Meta | Unified terminology reference with cross-document links |
| `how/templates/_adr.md` | Meta | ADR template (fill-in form) |
| `how/templates/_design-review.md` | Meta | Design review template (fill-in form) |
| `how/adr/ADR-*.md` | Meta | Architecture Decision Records — logged per-decision |
| `how/gates/_language-design.md` | Meta | Quality gate checklist for language design decisions |

Convention: **one file, one coherent topic**. Do not create a file titled "Miscellaneous" or "Various."

---

## 4. Language & Style

### 4.1 English Only

All documentation, code snippets, comments, commit messages, and agent reasoning MUST be written in **English**. This is a non-negotiable project rule.

### 4.2 Tone

- **Precise** over poetic. Avoid metaphor where direct language suffices.
- **Concise** over verbose. Prefer a short sentence to a long paragraph.
- **Authoritative** over speculative. Design documents state decisions, not opinions.
- **Consistent** terminology (see §6 Terminology).

### 4.3 Code Examples

Every code example in documentation should:

1. Be syntactically valid Orthon (or clearly marked as pseudocode).
2. Demonstrate exactly one concept.
3. Include **all canonical forms** when documenting a feature (see the *Show All Canonical Forms* principle in `what/DESIGN_PRINCIPLES.md`).
4. Precede semantic explanation rather than follow it.

### 4.4 File Structure

```
docs/
├── AGENTS.md                 # Meta — agent instructions
├── README.md                 # Project root reference
├── LICENSE
├── why/                      # WHY — purpose, vision, philosophy
│   ├── VISION.md
│   ├── MANIFESTO.md
│   └── ZEN.md
├── what/                     # WHAT — language design & reference
│   ├── ARCHITECTURE.md
│   ├── CORE_CONCEPTS.md
│   ├── DATA_MODEL.md
│   ├── DESIGN_PRINCIPLES.md
│   └── GLOSSARY.md
├── how/                      # HOW — implementation & process
│   ├── IMPLEMENTATION_STRATEGIES.md
│   ├── IMPLEMENTATION_POLICIES.md
│   ├── adr/
│   │   └── ADR-*.md
│   ├── gates/
│   │   └── _language-design.md
│   └── templates/
│       ├── _adr.md
│       └── _design-review.md
└── when/                     # WHEN — roadmap, milestones
```

### 4.5 Document Naming Conventions

All files in `docs/` follow a consistent naming pattern:

| Convention | Rule | Examples |
|------------|------|----------|
| **Content documents** | UPPER_CASE, single topic per file | `VISION.md`, `CORE_CONCEPTS.md`, `DESIGN_PRINCIPLES.md` |
| **Category directories** | Lowercase, plural | `why/`, `what/`, `how/`, `when/`, `adr/`, `templates/`, `gates/` |
| **Fill-in templates** | `_` prefix + kebab-case inside category dir | `how/templates/_adr.md`, `how/templates/_design-review.md` |
| **Gate checklists** | `_` prefix + kebab-case inside `gates/` | `how/gates/_language-design.md` |
| **ADR records** | `ADR-NNN-title-with-dashes.md` inside `adr/` | `how/adr/ADR-001-tuple-immutability.md` |
| **Glossary / reference** | UPPER_CASE, `GLOSSARY.md` | `what/GLOSSARY.md` |

**Rules:**

1. **Content docs** live inside their layer directory (`why/`, `what/`, `how/`), use `UPPER_CASE.md`.
2. **`AGENTS.md`** stays at `docs/` root as the single entry-point for agent instructions.
3. **Templates** (fill-in forms) use the `_` prefix so they sort first in directory listings. Place them inside the category directory they belong to.
4. **Category directories** (`why/`, `what/`, `how/`, `when/`, `adr/`, `templates/`, `gates/`) are lowercase, plural nouns.
5. **Actual records** (filled-in ADRs, completed gates) drop the `_` prefix — they are content, not forms.
6. **Do not nest directories deeper than `docs/{layer}/{category}/`** unless explicitly justified.
7. **When adding a new document type**, create a matching category directory and template, then update this section and §3 Document Map.

---

## 5. Agent Workflow

When assigned a task in this project, follow this protocol:

### 5.1 Orient

1. **Read the relevant layer first.** If the task is about a concrete feature, start with `what/CORE_CONCEPTS.md` and `what/DATA_MODEL.md`. If it is about a principle decision, start with `why/VISION.md` and `what/DESIGN_PRINCIPLES.md`.
2. **Check cross-references.** A design decision in one document may affect documents in other layers.
3. **Check `how/gates/_language-design.md`** if the task involves making a design decision — the gate defines the acceptance criteria.

### 5.2 Design

1. **Start from first principles.** Anchor every proposal in the project's existing philosophy. If a proposal contradicts `why/MANIFESTO.md` or `why/ZEN.md`, the proposal must either be rejected or the philosophy documents must be updated (and the change must be intentional, not accidental).
2. **Favor minimal additions.** Ask: *"Can this be expressed through composition of existing concepts?"* before introducing a new concept.
3. **Prefer named over symbolic.** If adding an operator, also define its named function equivalent.
4. **Document alternatives.** When proposing a design choice, briefly note what was considered and why it was rejected.
5. **Preserve orthogonality.** Every new construct must combine freely with existing constructs. No special cases.

### 5.3 Gate

Before finalizing any new or modified design document, verify against the `how/gates/_language-design.md` checklist. If the gate does not exist yet or is empty, propose a gate entry for the decision.

### 5.4 Write

1. Place the document in the correct `docs/` location.
2. Follow the document's existing structure and heading style.
3. Use Markdown (`*.md`).
4. Write in English (see §4.1).
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

### 7.3 ADR (for foundational or cross-cutting decisions)

For decisions with long-lasting impact — architectural choices, principle changes, or trade-offs that affect multiple documents — create an Architecture Decision Record in `docs/how/adr/`.

Use the template at [`docs/how/templates/_adr.md`](../how/templates/_adr.md). Name the file `ADR-NNN-title-with-dashes.md` where `NNN` is the next sequential number.

ADRs are appropriate when:
- The decision affects how future design work is evaluated.
- The rationale needs to be *findable* years later.
- The decision supersedes or deprecates a previous ADR.

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

- [ ] **Consistency:** Does this align with `why/MANIFESTO.md` and `what/DESIGN_PRINCIPLES.md`?
- [ ] **Orthogonality:** Does this combine freely with existing constructs?
- [ ] **Minimality:** Is this adding a new concept when composition would suffice?
- [ ] **Explicitness:** Are semantic changes syntactically visible?
- [ ] **Named equivalence:** If symbolic, does the named function form exist?
- [ ] **All canonical forms:** Are all equivalent forms documented?
- [ ] **English:** Is all content in English?
- [ ] **Terminology:** Are project terms used consistently with [`what/GLOSSARY.md`](what/GLOSSARY.md)?
- [ ] **Gate:** Does the decision have a gate entry if needed?
- [ ] **ADR:** Does this decision warrant an ADR in `docs/how/adr/`?
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
