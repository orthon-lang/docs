---
phase: 02-semantic-model-define-identity-ownership-mutation-evaluation
reviewed: 2026-07-27T00:00:00Z
depth: standard
files_reviewed: 4
files_reviewed_list:
  - how/decision_records/architecture/EDR-013-semantic-model.md
  - what/SEMANTIC_MODEL.md
  - how/decision_records/INDEX.md
  - what/GLOSSARY.md
findings:
  critical: 3
  warning: 4
  info: 4
  total: 11
status: issues_found
---

# Phase 02: Code Review Report

**Reviewed:** 2026-07-27
**Depth:** standard
**Files Reviewed:** 4
**Status:** issues_found

## Summary

Reviewed the four Phase 2 deliverables: EDR-013 (the acceptance record),
`what/SEMANTIC_MODEL.md` (the six-dimension semantic contract it accepts),
`how/decision_records/INDEX.md` (the master EDR journal), and
`what/GLOSSARY.md` (the terminology registry). The prose is careful and the
cross-dimension reasoning is mostly rigorous, but three correctness-grade
defects survive: (1) the Cross-Dimension Consistency table's central claim —
"fifteen pairwise interactions... documented" — is false as written, because
one of the fifteen slots (Visibility ↔ Lifetime) is silently replaced by a
cross-cutting summary row rather than an actual pairwise analysis; (2) the
Ownership section contradicts its own central invariant across two of its
own canonical code examples (one shows a named binding moved via bare `=`
with zero visible marker, the other treats the identical operation as a
compile error requiring an explicit marker); and (3) EDR-013's own
"Relationship to Other Records" table cites "EDR-011 (LLM Generability
Gate)," but the project's master index (`INDEX.md`) assigns EDR-011 to a
different, unrelated record ("Process Inventory") — the LLM Generability
Gate EDR exists on disk under the same ID in a different category
directory and is not listed in the index at all, i.e., a duplicate,
unregistered EDR ID that EDR-013 nonetheless cites as authoritative.

None of these are cosmetic: all three directly undermine either the
model's own "all fifteen pairs are orthogonal or resolved" claim, the
model's own worked Ownership example, or the traceability of an Accepted
EDR's cross-reference — exactly the properties `LOGICAL_CONSISTENCY_GATE`
and the EDR system are supposed to guarantee.

## Structural Findings (fallow)

None provided for this review (no `<structural_findings>` block was
supplied).

## Narrative Findings (AI reviewer)

## Critical Issues

### CR-01: Cross-Dimension Consistency table omits the Visibility ↔ Lifetime pair while claiming full 15-pair coverage

