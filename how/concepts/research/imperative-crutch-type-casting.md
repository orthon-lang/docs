# Imperative Crutch: Type Checking and Casting

## Problem

Manual type checking with `isinstance` / `instanceof` followed by explicit
casting duplicates effort, risks `ClassCastException`, and violates
polymorphism.

## Examples

### Type check before method call

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `if isinstance(obj, MyClass): obj.method()` | Duck typing: `obj.method()` with `try/except AttributeError`, or Protocols |
| Java | `if (obj instanceof MyClass) { MyClass my = (MyClass) obj; my.method(); }` | `if (obj instanceof MyClass my) { my.method(); }` (pattern matching, Java 16+) |

## Implications for Orthon

- Pattern matching for type-based dispatch is a built-in language feature,
  not a chain of if-else.
- If type checking is needed, it must be a single expression (pattern
  matching), not two steps (check + cast).
- Priority: dispatch through polymorphism / protocols / traits, not through
  type checking.
- Duck typing is useful but should be optionally statically checkable
  (structural typing).

**See also:** concepts `PATTERN_MATCHING.md`, `GENERICS.md`, `METAOBJECTS.md`
