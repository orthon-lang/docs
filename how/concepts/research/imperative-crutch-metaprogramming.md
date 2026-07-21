# Imperative Crutch: Metaprogramming and Reflection

## Problem

Reflection / dynamic runtime type manipulation is slow, type-unsafe, and
hard to read. Legacy approaches breed `try/catch` blocks and violate
encapsulation.

## Examples

### Dynamic attribute addition / method invocation

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Direct assignment `MyClass.new_attr = value` (messy) | `__slots__`, metaclasses, descriptors, Protocols |
| Java | `Class.forName()` + `Method.invoke()` with excessive `try/catch` | Annotations + annotation processors (compile-time), `MethodHandles` |

## Implications for Orthon

- Metaprogramming should be **compile-time** rather than runtime where
  possible.
- Reflection is a controlled capability, not universal access to
  encapsulation.
- Declarative alternatives (annotations / attributes) are preferred over
  reflection for code generation.
- If dynamic behavior is needed — protocols / traits / macros instead of
  `getattr` / `invoke`.
- The compiler must check metaprogramming constructs at compile time,
  minimizing runtime errors.

**See also:** concepts `METAOBJECTS.md`, `GENERICS.md`
