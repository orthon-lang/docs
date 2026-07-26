# Compile-Time Metaprogramming

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-metaprogramming.md`

## Issue (Why)

Reflection and dynamic runtime type manipulation is slow, type-unsafe,
and hard to read. Legacy approaches breed `try/catch` blocks, violate
encapsulation, and defer errors from compile time to runtime.

Orthon should eliminate unsafe runtime reflection as a primary
metaprogramming mechanism by providing compile-time alternatives.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Direct attribute assignment `MyClass.new_attr = value` | `__slots__`, metaclasses, descriptors, Protocols |
| Java | `Class.forName()` + `Method.invoke()` with excessive `try/catch` | Annotations + annotation processors (compile-time) |

## Hypothesis

Orthon can eliminate unsafe runtime reflection as a primary
metaprogramming mechanism by providing compile-time alternatives:
macros or compile-time code generation for code that needs to inspect
or transform types, and a controlled, opt-in reflection API for
legitimate runtime use cases.

## Implications for Orthon

- Metaprogramming should be **compile-time** rather than runtime where
  possible.
- Reflection is a controlled capability, not universal access to
  encapsulation.
- Declarative alternatives (annotations / attributes) are preferred over
  reflection for code generation.
- If dynamic behaviour is needed — protocols / traits / macros instead of
  `getattr` / `invoke`.
- The compiler must check metaprogramming constructs at compile time.
- Related concepts: `essential/COMPILE_TIME_EXECUTION.md`, `METAOBJECTS.md`

## Open Questions

- Should Orthon have a macro system at v0.1 or defer to v0.2?
- Is compile-time execution Turing-complete? What are the limits?

## Decision History

Initial hypothesis derived from `imperative-crutch-metaprogramming.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/COMPILE_TIME_EXECUTION.md`
- [ ] `deferrable/METAOBJECTS.md`
- [ ] `what/GLOSSARY.md`
