# Design Review Template

> A structured template for peer-reviewing design proposals in the Orthon language project.
> Use this when reviewing a new language feature, a change to an existing concept, or a cross-document update.

---

## Review Metadata

| Field | Value |
|-------|-------|
| **Proposal** | Link or name of the proposal under review |
| **Reviewer** | Name / agent identifier |
| **Date** | YYYY-MM-DD |
| **Review type** | [Initial review | Follow-up | Gate sign-off] |
| **Documents affected** | List of docs touched by the proposal |

---

## 1. Philosophical Alignment

Is the proposal consistent with the project's core beliefs?

| Criterion | ✅ Pass | ⚠️ Flag | ❌ Block | Notes |
|-----------|---------|---------|---------|-------|
| Aligned with `../../why/VISION.md` | | | | |
| Aligned with `../../why/MANIFESTO.md` | | | | |
| Aligned with `../../why/ZEN.md` | | | | |
| Aligned with `../DESIGN_PRINCIPLES.md` | | | | |

---

## 2. Orthogonality & Minimality

Does the proposal respect the principle of orthogonal design?

| Criterion | ✅ Pass | ⚠️ Flag | ❌ Block | Notes |
|-----------|---------|---------|---------|-------|
| Solves exactly one problem | | | | |
| Composes freely with existing constructs | | | | |
| No overlap with existing concepts | | | | |
| Could be expressed via composition instead | | | | |

---

## 3. Explicitness

Are semantic changes visible in syntax?

| Criterion | ✅ Pass | ⚠️ Flag | ❌ Block | Notes |
|-----------|---------|---------|---------|-------|
| No hidden conversions | | | | |
| Lifetime / ownership changes are explicit | | | | |
| Behaviour is predictable without compiler knowledge | | | | |

---

## 4. Completeness

Does the proposal cover all necessary ground?

| Criterion | ✅ Pass | ⚠️ Flag | ❌ Block | Notes |
|-----------|---------|---------|---------|-------|
| All canonical forms documented | | | | |
| Named function equivalent exists (if symbolic) | | | | |
| Interaction with existing concepts described | | | | |
| Alternatives documented with rejection rationale | | | | |
| Impact on affected documents assessed | | | | |

---

## 5. Terminology & Language

| Criterion | ✅ Pass | ⚠️ Flag | ❌ Block | Notes |
|-----------|---------|---------|---------|-------|
| Terms used consistently with `../../what/GLOSSARY.md` | | | | |
| All content is in English | | | | |
| Examples are valid Orthon (or clearly marked as pseudocode) | | | | |
| Cross-references to existing docs are correct | | | | |

---

## 6. Structural Integrity

| Criterion | ✅ Pass | ⚠️ Flag | ❌ Block | Notes |
|-----------|---------|---------|---------|-------|
| Document placed in correct `docs/` location | | | | |
| Follows existing heading style and conventions | | | | |
| `AGENTS.md` Document Map updated if new file added | | | | |
| Gate entry exists if required | | | | |

---

## 7. Summary

### Overall Verdict

- [ ] **Accept** — No blockers, ready to merge.
- [ ] **Accept with flags** — Minor issues to address (see flags above), no conceptual disagreement.
- [ ] **Revise and re-review** — One or more blocking issues must be resolved before sign-off.
- [ ] **Reject** — Fundamental misalignment with project philosophy; proposal should not proceed.

### Key Concerns

1. ...
2. ...
3. ...

### Recommendations

1. ...
2. ...
3. ...

---

## 8. Re-Review Log

| Round | Date | Verdict | Key Changes Since Last Round |
|-------|------|---------|------------------------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |

---

## Guide for Reviewers

- **✅ Pass** — The criterion is fully satisfied.
- **⚠️ Flag** — Minor concern; does not block acceptance but should be discussed or addressed.
- **❌ Block** — The proposal cannot proceed without resolving this issue.
- A single **Block** in any category means the overall verdict defaults to *Revise and re-review*.
- If a criterion is not applicable, leave it blank and add a note.
