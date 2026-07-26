# Ownership Transfer Operator `$`

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
>
> This hypothesis explores concrete syntax for the ownership transfer concept.
> The concept itself (move semantics — transferring ownership from one binding
> to another, invalidating the source) is defined in `OWNERSHIP.md`. This
> document proposes `$` as the canonical syntax for that concept.
>
> The `move` keyword appearing in other hypotheses (`DELEGATE.md`,
> `IDENTITY_BASED_SAFETY.md`, `REGION_BASED_MEMORY_MANAGEMENT.md`) refers to
> the same underlying concept. `$` is one candidate syntax; the keyword form
> `move` is an alternative. Both express the same semantics.
>
> **Related:** `OWNERSHIP.md` (semantic model), `DELEGATE.md` (primary consumer
> of the move concept), `IDENTITY_BASED_SAFETY.md` (implicit move),
> `REGION_BASED_MEMORY_MANAGEMENT.md` (move as default).
> **Competing hypothesis:** [`@ownership` metaproperty](OWNERSHIP_METAPROPERTY.md).

---

## Issue (Why)

Ownership transfer from one variable to another (or into a container such as
a delegate) must be **explicit** in Orthon. Implicit transfer would cause
unexpected loss of access and undermine the memory-safety guarantees of a
GC-free language.

The current baseline syntax is the `move` keyword:

```
let b = move a
delegate(move lst)
release(move lst)
```

`move` is unambiguous and self-documenting, but it is verbose for an
operation that appears frequently in idiomatic code. In nested expressions
it becomes visually heavy:

```
delegate(move lst)
```

A lighter but equally explicit syntax is desirable — one that serves as a
unified marker of ownership transfer throughout the language.

---

## Hypothesis

Introduce a **prefix unary operator `$`** that extracts ownership from a
variable:

```
let a = List()
let b = $a                // b takes ownership, a is invalidated
let c = consume($b)       // transfer into a function
let shared = delegate($c) // transfer into a delegate
```

For **temporary / freshly-constructed values**, the operator is not required
because they have no long-term owner:

```
let nums = delegate([1, 2, 3])   // fresh — $ not needed
```

### Symbol Choice

`$` was chosen because it universally associates with *resource*, *property*,
and *value*. It is visually lightweight yet conspicuous — hard to miss in
code, easy to scan. Its meaning in Orthon is fixed: **"transfer ownership."**
There is no other use of `$` in the language (no string interpolation via
`$`, no `$`-prefixed variable names).

### Precedence

`$` binds tighter than `.` (field/method access):

```
$a.b        // interpreted as $(a.b) — take ownership of the result of a.b
            // NOT ($a).b — do not take ownership of a and then access .b
```

Rationale: `$` is applied to the *value being moved*, which is the result of
the entire expression to its right. `($a).b` would move `a` and then access
a field on the moved-from value — a use-after-move error in nearly all cases.

### Fresh-Value Exemption

Temporary values — literals, constructor calls, and expressions whose result
has not been bound to a variable — do not require `$`:

```
let x = delegate(List())        // List() is fresh — no $ needed
let y = delegate([1, 2, 3])    // [1, 2, 3] is fresh — no $ needed
let z = delegate($existing)     // existing is a variable — $ required
```

The compiler infers that a temporary has no other owner and implicitly
transfers it. This keeps the common case concise while preserving
explicitness for named bindings.

---

## Design Principle

> **Ownership transfer is syntactically explicit. `$` is the single marker
> for "this binding gives up ownership here."**

The operator applies uniformly to all ownership-transfer contexts:

- Variable-to-variable assignment
- Function arguments (including delegate construction)
- Return from `release` (ownership return from a delegate)
- Any future construct that consumes ownership

No contextual rules. `$` means move. Always.

---

## Comparison with `move` Keyword

