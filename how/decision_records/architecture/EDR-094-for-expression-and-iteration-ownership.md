# EDR-094: For-Expression and Iteration Ownership — Amends EDR-053

**Status:** Accepted

**Date:** 2026-09-29

**Category:** Architecture

**Scope:** Subsystem

**Human Sign-off:** Reviewed-by: mniedre · Date: 2026-09-29 · Verdict: LOCKED

**Amends:** [EDR-053](./EDR-053-iteration-loop.md) — this EDR resolves EDR-053 Open
Questions 2 (break-with-value in `for`), 3 (`for`-`else`), and 4 (iteration
ownership), and **partially supersedes** EDR-053's statement-only reading of `for`:
in value position `for` now becomes an expression yielding `Optional<T>`.

---

### Context

EDR-053 (2026-07-27) accepted Orthon's loop model — one iteration construct
(`for ... in`), a condition construct (`while`), and an infinite `loop` — but
deferred three questions to `what/concepts/ITERATION_LOOP.md`'s Open Questions:

- **OQ2** — should `break` carry a value in `for` (as it may in `loop`)?
- **OQ3** — should `for` have a Python-style `else` clause (run when no `break` fired)?
- **OQ4** — how does iteration interact with ownership: does `for item in coll`
  consume or borrow the collection?

A human-signed concept review (2026-09-28, sign-off 2026-09-29) settled all three.
This EDR records that settlement as a durable amendment, following the EDR-092
amending-EDR precedent.

The forces at play:

- **Minimal core** — the search-and-produce-one-value pattern is worthy of support
  but must not earn a new loop type or a new comprehension marker; it reuses `for`
  and the already-accepted `Optional<T>` (EDR-018).
- **One symbol → one meaning** — reusing `else` for a loop's no-break branch
  overloads a symbol that already means "otherwise", so `for`-`else` is rejected.
- **Division of labour with EDR-092** — production earns syntax (`gen(...)`);
  aggregation stays a combinator (`.find`, `.fold`). The `for`-expression is scoped
  to Optional search only; general accumulation remains a combinator.
- **Intentional divergence from Rust** — Rust's bare `for` moves the collection;
  Orthon's bare `for` borrows (shared read), keeping the collection alive by default
  and making mutation and consumption explicit at the source.
- **Inheritance, not definition** — ITERATION_LOOP *inherits* the ownership /
  mutability vocabulary (`&`, `&mut`, `$`) from Reference, OWNERSHIP, and MUTABILITY;
  it does not define that spelling. The boundary is recorded here as deferrals, not
  decided.

---

### Decision

1. **`for` is an expression yielding `Optional<T>` (resolves OQ2, affirmatively).**
   In value position, `for` produces an `Optional<T>`:
   - `break v` and `return v` yield `Some(v)`;
   - running off the end without a break yields `None`;
   - a bare `break` (no value) in expression position yields `None` early.

   The loop's type in value position is `Optional<T>`. In **statement position**,
   `for` yields no value (unchanged from EDR-053). The simple single-predicate case
   stays the combinator `coll.find(|x| ...)` (`Option<T>`); the `for`-expression is
   for **multi-statement bodies with early exit**, where a predicate closure is
   awkward. Scope is **Optional search only** — general accumulation (`fold` /
   `reduce`) remains a combinator and the expression form is not widened to it.

   ```orthon
   # for-expression: multi-statement search with early exit
   let found = for row in table:
       let parsed = parse(row)
       if parsed.is_valid() and parsed.key == target:
           break parsed          # -> Some(parsed)
   # run off the end -> None ; found : Optional<Row>

   # simple predicate stays a combinator
   let first_even = numbers.find(|n| n % 2 == 0)   # Option<Int>
   ```

2. **`for`-`else` is rejected (resolves OQ3, negatively).** A Python-style `else`
   clause on `for` is not adopted: it is foreign syntax that overloads `else` (which
   already means "otherwise"), breaking one-symbol-one-meaning, and it adds control-flow
   complexity by overload. The search-and-not-found pattern it served is fully covered
   by the Optional `for`-expression plus `match` (Decision 1), so nothing is lost. This
   rejection is retained as anti-memory in Alternatives Considered.

   ```orthon
   match (for x in xs:
             if hit(x): break x):
       Some(x) -> use(x)
       None    -> not_found()
   ```

