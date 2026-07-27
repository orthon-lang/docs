# Declarative Multi-Key Sort

## Issue (Why)

Sorting by multiple fields requires manually writing comparators with comparison chains — code becomes verbose and fragile. Orthon should eliminate manual comparator chains by providing built-in lexicographic comparison for tuples and records, and a declarative `.sorted(by: ...)` API with chaining.

## Principles

1. **Lexicographic comparison** — Tuples and structs as natural sort keys; lexicographic comparison is built into the type.
2. **Declarative comparators** — The standard library provides declarative comparators with ascending/descending modifiers.
3. **Sort stability** — Guaranteed by default (see SORTING).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Sort API Policy | Determines the API shape for multi-key sorting |

## Model (What)

Multi-key sort is syntactic sugar over the sorting concept:

```orthon
# Declarative multi-key sort
let sorted = users.sorted(by: [.last_name, .first_name, .age])
let by_priority = tasks.sorted(by: .priority, order: descending)
```

This is syntactic sugar over a comparator constructed from key paths. No new language semantics.

## Default Strategy

Declarative multi-key sort is a **StdLib** API. It is syntactic sugar over the SORTING concept — the `sorted(by:)` method accepts a list of key paths and constructs a lexicographic comparator. No new language semantics.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Manual comparator chains | Write explicit comparison logic — verbose but explicit. |
| Builder pattern | `list.sort_by(.a).then_by(.b, desc: true)` — more structured but equivalent. |
| Anonymous comparator types | Dedicated comparator type with `then` combinator. |

## Open Questions

1. Should reverse/descending be a function or a wrapper type?
2. How to handle partial ordering (NaN, None)?

## Decision History

- **EDR-067:** Declarative Multi-Key Sort accepted as StdLib — syntactic sugar over SORTING. The `sorted(by:)` API constructs a lexicographic comparator from key paths; no new language semantics beyond what SORTING already provides.
- **Classification per D-03:** StdLib. Multi-key sort is a library API over the sorting primitive. Purely syntactic sugar.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
