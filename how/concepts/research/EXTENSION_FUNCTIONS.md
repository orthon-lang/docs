# Extension Functions

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

How does a programmer add behaviour to an existing type without modifying its source, extending it via inheritance, or wrapping it in an adapter?

Classic solutions each have drawbacks:

1. **Inheritance** — Subclass the type and add methods. Requires the type to be open for extension (Java: not `final`), couples the new behaviour to the type hierarchy, and cannot be applied to types the programmer does not control (stdlib types, third-party libraries).
2. **Wrapper/Adapter** — Create a new class that holds an instance and exposes the same interface plus new methods. Requires boilerplate forwarding for every existing method, and callers must use the wrapper type instead of the original.
3. **Utility functions** — Static methods that take the target type as the first parameter (Java's `Collections.sort(list)`). Works, but call-site syntax is less natural — `sort(list)` instead of `list.sort()`. No IDE autocompletion for "methods on type T" because there is no syntactic connection.

Kotlin introduced **extension functions** — a function that is defined outside a type but called with method-call syntax on values of that type: `list.sort()` where `sort` is defined elsewhere. The function body receives the receiver as `this`.

The core problem: **should Orthon allow functions to be defined separately from their receiver type while using method-call syntax, and if so, how does this interact with name resolution, visibility, and the type system?**

## Examples

| Language | Extension syntax | Callsite | Access to private members |
|---|---|---|---|
| **Kotlin** | `fun String.isEmail(): Boolean` | `str.isEmail()` | No — only public members |
| **C#** | `static bool IsEmail(this string str)` | `str.IsEmail()` | No |
| **Swift** | `extension String { func isEmail() -> Bool }` | `str.isEmail()` | No |
| **Rust** | `trait IsEmail { fn is_email(&self) -> bool; }` + `impl` | `str.is_email()` | No (trait impl) |
| **Go** | Cannot extend external types | N/A | N/A |
| **Python** | Monkey-patching | `str.is_email = ...` | Yes — no access control |

## Implications for Orthon

1. **Extension functions should be a language construct** — A function defined outside its receiver type but called with receiver syntax:

   ```
   // Definition (anywhere in the package)
   fun String.isEmail() -> Bool:
       contains("@")

   // Callsite (same syntax as method)
   email = user.email_address.isEmail()
   ```

2. **No access to private members** — Extension functions operate through the public API of the extended type only. They are syntactic sugar for a top-level function with the receiver as first parameter.

3. **Static dispatch** — Extension functions are resolved at compile time based on the **static type** of the receiver, not the runtime type. This avoids dynamic dispatch complications (no vtable modification, no monkey-patching).

4. **Import control** — Extension functions from other packages must be explicitly imported. An imported extension function is available within the importing scope.

5. **Member precedence** — If a type defines a member function with the same signature as an extension function, the member wins. The compiler issues a warning if an extension shadows a member.

6. **Extension properties** — The same mechanism should work for properties:

   ```
   fun String.isEmail: Bool
       get: contains("@")
   ```

   Note: extension properties cannot have backing fields — they must be computed.

## Open Questions

1. Can extensions be applied to types across module boundaries? Can package A extend types from package B?
2. How does resolution work if two imported packages define the same extension function for the same type?
3. Should extension functions participate in trait/interface satisfaction? Can an extension function satisfy a trait method requirement?
4. Can extensions be conditionally applied (e.g., only when a generic type parameter satisfies a bound)?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `FUNCTIONS.md`
- [ ] `TOP_LEVEL_DECLARATIONS.md`
- [ ] `COMPOSITION_OVER_INHERITANCE.md`
- [ ] `VISIBILITY_AND_ENCAPSULATION.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
