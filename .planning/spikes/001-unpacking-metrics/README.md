---
spike: 001
idea: semantic-simplicity-metrics
name: unpacking-metrics
type: standard
validates: "Given the accepted UNPACKING concept, when its constructs are derived over the 10 primitives and inventoried by form, then K, G, F, R, D, C can be recorded and tell sugar from core."
verdict: PARTIAL
related: [002, 003, 004]
tags: [simplicity, metrics, unpacking, derivation]
---

# Spike 001: UNPACKING metrics

Source: `what/concepts/UNPACKING.md` (EDR-055, syntax redesigned by EDR-096). Metric definitions: see Spike 004.

## What This Validates

Given UNPACKING, when each construct is derived over the 10 primitives of `what/PRIMITIVE_BLOCKS.md` and every grammar production is inventoried, then the per-concept metrics can be filled in without ambiguity and show that UNPACKING is derivable sugar.

## Investigation Trail

1. First pass treated UNPACKING as one idea ("destructuring"). It does not fit: `Model (What)` bundles at least eight separable ideas. The unit of measurement must be the **idea**, not the concept file.
2. Derived each idea over the kernel using the concept's own `Desugaring` section (tuple, rest, constructor pattern) and, where it has none, by analogy (labels, tuple spelling).
3. Checked the two "rules" (irrefutability, declare-versus-reassign complement). They have no runtime derivation at all: they are static checks, so they are residue, not kernel.
4. Cross-checked dependencies two ways (mined links versus curated reading). They disagree strongly (see C).

## Ideas, forms and derivations

| # | Idea | Grammar production(s) | Derivation over the kernel | Kernel elements |
|---|------|----------------------|----------------------------|-----------------|
| 1 | Positional destructuring | comma pattern list `x, y = p` | `tmp = p; x, y = unpack(tmp)` | assignment x2, identifier, unpack |
| 2 | Rest collect / spread | prefix `*x` (receiving collects, sending spreads: one production, two roles) | `first = tmp[1]; rest = pack(rest of tmp)`; spread = `unpack` then `pack` | assignment x2, identifier, call (`@get`), pack |
| 3 | Ignore | `_` (and `*_` = star applied to `_`) | no binding is created | none |
| 4 | Labels | `value as label` | label is metadata on a `pack` element; call-site override binds by parameter name | pack, identifier, call |
| 5 | Constructor-pattern destructuring | `T(p1 as n1, ...)`, mirrors construction | `n = p.name; a = p.age` | assignment, attribute access, identifier |
| 6 | Tuple spelling (1-tuple, empty tuple) | `tuple(a)`, `()`; `(a, b)` is sugar for `tuple(a, b)` | constructor call building a `pack` | call, pack |
| 7 | Irrefutability rule | none (static check) | not derivable: a totality check on the pattern | - |
| 8 | Declare-versus-reassign complement | none (static check) | not derivable: a name-resolution check ("all new or all existing") | - |

Parenthesised nested patterns `(a, b), c` and the whole-pattern `var` prefix are productions that serve ideas 1-5; they add no idea of their own.

## Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Ideas | 8 | 6 derivable, 2 static checks |
| P, grammar productions introduced | 7 | comma pattern list, `_`, `*x`, `as`, constructor pattern, nested parentheses, pattern `var` |
| Fdup, ideas with more than one production | 0 | `tuple(a, b)` / `(a, b)` is a named-library alias, allowed |
| K, kernel elements used | 7 of 10 | pack/unpack, assignment, identifier, attribute access, call, function, literal |
| G, kernel gaps (candidate primitives) | 0 | |
| R, residue | 4 | irrefutability check, complement check; plus two undecided: remainder representation for List/Array (drafting question h), move versus borrow (DEF-b) |
| D, derivation weight | 19 | idea 1: 4, idea 2: 5 (+2 for spread), idea 4: 3, idea 5: 3, idea 6: 2 (kernel applications in one template per idea) |
| C, curated depends_on / depended_on_by | 8 / 3 | out: PATTERN_MATCHING, NAMED_AND_OPTIONAL_PARAMETERS, DECLARATION_BY_ASSIGNMENT, COPY_ON_WRITE, NULL_SAFETY, SMART_CAST, INDEXING, GENERICS; in: ITERATION_LOOP, PATTERN_MATCHING, NAMED_AND_OPTIONAL_PARAMETERS |
| C, mined from markdown links | 3 / 2 | misses DECLARATION_BY_ASSIGNMENT, NULL_SAFETY, SMART_CAST, INDEXING, GENERICS (prose references without links) |
| Instability out/(in+out), curated | 0.73 | unstable: depends on much, little depends on it |

## Results

Verdict: **PARTIAL**. The metrics are fillable, and they say what intuition says (pure sugar over the kernel, G = 0, unstable), but two things the schema must handle came out:

1. **A concept is a bundle of ideas with different deferral costs.** Ideas 1, 3, 5, 6 are pure sugar. Idea 4 (labels) is used by call syntax in NAMED_AND_OPTIONAL_PARAMETERS (`value as name`), and idea 2 (star) has an open scope (DEF-a: variadic functions and spreading in calls may or may not be in v0.1). "Unpacking is convenient but we can do without it" is true of ideas 1, 3, 5 and false of 4. A grade belongs to an idea (or a declared slice of a concept), not to the file.
2. **UNPACKING sits in a six-member dependency cycle** (mined links): NAMED_AND_OPTIONAL_PARAMETERS, PATTERN_MATCHING, PATTERN_MATCHING_DISPATCH, STRUCTURAL_TYPING, TRAITS, UNPACKING. If those links are semantic, the six cannot be graded independently; if they are "see also" links, the cycle dissolves. The schema therefore needs edge strength (`normative` versus `see_also`).

Surprise worth a metric of its own: `as` is used for labels here and for generic bounds `<Iterator as T>` (EDR-086). F (forms per idea) does not catch it; a **symbols** field (meanings per symbol) does.

The concept is also the largest by normative text (1,545 words) while having G = 0 and the largest D: size and derivation weight track *how much sugar*, not how much *kernel*.