**File:** `what/SEMANTIC_MODEL.md:564-644`
**Issue:** The section header states "Six dimensions produce fifteen
pairwise interactions" and both EDR-013 (`EDR-013-semantic-model.md:111-113,
202-206`) and the model's own `LOGICAL_CONSISTENCY_GATE` verdict
(`SEMANTIC_MODEL.md:713`) rely on the claim that "all fifteen pairwise
interactions are documented" / "resolves all fifteen pairwise
interactions." Enumerating the table's 15 rows by dimension pair:

```
1 Identity–Ownership   6 Ownership–Mutation    11 Mutation–Visibility
2 Identity–Mutation    7 Ownership–Evaluation  12 Mutation–Lifetime
3 Identity–Evaluation  8 Ownership–Visibility  13 Evaluation–Visibility
4 Identity–Visibility  9 Ownership–Lifetime    14 Evaluation–Lifetime
5 Identity–Lifetime   10 Mutation–Evaluation   15 Lifetime–All (not a pair)
```

Row 15 is explicitly *not* a pairwise interaction — it is labeled
"Lifetime ↔ All" and the row's own text says "Lifetime is therefore not
merely 'one more pairwise interaction'" (line 638-639). With row 15
excluded as a genuine pair, only 14 of the 6-choose-2 = 15 required pairs
are actually analyzed, and **Visibility ↔ Lifetime is never addressed
anywhere in the document** — not in the table, not in a "Detail" callout,
not in the per-dimension sections (Visibility's section never mentions
Lifetime; Lifetime's section never mentions Visibility). This is exactly
the class of gap the review was asked to check for, and it directly
contradicts the accepted EDR's compliance claim ("all fifteen pairwise
dimension interactions are documented," `EDR-013-semantic-model.md:202-206`)
and the `LOGICAL_CONSISTENCY_GATE` Pass verdict's evidence
(`SEMANTIC_MODEL.md:713`), which cites this exact table as satisfying
"Composition ... documented for all existing concepts."
**Fix:** Add an explicit row (or fold it into row 15 but state it
explicitly) analyzing Visibility ↔ Lifetime, e.g.: "Orthogonal — a
value's visibility level (`priv`/default/`pub`) says nothing about how
long it lives, and a value's scope-bound lifetime says nothing about who
may reach it; a `priv` value and a `pub` value are destroyed by the same
scope-exit rule." Then correct the "fifteen pairwise interactions" claims
in both files to match whatever is actually enumerated, or add the
missing analysis so the claim becomes true.

### CR-02: Ownership section's own canonical examples contradict Semantic Invariant 6

**File:** `what/SEMANTIC_MODEL.md:196-202` vs. `what/SEMANTIC_MODEL.md:211-221`
**Issue:** Semantic Invariant 6 states: "Ownership transfer is
semantically explicit... The *fact* that a value's ownership changes
hands must always be visible at the transfer site" (`SEMANTIC_MODEL.md:60-65`).
The Ownership section restates this as: "a transfer must be syntactically
explicit... whenever the source is a named binding"
(`SEMANTIC_MODEL.md:223-234`). Yet the section's own first code example
directly violates this:

```
data = create_resource()
other = data              # move: data is now invalid
# use(data)                # compile error: data was moved
use(other)                 # OK
```

Here `other = data` transfers ownership of an already-named binding
(`data`) via a bare `=` — no `move`, `$`, `@ownership`, or any other
marker appears anywhere in the example, yet the comment asserts this is a
legal "move." Compare this to the Fresh-value exemption example twelve
lines later in the *same section*:

```
consume(create_resource())   # fresh value — no transfer marker needed
existing = create_resource()
consume(existing)             # ERROR in strict mode — existing is a named
                               # binding and requires an explicit transfer
