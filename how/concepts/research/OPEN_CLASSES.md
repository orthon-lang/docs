# Open Classes & Monkey Patching

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

Should an already-defined type's own method table be reopenable and mutated at
runtime, anywhere in the program, without declaring intent at the type's own
definition site?

Ruby permits reopening any class — including core/stdlib classes such as
`String` or `Integer` — to add or override methods at any point during
program execution. The change is visible globally and immediately to every
existing and future instance of that type, including instances already held
by code that has never seen the patch.

This is a distinct question from [`FINAL_BY_DEFAULT.md`](FINAL_BY_DEFAULT.md).
That document concerns whether a class can be *subclassed* by default at
*definition time* — creating a new type from an existing one. This concept
concerns *runtime redefinition of an already-defined type's own method
table* — no new type is created, there is no separate namespace, and the
change is global and immediate.

Monkey patching introduces several concrete consequences:

- **Global mutable behaviour** — a change made in one file alters behaviour
  everywhere the type is used, including inside third-party library internals
  that never anticipated the change.
- **Name-collision hazard** — two independently-loaded libraries that patch
  the same method on the same type produce a well-documented class of Ruby
  bugs ("monkey patch collision").
- **Loss of static verifiability** — no IDE or compiler can determine a
  type's complete method table at any point in the program.
- **Debuggability cost** — call-site behaviour no longer matches anything
  visible at the file containing the call.
- **Library-evolution hazard** — a library author can never assume their own
  public class definition is complete, since any downstream consumer may have
  patched it.

The core problem: should Orthon permit adding or overriding methods on an
already-defined type after its definition, and if so, through what mechanism
and with what visibility/safety boundary?

## Examples

| Language | Reopen/patch mechanism | Scope of the change | Compile-time visibility | Safety guard |
|---|---|---|---|---|
| **Ruby** | Reopen any class anywhere (`class String; ...; end`) | Global — every instance everywhere, including stdlib types | None | `refine` provides a scoped, opt-in alternative |
| **Python** | Assign directly to a class attribute post-hoc, or module-level patching | Global, though `unittest.mock.patch` provides scoped/temporary patching for tests | None | Convention-only ("don't monkeypatch stdlib") |
| **JavaScript** | Prototype extension (e.g. assigning a function onto `String.prototype`) | Global — same hazard class as Ruby | None | Community convention/linters (e.g. `no-extend-native`) |
| **C#** | Extension methods (`this` parameter modifier) | Files that import the extension's namespace via `using`; does not mutate the original type's own method table | Fully compile-time resolved, visible via `using` | Full, by construction |
| **Rust** | No reopening at all — the orphan rule forbids implementing a foreign trait for a foreign type in most cases | Disallowed by design | Compile-time enforced | Full, by construction |
| **Kotlin/Swift** | Extension functions/methods — same shape as C#, scoped by import/module visibility, does not mutate the original type | Scoped | Compile-time resolved | Full, by construction |

Ruby, reopening `String` to add a `shout` method:

```ruby
class String
  def shout
    upcase + "!"
  end
end

"hello".shout   # => "HELLO!"
```

C#, achieving the same call-site ergonomics without touching `System.String`'s
own method table:

```csharp
public static class StringExtensions
{
    public static string Shout(this string str) => str.ToUpper() + "!";
}

str.Shout();   // "HELLO!" — String's own method table is untouched
```

This C# pattern is the same mechanism already documented in
[`EXTENSION_FUNCTIONS.md`](EXTENSION_FUNCTIONS.md), whose Implication 3 states
extension functions are resolved by static receiver type with "no vtable
modification, no monkey-patching."

## Implications for Orthon

1. **Conflicts with Explicit Semantics** — Global mutable behaviour directly
   conflicts with Orthon's Explicit Semantics pillar and the Manifesto's
   "Semantics over syntax" principle — a method call's behaviour must be
   determinable from the call site and the receiver's declared type, not
   contingent on which files happened to load elsewhere in the program.

2. **Conflicts with LLM Generability** — It also conflicts with the LLM
   Generability pillar (`why/VISION.md` "Designed for the LLM Era" §
   "Small surface, consistent rules") — an LLM generating a call to a
   builtin or stdlib method cannot verify from local context whether that
   method has been silently redefined elsewhere, which is exactly the
   non-local, context-dependent rule the LLM-readiness pillar warns against.

3. **Extension Functions already cover the legitimate need** — Orthon's
   research already has a scoped, statically-visible answer to the
   legitimate underlying need ("add capability to a type I do not own") —
   [`EXTENSION_FUNCTIONS.md`](EXTENSION_FUNCTIONS.md)'s extension functions,
   resolved at compile time by static receiver type and imported explicitly
   per scope — and unrestricted open classes should not be introduced as a
   second, conflicting mechanism for the same need, per the Manifesto's "One
   concept — one syntax."

4. **Closed by default, applied to method tables** — Closed-by-default type
   definitions ([`FINAL_BY_DEFAULT.md`](FINAL_BY_DEFAULT.md)) address
   inheritance-time extensibility, while open classes address
   post-definition mutation of the same type — both should default closed,
   so that a type's method table, once defined, cannot be altered outside its
   own defining module; this is the direct analogue of "closed by default"
   applied to method-table mutation rather than subclassing.

5. **Any scoped exception must be narrow and distinct** — If a genuinely
   scoped, opt-in need exists (for example, stubbing a single dependency for
   one test file), it should be evaluated as a distinct, narrowly-scoped
   mechanism (analogous to Ruby's `refine` or Python's
   `unittest.mock.patch`) rather than unrestricted global monkey patching,
   and even then weighed against whether Extension Functions plus ordinary
   dependency injection already cover the need.

## Open Questions

1. Should Orthon forbid modifying an already-defined type's method table
   entirely (Rust's orphan-rule-style prohibition), relying solely on
   Extension Functions for "add capability to a type I don't own"?
2. Is there a legitimate, narrowly-scoped use case (test mocking/stubbing, a
   monkey-patch limited to a single file) that needs a distinct,
   explicitly-marked mechanism, and if so, does it belong in the core
   language or in a testing library?
3. How does this interact with Standard Library evolution guarantees — could
   library authors ship "official" extensions to built-in types purely via
   Extension Functions, eliminating any need for a monkey-patch capability?
4. Should there be any lexically-scoped exception (Ruby's `refine`) limiting
   a patch's visibility to the file/module that requested it, or does even
   that violate "one concept, one syntax" by introducing a second resolution
   mode for method calls?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [FINAL_BY_DEFAULT.md](FINAL_BY_DEFAULT.md)
- [EXTENSION_FUNCTIONS.md](EXTENSION_FUNCTIONS.md)
- [COMPOSITION_OVER_INHERITANCE.md](COMPOSITION_OVER_INHERITANCE.md)
