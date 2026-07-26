# Error Union

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

Should a fallible function's error type be an ordinary, separately-declared generic
parameter (`Result<T, E>`), or a distinct built-in type former — an **Error Union** —
that fuses a success type and an inferred, structurally-open set of possible error
tags into one type written as a single prefix, such as Zig's `!T` (short for
`ErrorSet!T`, most commonly written as the inferred `!T`)?

Zig's Error Union is not sugar for a generic `Result<T, E>`: it is a distinct kind of
type. An error set (`error{FileNotFound, AccessDenied}`) is a closed set of unit-like
error *tags* — no payload, no associated data, just a name — and any concrete error
set structurally coerces into any error set that is a superset of it, without an
explicit conversion call. A function's error set is usually not declared by hand at
all: writing `!T` as the return type asks the compiler to *infer* the full error set
from every fallible call inside the function body, so the signature grows and shrinks
automatically as the implementation changes, and merges automatically across nested
calls. `try expr` is sugar for "evaluate `expr`; if it is an error, return that error
from the enclosing function immediately"; `catch` supplies a fallback expression or
block for the error case. There is exactly one universal escape hatch for "any error
at all," `anyerror`, used when a fully precise set is impractical (e.g. a plugin
boundary).

`ERROR_HANDLING.md` already proposes a Default Strategy for this repository's error
handling concept: a monadic `Result<T, E>` type, an explicit `?` propagation operator,
and combinators (`map`, `and_then`, `or_else`). That document's `E` is an ordinary,
user-supplied generic type parameter — it can be any type, including a payload-bearing
enum or struct, and it is never inferred; the caller (or the language) must name a
concrete `E` for every fallible function, and set-widening across call sites is not
part of its model. This document's scope is narrower and distinct: it researches
whether Orthon's error type should instead (or additionally) be a purpose-built Error
Union type former — one that infers a minimal error tag set from usage, that supports
implicit substructural coercion between error sets without a combinator call, and
that models "the error side of a fallible result" as tags-only rather than as an
arbitrary payload-bearing type. Whichever model wins at Concept Design Review, the two
proposals should be compared side by side rather than either silently subsuming the
other, since the trade-offs (signature-inferred vs. explicitly-declared error types;
tag-only vs. payload-bearing errors) cut against different parts of Orthon's Explicit
Semantics and LLM Generability pillars.

The core problem statement: should Orthon's fallible-function error type be an
ordinary generic parameter as `ERROR_HANDLING.md` currently proposes, or a distinct
Error Union type former with inferred, structurally-widening error sets, and if the
latter, does it coexist with `Result<T, E>` for payload-bearing errors or replace it?

## Examples

| Language | Error type shape | Error type declared or inferred | Payload on error | Widening across call sites | Propagation sugar |
|---|---|---|---|---|---|
| Zig | Error Union `ErrorSet!T` (commonly written `!T`) | Usually inferred from the function body; can be declared explicitly | No — error values are unit tags only (no fields) | Implicit — a narrower error set coerces into any superset automatically | `try expr` (propagate), `catch` (fallback) |
| Rust | `Result<T, E>` | Always declared; `E` is an ordinary type parameter | Yes — `E` is commonly an enum carrying data, or `Box<dyn Error>` | None automatic — requires `From`/`?`-driven conversion impls | `?` operator (propagate via `From::from`) |
| Swift | `throws` + `Error` protocol | Declared via `throws`; the concrete error type is usually erased to the `Error` protocol | Yes — conforming types can carry associated data | Implicit — any `Error`-conforming type is accepted, effectively type-erased | `try`/`try?`/`try!`, `catch` |
| Haskell | `Either e a` | Always declared; `e` is an ordinary type parameter | Yes — `e` is any type, typically an ADT | None automatic — requires explicit `mapLeft`/newtype wrapping | monadic `do`-notation short-circuiting via `Monad`/`MonadError` |
| Go | `(T, error)` tuple return | Declared per-function by convention, not enforced by the type system | Yes — `error` is an interface; concrete types carry arbitrary fields | None automatic — manual `errors.Is`/`errors.As`/wrapping | none — manual `if err != nil { return err }` at every call site |

Zig sample — inferred error set, no explicit declaration:

```
fn readConfig(path: []const u8) !Config {
    const data = try fs.readFile(path);   // error set inferred from fs.readFile
    return parseConfig(data);             // parseConfig's error set merges in too
}

const cfg = readConfig("app.toml") catch |err| switch (err) {
    error.FileNotFound => default_config(),
    else => return err,
};
```

