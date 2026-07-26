# Collection Literal Syntax

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-collection-init.md`

## Issue (Why)

Creating a collection and then manually adding each element is verbose,
hard to read, and encourages mutation where a literal would suffice.
Java's `Map.of()` 10-pair limit and the verbosity of `new ArrayList<>()`
+ repeated `.add()` calls exemplify the problem.

Orthon should eliminate the create-then-mutate pattern by providing
compact, readable collection literals as a basic syntax element.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `lst = []; lst.append(1); lst.append(2)` | `lst = [1, 2]` |
| Java | `list = new ArrayList<>(); list.add(1); list.add(2);` | `List.of(1, 2)` |
| Python | `d = {}; d['a'] = 1; d['b'] = 2` | `d = {'a': 1, 'b': 2}` |
| Java | `map = new HashMap<>(); map.put("a", 1); map.put("b", 2);` | `Map.of("a", 1, "b", 2)` |

## Hypothesis

Orthon can eliminate the create-then-mutate pattern by providing compact
collection literals for lists, maps, and sets as a basic syntax element —
with immutable-by-default semantics and explicit `mut` opt-in for mutable
variants.

## Implications for Orthon

- Collection literals (list, map, set) are a basic syntax element, not a
  library feature.
- Compact syntax for immutable collections by default.
- If a mutable collection is needed, it requires explicit notation (e.g.,
  `MutableList` or a `mut` qualifier).
- No arbitrary size limits — built-in support for collections of any size.
- Related concepts: `essential/DATA_MODEL.md`, `essential/MUTABILITY.md`

## Open Questions

- How to disambiguate map `{}` from block/scope `{}`?
- What syntax for set literals vs list literals?

## Decision History

Initial hypothesis derived from `imperative-crutch-collection-init.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/DATA_MODEL.md`
- [ ] `essential/MUTABILITY.md`
- [ ] `what/GLOSSARY.md`
