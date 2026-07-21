# Imperative Crutch: Null Handling

## Problem

Cascading manual null checks clutter code, duplicate logic, and miss NPEs
when one check is forgotten.

## Examples

### 1. Safe nested field access

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Nested `if obj and obj.attr and obj.attr.sub:` | `getattr(obj, 'attr', None)`, `try/except AttributeError` |
| Java | Cascade `if (obj != null && obj.getAttr() != null && ...)` | `Optional.ofNullable(obj).map(Obj::getAttr).map(Attr::getSub).orElse(null)` |

### 2. Default value

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `value = obj.attr if obj and obj.attr else default` | `value = getattr(obj, 'attr', default)` |
| Java | Ternary with triple check | `Optional.ofNullable(obj).map(Obj::getAttr).orElse(default)` |

## Implications for Orthon

- Build an `Optional`-like monad at the type level, not the library level.
- Nullable types must be explicit in the type system (`T?`), and accessing
  a nullable field without dereferencing must be a compile error.
- Safe navigation operator (`?.`) is a mandatory syntax element.
- Elvis operator (`??`) for default values is mandatory.
- Avoid Hoare's "billion-dollar mistake" — make values non-nullable by default.

**See also:** concept `ERROR_HANDLING.md`
