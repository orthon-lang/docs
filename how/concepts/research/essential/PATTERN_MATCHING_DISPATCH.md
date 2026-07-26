# Pattern Matching Dispatch

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-type-casting.md`

## Issue (Why)

Manual type checking with `isinstance` / `instanceof` followed by
explicit casting duplicates effort, risks `ClassCastException`, and
violates polymorphism. The two-step pattern (check + cast) is both
verbose and error-prone.

Orthon should eliminate this pattern by providing built-in pattern
matching with type-based dispatch as a single expression.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `if isinstance(obj, MyClass): obj.method()` | Duck typing + `try/except` / Protocols |
| Java | `if (obj instanceof MyClass) { MyClass my = (MyClass) obj; my.method(); }` | `if (obj instanceof MyClass my) { my.method(); }` (Java 16+) |

## Hypothesis

Orthon can eliminate the two-step type-check-then-cast pattern by
providing built-in pattern matching with type-based dispatch, making
`isinstance`/`instanceof` followed by explicit casting obsolete.

## Implications for Orthon

- Pattern matching for type-based dispatch is a built-in language
  feature, not a chain of if-else.
- If type checking is needed, it must be a single expression (pattern
  matching), not two steps (check + cast).
- Dispatch through polymorphism / protocols / traits is preferred over
  type checking.
- Duck typing should be optionally statically checkable (structural
  typing).
- Related concepts: `PATTERN_MATCHING.md`, `TRAITS.md`, `SMART_CAST.md`

## Open Questions

- Should structural typing (duck typing with static checks) be an
  alternative to pattern matching?
- How does pattern matching interact with nullable types?

## Decision History

Initial hypothesis derived from `imperative-crutch-type-casting.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/PATTERN_MATCHING.md`
- [ ] `essential/TRAITS.md`
- [ ] `important/SMART_CAST.md`
- [ ] `what/GLOSSARY.md`
