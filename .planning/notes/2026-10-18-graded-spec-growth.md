---
date: "2026-10-18 00:00"
promoted: false
---

# Graded Specification Growth: MVP First, Then Strengthen

> **Exploratory note.** Outcome of a `/gsd-explore` session on 2026-10-18. Decisions below are the author's; claims marked *unverified* come from model memory or keyword search and were not checked against primary sources.

## The idea

Accepted concepts are not equally necessary. Instead of specifying and freezing all of them at once, mark each with a **grade** and grow the specification in stages:

| Grade | Meaning | Ships in |
|-------|---------|----------|
| 1 | Maximally necessary: the MVP language | v0.1 |
| 2 | Important | v0.2 |
| 3 | Useful but not required (example: unpacking: convenient, but expressible without it) | v0.3 |

Goal: raise complexity gradually, strengthening the MVP with already-agreed concepts rather than attempting the absolute language at once.

## Decisions

1. **Variant B: the specification itself grows by grade.** v0.1 contains grade 1 only. It is not merely an implementation order for a complete v0.1 spec. Consequences: Core Value ("every concept accepted") and Phase 8 (freeze) must be rescoped to grade 1; Phases 5 and 6 shrink (pairwise interaction checking is quadratic in the number of concepts, so cutting the concept set cuts that work superlinearly).
2. **A grade is the cost of deferral, not importance.** The question asked of every concept: *what would have to be rewritten or would break if this concept arrived one grade later?* Unpacking: nothing (it desugars to lower-grade constructs), so grade 3. Generics, null safety, error handling, ownership, concurrency: they change the meaning of already-accepted constructs, so a late arrival is expensive regardless of how "optional" they feel.
3. **Before grading, verify that the primitive set is closed, layer by layer.** The dynamic core (values, calls, control), static semantics (types) and staging (macros, comptime) each need their own closed set. "No new primitive will ever be needed" cannot be proven; it can only be made falsifiable and given an amendment procedure (precedent: EDR-085 added the 10th primitive and re-ran the minimality proof).
4. **Naming.** The new concept must not reuse "tier" (already three meanings, see [[2026-10-17-spec-level-complexity-management]], task T3). "Grade" is the working name.
5. **Already-accepted grade 2/3 concepts stay accepted (decided 2026-10-18).** They are not returned to hypothesis status and their EDRs are not superseded. Registry status takes the form `Accepted (grade N, target vX)`, extending the existing precedent `Accepted (deferred to v0.2)` of PERSISTENT_DATA_STRUCTURES. Rationale: re-opening would repeat design work on the same information; new information arrives only with Phase 5 scenarios and the implementation (Milestone 10). Two rules keep "frozen" from meaning "sealed":
   - **Pre-release gate.** Before a concept ships in its own version it passes the Concept Design Review gates again against the specification as it then stands; any change goes through a normal EDR amendment.
   - **Grade promotion.** If grade 1 would have to be designed around a deferred concept, the concept's grade is wrong and it is promoted, not reopened.

6. **Process EDR now, Core Value later (decided 2026-10-18).** The grade mechanism is a Tier 3 decision (how design work is done, [`DECISION_PROCESS.md`](../../how/process/DECISION_PROCESS.md)): it needs an EDR in the Process category (template `_edr-process.md`), no human sign-off (that rule covers Tier 1-2). The EDR records: the definition of a grade, the cost-of-deferral question, the status format `Accepted (grade N, target vX)`, the pre-release gate, the promotion rule, and the relation to Decision Pipeline Q10 ("Worth adding?", which already yields informal "Yes, but deferred" verdicts). It is written at the start of Phase 04.5. Redefining v0.1 is a separate, later step: Core Value in `.planning/PROJECT.md` (and its copies: the "Definition of Done" Key Decision, the `.planning/REQUIREMENTS.md` header, the M1 goal and Phase 8 in `when/ROADMAP.md`) is rewritten only at the end of Phase 04.5, after grade 1 closure is demonstrated. Reason: the mechanism is harmless and reversible; moving the v0.1 boundary before closure is shown risks rolling it back, and a freeze is close to irreversible.
   - The Core Value clause "every concept accepted (no DRAFT)" stays true under decision 5. What changes is that "complete" and "freeze" apply to grade 1.

