# Smart Cast

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

After checking that a value is of a specific type, the programmer should not need to cast it again — the compiler should know.

In languages without smart casts (Java, C++ pre-C++17), the following pattern is verbose and redundant:

```java
if (value instanceof String) {
    String s = (String) value;  // redundant — we just checked!
    System.out.println(s.length());
}
```

The type check proves `value` is a `String`, yet the programmer must cast it again. The compiler has lost the information that the check provided.

Smart casting solves this: **the compiler tracks type-narrowing information through control flow constructs** (`if`, `when`, `&&`, `||`, `return`, `throw`) and automatically narrows the type of a variable within the relevant scope.

The core problem: **to what extent should Orthon's compiler automatically narrow types based on control flow, and where should the programmer be required to insert explicit casts?**

## Examples

| Language | Smart cast | Works on | Limitations |
|---|---|---|---|
| **Kotlin** | Yes | `val` locals, `val` properties (if not open) | `var` requires manual cast; not across function calls |
| **TypeScript** | Yes (type narrowing) | Any variable within a control flow branch | Complex narrowing with unions can be subtle |
| **C#** | Pattern-based (`is` + `when`) | Match expression results | Explicit cast still needed in some contexts |
| **Rust** | Via `match` + pattern matching | Match arms | No implicit casting — exhaustive matching |
| **Java 16+** | Pattern matching for `instanceof` | Locals in scope of `if` | Limited to `instanceof`; no smart cast for null checks |
| **Go** | No | Manual type assertion | `v.(Type)` always required |

## Implications for Orthon

1. **Automatic type narrowing after type checks** — After `if value is String`, the compiler narrows the type of `value` to `String` within the true branch:

   ```
   if value is String:
       print(value.length)   // no cast needed — compiler knows it's String
   ```

2. **Null check narrowing** — After checking `value isnt None`, the compiler unwraps `Option[T]` to `T`:

   ```
   if user.email isnt None:
       send_email(user.email)   // compiler knows email is not None
   ```

3. **`when` branch narrowing** — Each branch of a `when` expression narrows the scrutinee's type for that branch (see `PATTERN_MATCHING.md`).

4. **Immutability prerequisite** — Smart cast only applies to effectively-immutable variables (`val` or non-reassigned `let`). If the variable could change between the check and the use, the cast is unsafe and the compiler requires an explicit cast.

5. **No cast in compound expressions** — In `value is String && value.length > 0`, the smart cast applies to the second operand because short-circuit evaluation guarantees the check passed.

6. **Explicit cast escape hatch** — `value as Type` performs an unchecked cast (assertion) when the programmer knows more than the compiler can prove.

## Open Questions

1. Should smart casts apply to mutable variables if the compiler can prove they are not modified between check and use (escape analysis)?
2. How does smart cast interact with `EXTENSION_FUNCTIONS.md` — can the compiler narrow to a specific interface to enable extension function calls?
3. Should smart cast be available in negated branches (`if value isnt String` → value is `NotString`)?
4. Should function return types participate in narrowing? E.g., if a function returns `T | None` and the caller checks the result, does the narrowed type propagate?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `PATTERN_MATCHING.md`
- [ ] `NULL_SAFETY.md`
- [ ] `ALGEBRAIC_DATA_TYPES.md`
- [ ] `TYPE_SYSTEM.md`
- [ ] `what/GLOSSARY.md`