Contrasting Rust sample — the same shape, but `E` is declared and payload-bearing,
and widening across two different error types requires an explicit `From` impl:

```rust
fn read_config(path: &str) -> Result<Config, ConfigError> {
    let data = fs::read_file(path)?;   // requires impl From<IoError> for ConfigError
    parse_config(data)
}
```

The Rust sample requires `ConfigError` to declare a conversion from `fs::read_file`'s
`IoError`; the Zig sample requires nothing — the inferred error set is simply the
union of every callee's error set, discovered by the compiler, not authored by the
programmer.

## Implications for Orthon

1. Inferred, structurally-widening error sets directly serve the Manifesto's
   "Explicit Semantics" and "Semantics over syntax" principles differently than
   `ERROR_HANDLING.md`'s Default Strategy does: `Result<T, E>` makes the error type
   maximally *explicit* (always named, never inferred), while an Error Union makes it
   maximally *low-friction* (never separately declared, automatically correct as the
   implementation changes) — these are opposite trade-offs on the same explicitness
   axis, not compatible refinements of one idea, so Concept Design Review must pick
   one as the Default Strategy rather than average them.
2. Signature inference cuts against the LLM Generability pillar's "Small surface,
   consistent rules" framing in `why/VISION.md`: an LLM generating a call to a
   function whose error set is inferred cannot determine, from the signature alone,
   which errors are possible without either reading the full call graph or querying
   compiler-derived tooling — this is the same discoverability trade-off
   `COMPILE_TIME_EXECUTION.md` already raises for duck-typed `comptime` generics, and
   the two documents' Open Questions should eventually be reconciled together, since
   both concern "the compiler infers a contract; the source text does not declare it."
3. Tag-only, payload-free error values are a strictly narrower model than
   `ERROR_HANDLING.md`'s payload-bearing `E`; if Orthon needs both (cheap, structural
   tags for common cases and rich payload-bearing errors for cases needing detail,
   e.g. a parse error with a line/column), the two proposals may need to coexist as
   two distinct representations rather than one subsuming the other — analogous to
   how `DATA_MODEL.md` already keeps multiple representations (Value, Tuple,
   Reference, …) rather than forcing every value through one universal shape.
4. Implicit widening (a narrower error set silently coercing into a wider one) is the
   one place this concept's model relaxes explicitness in exchange for ergonomics;
   this should be weighed against `EXTENSION_FUNCTIONS.md`'s and `GENERICS.md`'s
   general stance that Orthon prefers explicit conversions — if adopted, the widening
   rule needs a precise, statically-checkable definition (structural subset
   comparison of tag sets) so it remains a closed, decidable compile-time check and
   not an open-ended coercion mechanism.
5. `try`/`catch` as dedicated keyword-level propagation sugar is a second syntax for
   the same need `ERROR_HANDLING.md`'s `?` operator already serves; per the
   Manifesto's "One concept — one syntax," Orthon should not carry both an
   Error-Union-flavoured `try`/`catch` and a `Result`-flavoured `?` operator side by
   side unless they are propagating genuinely different underlying types — this is a
   design question Concept Design Review must resolve, not something this document
   can resolve unilaterally.

## Open Questions

1. Should Orthon's fallible-function error type be `ERROR_HANDLING.md`'s
   `Result<T, E>` (declared, payload-bearing), an Error Union (inferred,
   tag-only), or both as distinct representations for distinct needs?
2. If an Error Union is adopted, should its error set always be inferred (as in
   Zig), always be explicitly declared, or optionally either — and if optional, does
   that violate "one concept, one syntax" by giving the same feature two authoring
   modes?
3. Should implicit error-set widening across function boundaries be unconditional
   (any subset coerces into any superset, as in Zig), or should it require an
   explicit marker at the call site to preserve local readability of "which errors
   can this call actually produce"?
4. How would LLM-facing tooling (the Schema Provider, `LLM_NATIVE_TOOLCHAIN.md`)
   surface an inferred error set to an LLM generating a call site, given that the
   set is not present in the written signature?
5. Does `anyerror` (or an equivalent universal error supertype) belong in Orthon at
   all, given that it reintroduces exactly the "any error, unknown shape" escape
   hatch that typed, closed error handling is meant to eliminate?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [ERROR_HANDLING.md](ERROR_HANDLING.md)
- [COMPILE_TIME_EXECUTION.md](COMPILE_TIME_EXECUTION.md)
- [DATA_MODEL.md](DATA_MODEL.md)
- [GENERICS.md](GENERICS.md)