3. **Iteration is borrow-by-default with three source-side ownership markers
   (resolves OQ4).** Bare `for x in coll` is a shared read borrow — the collection
   survives the loop. Modification and consumption are explicit, expressed by
   ownership markers **on the source**, inherited from the language-wide vocabulary
   (NOT loop-specific `@iter_mut()` / `@borrow()` / `@grant()` methods):

   - `for x in coll` — shared read borrow; element type `&T`.
   - `for x in &mut coll` — exclusive write-borrow; element type `&mut T`; requires
     `mut coll`.
   - `for x in $coll` — move / consume; element type `T`; consumes the binding
     (rebind required after).

   **Structural mutation is NOT a `for`-mode.** A live iterator borrow freezes the
   composition (iterator-invalidation safety, provided by the borrow rules, not a
   special loop rule); changing length / membership (`coll.push` / `coll.remove`)
   happens *outside* the loop and requires `mut coll`. **Interior mutability**
   (immutable collection with mutable elements) is **not introduced in v0.1**,
   preserving a clean transitive model (`let` deeply immutable, `let mut` a mutable
   path) — recorded as a recommendation to MUTABILITY OQ1.

   The iteration matrix (verbatim from the concept review):

   | Mode (loop side) | Borrow | Composition | Element values | `x` type | Needs `mut coll` | Collection after |
   |---|---|---|---|---|---|---|
   | `for x in coll` | shared | frozen | immutable | `&T` | no | same, unchanged |
   | `for x in &mut coll` | exclusive | frozen | **mutable** | `&mut T` | **yes** | same, elements changed |
   | `for x in $coll` | move | dismantled | handed out by value | `T` | — (consumes) | **gone**, rebind required |
   | `coll.push/remove(...)` *(outside loop)* | exclusive, no iterator | **mutable** | — | — | **yes** | composition changed |

   **Authority frame** adopted to explain the model: a **binding** is authority held
   over a storage slot (`mut` = write authority, non-`mut` = read-only); a **grant** is
   a permanent transfer of that authority (`$` / move / reassign); a **rent** is a
   temporary loan (a reference / borrow), split by the aliasing invariant into a
   **read-rent** (`&T`, many concurrent) XOR a **write-rent** (`&mut T`, exactly one).
   Five properties the model carries:
   1. **Aliasing invariant** — many read-rents XOR one write-rent, never both.
   2. **Derivation rule** — one cannot lend more than one holds: a write-rent
      (`&mut coll`) requires a `mut` binding.
   3. **Lifetime / return** — a rent ends automatically at loop scope (nothing is
      returned by hand); a grant never returns.
   4. **Grant consumes the holder** — `$coll` needs no `mut` (it is not in-place
      mutation) but leaves the binding dead, so a rebind is required.
   5. **Level** — grant / rent in a loop operate on **elements**; the structural axis
      is separate and lives outside `for`.

