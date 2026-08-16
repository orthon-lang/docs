---
date: "2026-08-15 00:00"
promoted: false
---

## Colon overload analysis — is "one symbol, one meaning" achievable?

### The question

The Manifesto declares "One concept — one syntax", and `DESIGN_PRINCIPLES.md`
declares **Semantic Purity**: "Each symbol or syntactic construct has exactly
one meaning, independent of context. No exceptions." Is this achievable in
practice? The `:` symbol is the test case: it appears in multiple constructs,
disambiguated by context.

### Finding: the mapping is non-linear, and the repo is already inconsistent

The concept ↔ syntax mapping is not a bijection: one syntax element (`:`)
maps to several concepts — exactly what Semantic Purity forbids. The repo's
own decisions contradict each other:

- **EDR-083** rejected `0..10:step(2)` because `:` "conflicts with type
  annotations" — ranges never use `:`.
- **EDR-041** (2026-07-27) reserved `{key: value, ...}` as the map-literal
  candidate (deferred to Phase 5).
- The desugaring in `COLLECTION_LITERAL_SYNTAX.md` uses `->` for the pair:
  `{"a": 1, "b": 2}` → `Map("a" -> 1, "b" -> 2)`.

Two distinct problems follow:

1. **Semantic Purity violation** — `:` carries two meanings (type annotation
   and key-value), which the principle forbids with "no exceptions".
2. **DRY violation** — `:` in the literal duplicates `->`, which already
   means "association / pair construction". Two mechanisms, one concept.

### Secondary observations

- `TYPE_ANNOTATION_SYNTAX.md` still writes "dictionaries, if Orthon adopts a
  dictionary literal", although EDR-041 already accepted it — the document is
  stale on this point.
- The Semantic Purity table itself lists `*` as `pack/unpack` (two opposite
  roles), while POLA explicitly forbids "context-dependent role reversal".
  The principle is not self-consistent in its own text.

### Conclusion

"One symbol, one meaning" is achievable only as a deliberate choice of one
side, and the repo has not made that choice consistently. Direction chosen in
exploration: soften Semantic Purity from "one symbol = one meaning" to "one
symbol = one **semantic role**" (a role may appear in multiple grammatical
positions), and enumerate roles explicitly.

### Related

- [[semantic-purity-tier1-edr]] — draft the Tier 1 EDR.
- [[fix-colon-overload-stale-reference]] — fix the stale reference.
