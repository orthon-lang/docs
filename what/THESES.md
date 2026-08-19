# Theses — Distilled Understanding Anchors

> A compact, reviewable store of distilled theses about Orthon's language
> semantics. Each thesis is a single, verifiable claim that restores
> understanding of a concept at a glance.
>
> Theses are **not specification**. Canonical truth lives in the linked
> sources (concept docs, EDRs, GLOSSARY). A thesis is a memory anchor
> that points back to that truth.

---

## Format

Each thesis follows this shape:

```markdown
### <Concept> — <short label>

**Status:** DRAFT | VERIFIED | SUPERSEDED | REJECTED
**Source:** <canonical document or EDR>

**Thesis:** <one verifiable claim>

> <optional elaboration, counterpoints, what it does NOT claim>
```

**Verification rule.** A thesis is only as good as its source. A thesis
marked `VERIFIED` has been checked against its canonical source (concept
doc, EDR, GLOSSARY). `DRAFT` theses are unverified and must not be relied
on to restore understanding. Incorrect theses are kept as `REJECTED`
entries — they serve as anti-memory so the error is not repeated.

**Routing.** Once verified, a thesis may be promoted into its canonical
destination (GLOSSARY term, concept doc, or EDR), then marked
`SUPERSEDED` here with a pointer to the new home.

---

## Theses

### T (non-optional) — contract of guarantee

**Status:** VERIFIED
**Source:** [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md),
[EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md),
[`what/concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md)

**Thesis:** A non-optional type `T` is a **contract of guarantee**: a
function returning `T` always produces a value of type `T`, and can
never return `None` — the compiler forbids assigning `None` to a
non-optional `T`.

> Corollary: `T` and `Option<T>` are distinct, incompatible types.
> The `!` operator produces `T` at its usage site (explicit "I know this
> is non-null" escape hatch) — it does not narrow the original variable.

### Option&lt;T&gt; — contract of handling

**Status:** VERIFIED
**Source:** [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md),
[EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md),
[`what/concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md)

**Thesis:** `Option<T>` is a **contract of handling**: a function
returning `Option<T>` *may* produce `None`, and the compiler forces the
caller to handle both `Some(T)` and `None` — pattern matching on
`Option` must be exhaustive.

> Corollary: after a `match`/guard establishes a value is `Some(T)`, the
> compiler narrows the type to `T` in that branch (flow-sensitive
> narrowing, EDR-028).

### None — a distinct type, not assignable to non-optional T

**Status:** VERIFIED
**Source:** [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md),
[`what/concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md)

**Thesis:** `None` is a value of its own distinct type `None`, not of
type `T` and not of type `Option<T>`. Assigning `None` to a non-optional
`T` is a compile-time error; it is assignable only where `Option<T>` is
expected.

> Corollary: `None` is the `None` variant of `Option<T>` — a value of
> type `None` coerces into `Option<T>` because `None` is one of its two
> variants. There is no `null` sentinel that silently inhabits `T`.