4. **Boundary flags B1–B4 are recorded as deferrals, not decided here.** These are
   spelling / ownership questions owned by other concepts; ITERATION_LOOP inherits
   their final resolution:
   - **B1** — ITERATION_LOOP inherits ownership / mutability markers; it does not
     define them. The `&` / `&mut` / `$` spelling is owned by **Reference** (GLOSSARY
     § Reference — "two forms: shared read-only reference and exclusive mutable
     reference") plus **OWNERSHIP** (borrow / aliasing rules) and **MUTABILITY**
     (`mut` / `&mut`).
   - **B2** — the `$` vs `move` glyph is open in **OWNERSHIP_TRANSFER_OPERATOR**
     (competing with an `@ownership` metaproperty); the loop inherits the final glyph.
   - **B3** — the `&mut` spelling depends on **MUTABILITY OQ3** (`mut` vs `&mut`); the
     loop inherits it.
   - **B4** — `&coll` cannot be repurposed as write-borrow: `&` already denotes the
     shared read-only reference. The `$` (one operation) vs `&` + `&mut` (two flavours)
     asymmetry is inherent — move is a single operation, while borrow is read XOR write
     forced by the aliasing invariant, so the more powerful write form is the more
     marked.

   These concepts are research-stage (`how/concepts/research/essential/`); this EDR
   asserts no acceptance of their spellings.

---

### Consequences

- **Positive:**
  - Three long-standing Open Questions on EDR-053 are closed, faithfully to a
    human-signed review.
  - Borrow-by-default keeps collections alive across a loop — the common case needs
    no marker, and mutation / consumption are explicit and visible at the source.
  - Source-side markers (`coll` / `&mut coll` / `$coll`) reuse the language-wide
    ownership vocabulary rather than inventing loop-specific methods, which is highly
    LLM-generable and keeps one mental model across the language.
  - The `for`-expression reuses the already-accepted `Optional<T>` (EDR-018) instead of
    adding a new loop type or comprehension marker — minimal core preserved.
  - The iteration matrix is a single, verbatim reference shared by the EDR and the
    concept spec.
- **Negative:**
  - Intentional divergence from Rust's move-by-default `for` — a reader arriving from
    Rust must relearn that Orthon's bare `for` borrows.
  - `for` now returns a value in value position — a wrinkle to learn versus a pure
    statement loop.
  - The concrete ownership spelling (`&mut` vs `mut`, `$` vs `move`) remains pending on
    boundary flags B1–B4 and their owning concepts; ITERATION_LOOP tracks, not fixes, it.

---

### Compliance

1. `what/concepts/ITERATION_LOOP.md` records the OQ2 / OQ3 / OQ4 resolutions, the
   `for`-expression (Optional) form, and the ownership matrix + source-side markers.
2. `coll.find(|x| ...)` remains the simple-predicate combinator; `fold` / `reduce`
   remain combinators — the `for`-expression is not widened beyond Optional search.
3. No `for`-`else` construct and no loop-specific `@iter_mut` / `@borrow` / `@grant`
   method appears as a language construct in accepted (`what/`) documents.
4. The `&` / `&mut` / `$` spelling is not asserted as accepted here — it is inherited
   from Reference / OWNERSHIP / MUTABILITY once those concepts settle (B1–B4).

---

### Alternatives Considered

> Populated from the concept-review discussion (2026-09-28). Retained as anti-memory.

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Bare comprehension `expr for x in src where cond` for the search form | Conflicts with EDR-092 (the bare parenthesised comprehension was withdrawn; no production marker), and `where` duplicates `gen(... if cond)`'s `if`. |
| `opt(...)` / `find(...)` comprehension marker | Duplicates `.find` and dresses *aggregation* as a *production* marker. The division of labour is fixed — production → syntax (`gen`), aggregation → combinator — and finding the first match is not boilerplate, so it does not earn syntax. |
| `until` clause on the loop | An extra loop keyword overlapping `while`'s negation; adds a construct with no new semantics. |
| `for`-`else` (Python-style no-break branch) | Foreign syntax that overloads `else` (already "otherwise"), breaking one-symbol-one-meaning. The pattern is covered by the Optional `for`-expression + `match`, so nothing is lost. |
| Loop-specific `@iter_mut()` / `@borrow()` / `@grant()` methods | Rejected in favour of source-side ownership markers inherited from the language-wide vocabulary; loop-specific methods would fork the ownership model and duplicate meaning already carried by `&` / `&mut` / `$`. |

### Gate Validation

> This EDR adds semantics (the `for`-expression and the iteration-ownership model are
> not pure sugar), so the full seven-gate Architecture table is reproduced, modelled on
> EDR-053's own gate table.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | Stated in programmer terms: "I need to search a collection with a multi-statement body and get the first match, or nothing." The `for`-expression returns `Optional<T>` directly; borrow-by-default means the everyday loop needs no ownership marker. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | `Some`/`None` production rules are total (break-with-value, run-off-end, bare break all defined); the aliasing invariant (read-rent XOR write-rent) and the derivation rule (write-rent needs a `mut` binding) are mutually consistent and match EDR-018's Optional. No self-referential paradoxes. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "search-and-produce-one-value needs no new construct." Confirmed — it reuses `for` + `Optional<T>`; a new loop type or comprehension marker would be redundant. Ownership reuses one language-wide marker set rather than loop-specific methods. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | The `for`-expression sits in the same Core Language layer as EDR-053's loop and reuses EDR-018's Optional; ownership markers are *inherited* from Reference / OWNERSHIP / MUTABILITY (B1–B4), so no layer is bypassed and no ownership spelling is defined here. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Borrow / move / write-borrow are strategy-independent authority relations; whether the collection is GC'd, ref-counted, or arena-allocated, the `&T` / `&mut T` / `T` element contracts hold. Structural-mutation-outside-the-loop is a rule about iterators, not about any allocation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "`for` borrows by default and yields `Optional<T>` in value position; add `&mut` to write elements, `$` to consume." The authority frame (binding / grant / rent) is a single durable mental model that extends beyond loops. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Source-side markers are explicit at the point of use, so an LLM reads intent directly from the loop header; the Optional result matches the pervasive `Option`/`Optional` corpus. Self-correction: `&mut coll` without `mut coll` is a compile error; using a moved `$coll` binding afterwards is a use-after-move error. |

**Gates not applied:** None — all seven gates are required (this EDR adds semantics).

**Detailed reasoning:** See `DECISION_LOG.md` and the concept-review record
(2026-09-28, sign-off 2026-09-29) for the per-gate reasoning trail.