| Aspect | `$` (prefix operator) | `move` (keyword) |
|--------|----------------------|-------------------|
| Visual weight | 1 character | 4 characters + space |
| Nested expression density | `delegate($lst)` | `delegate(move lst)` |
| Discoverability | Must be learned | Self-documenting English |
| Conflict with other languages | `$` used in string interpolation (JS, PHP, C#) | `move` is a keyword in Rust, C++11 |
| Mnemonic | "value, resource" — abstract | "transfer" — direct |
| Orthon string interpolation | Not via `$` (planned: `"Hello {name}"`) | — |

---

## Examples

### Basic transfer

```
let a = List()
a.add(1)
let b = $a
// a.add(2)          // COMPILE ERROR: a was moved
b.add(2)              // OK
```

### Delegate construction

```
let lst = List()
lst.add(1)
let lst = delegate($lst)    // move into delegate, shadow with DelegatedContext
lst <- append(2)            // delegated call via <-
```

### Release from delegate

```
let lst = release($lst)     // ownership returned, delegate destroyed
lst.add(3)                  // direct call via . restored
```

### Function argument

```
proc consume(data: List)
    // takes ownership of data

let source = List()
consume($source)
// source is invalid here
```

### Fresh temporaries (no `$`)

```
let d1 = delegate(List())         // OK — fresh
let d2 = delegate([1, 2, 3])     // OK — fresh
let items = List()
let d3 = delegate($items)         // $ required — items is a named binding
```

---

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Ownership Policy | Core — `$` is the syntax for ownership transfer; the semantics are defined by `OWNERSHIP.md` |
| Lifetime Policy | Move invalidates the source binding; lifetime ends at the point of transfer |
| Allocation Policy | Move may require bytewise copy between arenas (see `REGION_BASED_MEMORY_MANAGEMENT.md`) |

---

## Trade-offs

### Advantages

- **Maximum conciseness** — one character, no keyword.
- **Visually conspicuous** — stands out in code, hard to miss.
- **Orthogonal** — a single operator for all ownership-transfer contexts
  (assignment, function arguments, delegate construction, release).
- **No keyword namespace conflict** — leaves `move` available if needed for
  other purposes, or keeps it as a secondary long-form.
- **Fresh-value exemption** keeps the common case clean.

### Disadvantages

- **`$` is widely used in other languages** for string interpolation, variable
  prefix, or shell expansion. Newcomers may initially misread it. Acceptable
  if Orthon uses `{name}` for interpolation and reserves `$` exclusively
  for ownership transfer.
- **No direct English mnemonic** — the meaning must be memorized rather than
  inferred from the word.
- **Precedence must be learned** — `$a.b` = `$(a.b)` is not obvious to a
  reader unfamiliar with the rule.
- **Requires the fresh-value exemption rule** — otherwise `delegate(List())`
  would need `$`, defeating the conciseness goal.

---

## Related Concepts and Alternatives

- **`move` keyword** — the baseline alternative from which this hypothesis
  departs. May remain as a secondary/documentation form.
- **Postfix `a$`** — less convenient for parsing and reading; not recommended.
- **`@ownership` metaproperty** — a separate, competing hypothesis (see
  [`OWNERSHIP_METAPROPERTY.md`](OWNERSHIP_METAPROPERTY.md)). More verbose but
  more self-documenting. Uses the `@` operator as a universal metaproperty
  accessor.
- **Implicit move** (no syntax at all) — `y = x` transfers ownership silently.
  Rejected: violates Orthon's principle of explicitness. The loss of access
  to `x` must be syntactically visible.

---

## Open Questions

1. Should `$` be the **only** syntax for ownership transfer, or should
   `move` remain as an equivalent long-form (like `and`/`or` vs `&&`/`||`)?
2. Can the fresh-value exemption be formalized precisely? What exactly
   qualifies as "temporary" — only literals and constructors, or any
   expression whose result is not bound?
3. Does `$` interact correctly with the identity-based safety model
   (`IDENTITY_BASED_SAFETY.md`), where `.` methods may mutate `self`
   only if the compiler proves unique ownership?
4. Should `$` be overloadable for user-defined types (like Rust's `Deref`
   or C++ move constructors), or is it purely a compiler primitive?
5. How does `$` compose with pattern matching and destructuring? Does
   `let (a, b) = $(tuple)` move the entire tuple and then destructure?

---

## Decision History

| Date | Decision |
|------|----------|
| 2026-07-26 | Hypothesis created. `$` proposed as candidate syntax for the ownership transfer concept. `move` remains the concept name; `$` is one possible concrete syntax. `DELEGATE.md`, `OWNERSHIP.md`, and other files continue to use `move` as the concept term. |