```

Here, passing an existing named binding (`existing`) without an explicit
marker is declared a compile **ERROR**. Both examples perform the same
operation — transferring ownership of an already-bound, named value —
but one shows it succeeding silently with zero visible syntax and the
other shows it failing without an explicit marker. This is a direct,
provable self-contradiction within the same section of the same accepted
document, and it undermines the very invariant (Semantic Invariant 6)
the section exists to establish.
**Fix:** Make the first example consistent with the invariant it is
illustrating — either use the placeholder baseline syntax the document
itself names (`move`) to keep the example syntax-agnostic while still
visible, e.g. `other = move data`, or add a comment clarifying that this
specific example intentionally elides the (TBD) transfer marker and is
not itself a canonical illustration of Invariant 6. As written, a reader
(human or LLM) cannot tell from this document alone whether `other = data`
is legal Orthon or a compile error.

### CR-03: EDR-013 cites "EDR-011 (LLM Generability Gate)," but INDEX.md assigns EDR-011 to a different record — duplicate/unregistered EDR ID

**File:** `how/decision_records/architecture/EDR-013-semantic-model.md:235`,
`how/decision_records/INDEX.md:24,59`
**Issue:** EDR-013's "Relationship to Other Records" table states:
"EDR-011 (LLM Generability Gate) | The `LLM_GENERABILITY_GATE` criteria
from that EDR were applied..." However, `how/decision_records/INDEX.md`
— the project's declared "master index" / "unified journal of all
engineering decisions" — registers EDR-011 as **Process Inventory**
(`process/EDR-011-process-inventory.md`), a completely unrelated record
about cataloguing process tools. A second file,
`how/decision_records/architecture/EDR-011-llm-generability-gate.md`,
does exist on disk and is genuinely about the LLM Generability Gate (its
own header even reads `# EDR-011: LLM Generability Gate`), but **it is
not listed anywhere in INDEX.md** — not in "All Records," not in "By
Category" → Architecture (which lists only EDR-010, EDR-012, EDR-013).
Per the project's own convention ("ID: EDR-NNN (globally unique)" and
"master index at `decision_records/INDEX.md`," per `GLOSSARY.md`'s EDR
entry), this is a duplicate ID collision plus an incomplete index:
EDR-013 cites a real file by an ID that the canonical index has already
assigned to a different decision, so a reader following EDR-013's
citation through the index will land on the wrong document.
**Fix:** Renumber one of the two EDR-011 files to a free slot (e.g., the
reserved-but-unfilled EDR-009 gap noted in `INDEX.md`'s Gap Registry, or
the next available ID after EDR-014), update all cross-references
(including `DECISION_VALIDATION.md:279`'s `EDR-011-llm-generability-gate`
citation and EDR-013's own reference), and add the renumbered record to
`INDEX.md`'s "All Records" and "By Category" tables.

## Warnings

### WR-01: EDR-013 claims "exclusive access" is registered in GLOSSARY.md, but no such entry exists

**File:** `how/decision_records/architecture/EDR-013-semantic-model.md:207-210`
**Issue:** EDR-013's Compliance section states: "all new terminology
introduced by this synthesis (binding identity, value identity,
**exclusive access**, the `fun`/`proc`/`new` declaration kinds, etc.) is
registered per the Glossary maintenance workflow." `what/GLOSSARY.md` has
no "Exclusive Access" entry (verified: the only occurrence of the phrase
in the glossary is inside the *definition* of "Semantic Invariant," not
as its own heading). The other four named terms (Binding Identity, Value
Identity, Declaration Kind, and by extension the seven glossary
additions actually made — see `DECISION_LOG.md:78`, which explicitly
counts "seven new terms") do not include "Exclusive Access" as one of the
seven.
**Fix:** Either add a dedicated "Exclusive Access" glossary entry (cross-
referencing Semantic Invariant 2 and both the Ownership and Mutation
sections it bridges), or correct EDR-013's Compliance section to drop
"exclusive access" from the list of registered terms so the claim matches
what was actually done.

### WR-02: INDEX.md's Status Summary count is stale (says 11 Accepted, table lists 12)

**File:** `how/decision_records/INDEX.md:74-79` vs. `INDEX.md:16-27`
**Issue:** The "Status Summary" table reports `Accepted | 11`. Counting
the rows in "All Records" (`INDEX.md:16-27`): EDR-001, 002, 003, 004,
005, 006, 007, 010, 011, 012, 013, 014 — 12 rows, all with Status
`Accepted`. The "By Category" tables sum to the same total (Architecture
3 + Process 8 + Quality 1 = 12). The Status Summary was not updated when
EDR-013 and/or EDR-014 were added.
**Fix:** Update `Accepted | 11` to `Accepted | 12` in the Status Summary
table.

### WR-03: EDR-013's "one non-trivial coupling" framing understates the model's own Cross-Dimension Consistency table

**File:** `how/decision_records/architecture/EDR-013-semantic-model.md:111-116`
**Issue:** EDR-013 states: "the one non-trivial coupling identified —
Ownership and Mutation both instantiating a single shared invariant... —
is accepted as intentional, not as an orthogonality violation." But the
Cross-Dimension Consistency table in `SEMANTIC_MODEL.md` itself assigns
non-"Orthogonal" relationship labels to *four* other pairs beyond
Ownership↔Mutation: pair 8 (Ownership↔Visibility, "Edge case documented"),
pair 9 (Ownership↔Lifetime, "Bridges via scope"), pair 10
(Mutation↔Evaluation, "Interacts at evaluation order"), and pair 15
(Lifetime↔All, "Cross-cutting by construction"). Framing the model as
having exactly "the one" non-trivial coupling is inconsistent with the
model's own table, which documents five pairs requiring more than a bare
"Orthogonal" label. This is a characterization mismatch between the EDR's
summary and the artifact it summarizes, not a fatal defect, but it could
mislead a future reader (or an LLM ingesting the EDR alone) into
under-counting the interactions that require special handling.
**Fix:** Either qualify the EDR's language (e.g., "the sharpest of several
documented couplings" instead of "the one non-trivial coupling"), or
explicitly state why pairs 8/9/10/15 don't count as "couplings" in the
same sense as pair 6, so the two documents' characterizations agree.

### WR-04: GLOSSARY.md structural defects — duplicate/misfiled headings and broken alphabetical order

**File:** `what/GLOSSARY.md:60-76, 153-260, 285-318, 586-626`
**Issue:** Several structural problems exist in the glossary file under
review, in violation of the project's own documented convention
("Alphabetical order. Each section uses the term's first letter as a
heading anchor" — `what/GLOSSARY.md:698`):
- Duplicate `### Data` heading: an empty stub heading at line 62
  (immediately followed by `### Dependency Flow` with no content of its
  own) and the real, populated "Data" entry again at line 75.
- Duplicate `### Explicit Optimization` heading with identical content at
  lines 208 and 243.
- A `## D (cont.)` section (line 123) placing "Decision Log" and
  "Decision Validation" *after* "Declaration Kind," "Declaration Model,"
  and "Deterministic Behavior" — alphabetically, "Decision" (Dec-i)
  precedes "Declaration" (Dec-l), so these entries are out of order and
  were evidently appended rather than merged in-place.
- Entries misfiled under the wrong letter section entirely: "Program
  Enricher" (line 305) sits under the `## I` header instead of `## P`;
  "Stable Mental Model" (line 610) and "Standard Library" (line 617) sit
  under the `## V` header instead of their own sections.
- "Execution Program" (line 215) appears *after* "Explicit Optimization"
  (line 208) under `## E`, breaking alphabetical order (Execution <
  Explicit).
**Fix:** Consolidate the duplicate "Data" and "Explicit Optimization"
entries into single entries, move "Program Enricher," "Stable Mental
Model," and "Standard Library" under their correct letter headers, merge
the `## D (cont.)` entries back into `## D` in alphabetical position, and
re-sort `## E` so "Execution Program" precedes "Explicit Optimization."

## Info

### IN-01: Unsourced "~95%" statistic repeated without citation or methodology

**File:** `what/SEMANTIC_MODEL.md:179`, `how/decision_records/architecture/EDR-013-semantic-model.md:73`
**Issue:** Both documents assert "an estimated 95%" / "~95% of
ordinary-value code never needs to reason about [ownership]" as if it
were a measured fact, with no source, methodology, or even a qualifier
like "we estimate" beyond the word "estimated" itself. This is a stylistic
nit for a documentation-only spec, but a load-bearing-sounding number
like this in an Accepted EDR could be mistaken for empirical grounding it
doesn't have.
**Fix:** Either soften to explicitly-qualitative language ("the large
majority of ordinary-value code") or add a footnote clarifying this is an
informed guess, not a measured statistic.

### IN-02: Inconsistent pair-name ordering between table row and "Detail" callout

**File:** `what/SEMANTIC_MODEL.md:583, 627`
**Issue:** Table row 8 is labeled "Ownership ↔ Visibility," but its
corresponding elaboration is headed "Detail: Visibility ↔ Ownership (pair
8)" — the dimension order is reversed between the two. Purely cosmetic,
but inconsistent naming direction elsewhere (all other Detail headings
match their row's order: "Identity ↔ Ownership," "Ownership ↔ Mutation,"
"Mutation ↔ Evaluation," "Lifetime ↔ All").
**Fix:** Rename to "Detail: Ownership ↔ Visibility (pair 8)" for
consistency.

### IN-03: Borderline Why-layer motivational aside inside the What-layer document

**File:** `what/SEMANTIC_MODEL.md:143-146`
**Issue:** "This mirrors Swift's `struct`-by-default, `class`-by-opt-in
model rather than Java's reference-by-default model" is a comparative,
motivational aside rather than a specification statement. It's brief and
doesn't rise to a full Why-layer argument, but per the project's own
anti-pattern guidance ("Why → What Smuggling": philosophical rationale
belongs in `why/`, concrete specification in `what/`, linked rather than
restated), this is the kind of sentence worth watching as the document
grows — it explains *why this feels familiar* rather than *what the rule
is*.
**Fix:** No immediate action required; if future edits add more such
comparisons, consider moving them to a "Prior Art" aside in the EDR or a
cross-reference to a Why-layer document instead of inline in the What
spec.

### IN-04: Top-of-document link text doesn't match its target heading

**File:** `what/SEMANTIC_MODEL.md:9-12`
**Issue:** The status banner links to `[Design Principles
verification](#relationship-to-design-principles)`, but the actual
heading at that anchor is "## Relationship to Design Principles" (line
648) — the anchor resolves correctly, but the visible link text
("Design Principles verification") doesn't match the section's real
title, which could make it harder for a reader to confirm they landed on
the intended section, or to find the section again by title search.
**Fix:** Change the link text to "Relationship to Design Principles" to
match the actual heading.

---

_Reviewed: 2026-07-27_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
