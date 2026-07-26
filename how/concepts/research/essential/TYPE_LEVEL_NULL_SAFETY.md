# Type-Level Null Safety

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-null-handling.md`

## Issue (Why)

Cascading manual null checks clutter code, duplicate logic, and miss NPEs
when one check is forgotten. This is Hoare's "billion-dollar mistake" —
null references are the most common runtime failure across languages
without built-in null safety.

Orthon should eliminate manual null-check cascades by making `T?`
(nullable) vs `T` (non-nullable) a type-system distinction.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Nested `if obj and obj.attr and obj.attr.sub:` | `getattr(obj, 'attr', None)` |
| Java | Cascade `if (obj != null && obj.getAttr() != null && ...)` | `Optional.ofNullable(obj).map(Obj::getAttr).orElse(null)` |
| Python | `value = obj.attr if obj and obj.attr else default` | `getattr(obj, 'attr', default)` |
| Java | Ternary with triple check | `Optional.ofNullable(obj).map(Obj::getAttr).orElse(default)` |

## Hypothesis

Orthon can eliminate manual null-check cascades by making `T?` vs `T`
a type-system distinction, with safe navigation (`?.`) and default-value
(`??`) operators as mandatory syntax, and by making access to a nullable
value without dereferencing a compile error.

## Implications for Orthon

- Build an `Optional`-like monad at the type level, not the library level.
- Nullable types must be explicit in the type system (`T?`), and accessing
  a nullable field without dereferencing must be a compile error.
- Safe navigation operator (`?.`) is a mandatory syntax element.
- Elvis operator (`??`) for default values is mandatory.
- Related concepts: `NULL_SAFETY.md`, `ERROR_HANDLING.md`

## Open Questions

- Should `?` propagate through function return types automatically?
- How to handle legacy interop where nullability is unknown?

## Decision History

Initial hypothesis derived from `imperative-crutch-null-handling.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/NULL_SAFETY.md`
- [ ] `essential/ERROR_HANDLING.md`
- [ ] `what/GLOSSARY.md`