7. **Strict one-way layer order (decided 2026-10-18).** Dynamic core ← static semantics ← staging; an arrow means "depends on". Staging may read types (`@typeInfo`, type-checking of macro output) and run dynamic-core code in the comptime interpreter; the dynamic core and the type system never depend on staging. Evidence in the accepted concepts: EDR-046 (closed set of 8 type intrinsics, non-recursive, evaluated by the compiler during type checking), no value-dependent types or const generics, EDR-029 (single-pass macro expansion before type-checking the expanded code, no macro depends on another's expansion). Consequences:
   - Staging is a translator from source to a program of the lower layers (like Racket's expansion to a fully-expanded program), not a set of kernel primitives. Its audit question is "does expansion always land in the lower layers and terminate", not "which primitives does it add".
   - Staging concepts (AST_MACROS, COMPILE_TIME_EXECUTION, derive macros, user-visible contracts machinery) become natural candidates for a later grade, provided nothing in grade 1 stands on them.
   - **The one place the rule is violated today:** EDR-031 says comptime *is* the generic mechanism (`comptime T: type`, a type as a compile-time value), while EDR-086 moved generics to `<>`. The rule resolves the open question in favour of `<>`: a type is never a comptime value; `comptime` parameters are values only. `COMPILE_TIME_EXECUTION` OQ 4 and the stale conditional "if `comptime` is adopted" in `GENERICS.md` must be reconciled (EDR-031 amendment).
   - **A monotonicity hazard already visible:** NAMED_AND_OPTIONAL_PARAMETERS "desugar to positional calls via macro layer" on `@derive(init)` and AST macros. If named arguments are grade 1, either macros are grade 1 too or named arguments are implemented as a compiler built-in; otherwise they get a later grade.

## Findings about the current primitive set (preliminary)

- [`what/PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) § 10 declares the 10-primitive set complete ("no gaps"). The verification was done per category batch and, for deferrable concepts, "conceptually", and predates the current form of the 58 accepted concepts. "Decomposes onto the primitives" is not defined as a checkable artifact.
- The primitive list has **no branching, loop or jump primitive**. `ITERATION_LOOP` says "`loop` → primitive infinite loop; `break`/`continue` → primitive control flow", while § 10.3 says the control-flow category decomposes onto `function + call + scope` with no gap. These two statements contradict each other.
- The `Primitive Decomposition` rows of [`what/CORE_CONCEPTS.md`](../../what/CORE_CONCEPTS.md): a keyword search (*beyond primitive composition*, *compiler-level*, *not primitive-expressible*, *compiler-generated/-enforced/-recognized*) matches **about 32 of 58 rows** (*unverified count, keyword-based; some matches are benign derive/sugar*). Rough clusters of the residue:
  - Control flow (`?`, `?.`, loop/break, match, suspend/resume): about 6 concepts. Candidate missing primitive.
  - Type level (generics, union/intersection, literal types, ADT tags, `!T`, inference, narrowing, structural and gradual typing): about 11 concepts. A separate layer, not a runtime primitive.
  - Staging (macros, comptime, contracts, module effect annotations): about 5 concepts. A separate layer.
  - Static analysis (definite assignment, lifetime tracking, error-set inference): about 3 concepts. Checks, not constructs.

## Proposed method: Primitive Closure Audit

1. **Kernel as the oracle of closure.** For each layer, fix a kernel (the "fully expanded program" grammar). Anything outside it must be derivable. Reference points, *from memory, unverified*: Racket's fully-expanded-program grammar (everything outside it is a macro); Oz's kernel language (Van Roy & Haridi, everything else is a "linguistic abstraction"); Haskell Core and Rust MIR as typed kernels.
2. **Derivation table**, one row per accepted concept construct: construct → expression over the layer's kernel → residue class (pure composition / StdLib / type level / staging / static analysis / **new-primitive candidate**). Any new-primitive candidate blocks freezing the kernel.
3. **Adversarial constructs**: try to derive coroutines, re-entrant exceptions, effect handlers, transactions, channel `select` from the kernel. A failure here is a cheap early signal.
4. **Amendment procedure** for a later gap: an EDR in the style of EDR-085 plus a re-run of the minimality proof.

## Experience of other languages *(from memory, unverified)*

Worked: Go (generics added in 1.18 after about ten years, without breaking earlier code); Rust (1.0 as a deliberate MVP, async/await in 1.39, const generics as an "MVP" subset); Racket teaching languages (Beginning → Intermediate → Advanced Student: explicit subsets with per-level checking); Scheme (R7RS-small vs R7RS-large); Haskell (2010 core plus extension sets).

Hurt: Java generics (retrofitted via type erasure for bytecode compatibility); Go before generics (`interface{}` ecosystem); async/await retrofits in several languages (function colouring, interaction with errors and lifetimes); null.

Lesson: deferring is safe for things that desugar to lower grades or that can be added without changing the meaning of existing constructs. It is unsafe for things that change core semantics.

## Risks of variant B

- **Forward compatibility.** If v0.1 ignores a grade 2/3 construct, it might claim its syntax or give it different meaning, and adding it later breaks v0.1 programs. Needs a reserved-syntax list for later grades (Phase 5 must sketch grade 2/3 syntax enough to reserve space).
- **Monotone dependencies.** A grade 1 concept must not depend on a grade 2 concept; check against the dependency graph from Phase 04.3 task T4.
- **Closure of the MVP.** Without an implementation, "the MVP is usable" can only be checked with scenario programs (task T9 of the complexity note). Grade 1 needs an explicit closure criterion: non-trivial programs can be written with it alone.

## Open questions (not yet decided)

1. ~~Accepted concepts of grade 2/3: frozen or returned to hypothesis?~~ **Resolved 2026-10-18**: frozen, with a pre-release gate (Decision 5).
2. ~~Process EDR and Core Value?~~ **Resolved 2026-10-18**: Process EDR (Tier 3) at the start of Phase 04.5; Core Value rewritten at its end (Decision 6).
3. ~~Layer dependencies?~~ **Resolved 2026-10-18**: strictly ordered, types never depend on staging (Decision 7).
4. How does Phase 6 treat deferred concepts: only their boundary against grade 1 (reserved syntax, no grade 1 dependency), or a full pairwise check?
5. *Proposed, not yet confirmed.* The grade scale applies to the 58 accepted concepts only. The roughly 50 concepts in `how/concepts/research/deferrable/` are not accepted; they stay hypotheses outside the scale and may enter it later through the Decision Pipeline. Confirm and state it in the Process EDR.

## Planned placement

Phase 04.4 (Primitive Closure Audit) and Phase 04.5 (Concept Grading and MVP Slice), inserted after Phase 04.3. Phase 04.5 consumes both the audit result and the dependency metadata produced by Phase 04.3.
