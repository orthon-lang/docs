# Parse, Don't Validate — Orthon Idiom

> Exploratory note: a documented idiom for Orthon, not a language
> feature. Proposed as an Orthon Best Practice.

## The Problem: Boolean Blindness

A validation function returns `Bool`, erasing the fact of validity as
soon as the check ends:

```orthon
fun isValidEmail(s: String) -> Bool
    return s.contains("@") and s.contains(".")
```

```orthon
if isValidEmail(input):
    send(input)      # The compiler does not know input is a valid email.
                     # The witness of validity was erased at the closing
                     # parenthesis of isValidEmail().
```

The program *knows* `input` is valid; the type system does not. This is
**Boolean Blindness** — the "evidence" of validity is ephemeral and
cannot be used as a guarantee.

The fix is the idiom **Parse, don't validate** (Alexis King,
*[Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)*):
instead of checking a value and returning `Bool`, *parse* it into a more
precise type whose constructors guarantee validity. The type itself
becomes the witness.

## The Idiom

```orthon
# ❌ Validate: returns Bool, witness erased
fun isValidEmail(s: String) -> Bool
    return s.contains("@") and s.contains(".")

# ✅ Parse: returns Option<Email>, witness is the type
fun parseEmail(s: String) -> Option<Email>
    if looksLikeEmail(s):
        return Some(Email(s))
    else:
        return None
```

At the call site, the difference is decisive:

```orthon
# Boolean Blindness — compiler cannot help
if isValidEmail(input):
    send(input)          # input is still String; any use is unchecked

# Parse — the compiler enforces handling of both outcomes
match parseEmail(input):
    case Some(email) => send(email)   # email is Email — valid by construction
    case None        => reject("invalid email")
```

## Mechanisms That Realize It in Orthon

| Technique | Example | Guarantee |
|-----------|---------|-----------|
| **`match` / `case` with ADT** | `match parseEmail(s): case Some(e) => ... case None => ...` | Compiler verifies both cases handled |
| **`Option<T>` as parse result** | `fun parseEmail(s) -> Option<Email>` | Validity sealed in type `Email` |
| **Smart constructor + private fields** | `Email` with checked `parseEmail` only path | Cannot construct `Email` bypassing validation |
| **Literal types for closed sets** | `status: "open" \| "closed"` | Invalid values unrepresentable |
| **Constrained types (EDR-080)** | `type Email = String requires matches(v, pattern)` | Constraint declared once on the type, enforced at every entry boundary |
| **`Result<T, E>` for fallible parsing** | `fun parse(s) -> Result<Email, ParseError>` | Failure carries a payload |
| **`?` operator** | `let email = parseEmail(s)?` | Short-circuit propagation of `Option`/`Result` |
| **Exhaustive `when`** | `when status: case "open" => ... case "closed" => ...` | Compiler checks coverage of the literal type |

## Anti-Pattern Detection (proposed lint rule)

The compiler cannot reliably detect *intent* behind a `Bool` return —
`fun isAdult(age: Int) -> Bool` used in a `filter` is legitimate, not
Boolean Blindness. But a heuristic lint rule can flag the most common
smell:

- **Rule:** A function named `isValid*` / `check*` / `is*` returning
  `Bool`, whose result is consumed only inside a condition and never
  bound to a typed value, triggers a warning suggesting a `parse*`
  function returning `Option<T>` / `Result<T, E>`.

This is a *best-practice lint*, not a language error: the code is valid
Orthon; the rule steers toward the stronger form.

## Why This Matters for an LLM-Native Language

An LLM sampling tokens will eventually produce invalid states. If the
LLM's code uses the Parse idiom, invalid input fails *at the boundary*
(parse) and the resulting type is valid by construction — the LLM
cannot accidentally use an unvalidated value where a validated one is
required. If it uses Validate, the compiler has no hook to catch the
error. Parse, don't validate converts the LLM's nondeterminism into
compile-time-checkable structure.

## See also

- [`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md) — the underlying type-level pattern
- [`CONSTRAINED_TYPES.md`](../what/concepts/CONSTRAINED_TYPES.md) — EDR-080, the type-level `requires` form of this idiom
- [`REFINEMENT_TYPES.md`](../how/concepts/research/deferrable/REFINEMENT_TYPES.md) — the value-range form of the same idea
- [`CORRECTNESS_BY_CONSTRUCTION.md`](../how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md) — why structural guarantees beat runtime checks
- [`ALGEBRAIC_DATA_TYPES.md`](../what/concepts/ALGEBRAIC_DATA_TYPES.md) — the ADT mechanism behind `Option`/`Result`
