# Properties

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language expose the data of a type without coupling consumers to its internal representation?

The classical OOP answer (Java, C++) treats fields as private implementation details exposed through explicit getter and setter methods. This has two consequences:

1. **Ceremony asymmetry** — A simple field (`name: String`) requires 2–6 lines of accessor boilerplate (`getName()`, `setName(...)`), which programmers often skip, resorting to `public` fields that break encapsulation.
2. **No uniform access** — Consumers cannot tell whether `.name` is a field access or a method call. Refactoring a field into a computed value changes the calling convention (add/remove `()`), breaking all call sites.

Later languages (C#, Kotlin, Swift) solved this with **properties** — a single syntactic construct that unifies field storage and computed access behind a uniform `.name` interface. The caller writes `obj.name` regardless of whether it is a stored field, a computed value, a lazy-evaluated expression, or a validated setter.

The core problem: **should Orthon have properties as a first-class language construct, or should it use explicit getter/setter methods with a convention-based uniform call syntax?**

## Examples

| Language | Mechanism | Uniform access | Call-site change on refactor |
|---|---|---|---|
| **Java** | `public` fields or explicit getters/setters | No — getters use `getX()` syntax | Yes — field → getter adds `()` |
| **C#** | `public string Name { get; set; }` | Yes — `obj.Name` regardless of storage | No — field and property share syntax |
| **Kotlin** | `var name: String` with implicit get/set | Yes — `obj.name` always | No — property syntax is uniform |
| **Python** | `@property` decorator | Yes — `obj.name` always | No — property replaces field transparently |
| **Rust** | No properties; fields + methods are separate | No — field access and method call differ | Yes |
| **Go** | Exported fields via capitalization; no computed fields | Partially — exported fields are accessed by name | Yes — field → method requires call-site change |

## Implications for Orthon

1. **Properties over raw fields** — Every `field` in a type declaration is implicitly a property with a getter and optional setter. The caller always uses `.name` syntax. Changing storage to computation never changes the call site.
2. **Getter/setter bodies optional** — A property declared as `name: String` auto-generates a trivial getter (return the backing field). For computed properties, the getter body is specified explicitly:

   ```
   struct Person:
       name: String                          // implicit getter, no setter
       var age: Int                           // implicit getter + setter
       is_adult: Bool                         // computed property
           get: self.age >= 18
   ```

3. **Visibility on getter vs. setter independently** — A property may have a public getter but a private setter (`public get, private set` in C# parlance).
4. **No `()` for reading** — Property access never requires parentheses. Method calls with side effects may still use `()`.
5. **Backing field access** — The getter/setter body accesses the underlying storage via an explicit `self.field` or `backing` reference, not by recursive property call.

## Open Questions

1. Should properties support **field-level attributes/annotations** (validation, serialisation) natively, or delegate this to `METAOBJECTS.md` mechanisms?
2. Should there be a **property shorthand** for delegation (see `DELEGATION.md`), e.g., `name: String by lazy { ... }`?
3. How do properties interact with traits/interfaces — can an interface declare a property without specifying whether it is stored or computed?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `DATA_MODEL.md`
- [ ] `VISIBILITY_AND_ENCAPSULATION.md`
- [ ] `DELEGATION.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
