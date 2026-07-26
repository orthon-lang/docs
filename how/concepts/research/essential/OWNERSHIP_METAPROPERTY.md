# Ownership Metaproperty `@ownership`

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
>
> This hypothesis explores the `@ownership` metaproperty as concrete syntax for
> ownership transfer. The concept itself (move semantics — transferring ownership
> from one binding to another, invalidating the source) is defined in
> `OWNERSHIP.md`. This document proposes `@ownership` (with optional short alias
> `@own`) as the canonical syntax for that concept.
>
> The `move` keyword appearing in other hypotheses (`DELEGATE.md`,
> `IDENTITY_BASED_SAFETY.md`, `REGION_BASED_MEMORY_MANAGEMENT.md`) refers to
> the same underlying concept. `@ownership` is one candidate syntax; both the
> `move` keyword and the `$` prefix operator are alternatives. All three express
> the same semantics.
>
> **Related:** `OWNERSHIP.md` (semantic model), `DELEGATE.md` (primary consumer
> of the move concept), `IDENTITY_BASED_SAFETY.md` (implicit move),
> `REGION_BASED_MEMORY_MANAGEMENT.md` (move as default).
> **Competing hypothesis:** [`$` prefix operator](OWNERSHIP_TRANSFER_OPERATOR.md).

---

## Issue (Why)

Ownership transfer from one variable to another (or into a container such as
a delegate) must be **explicit** in Orthon. Implicit transfer would cause
unexpected loss of access and undermine the memory-safety guarantees of a
GC-free language.

The `move` keyword is functional and unambiguous, but it does not align with
Orthon's pursuit of conciseness and expressiveness through lightweight
syntactic constructs. A syntax is desired that, while remaining explicit,
is intuitively clear and fits naturally into a potential system of
compile-time meta-attributes.

The `@` operator — a universal accessor for compile-time metadata — provides
a general mechanism. Ownership transfer becomes one application of that
mechanism rather than a special-purpose syntax. This keeps the language
surface smaller while making intent legible: `a@ownership` reads as
"data, then its property."

---

## Hypothesis

Introduce the concept of **metaproperties** — compile-time-only attributes
accessible via the `@` operator. Ownership transfer is expressed as access
to the `ownership` metaproperty of a variable:

```
let a = List()
let b = a@ownership       // b takes ownership, a is invalidated
let c = delegate(b@ownership)
```

A short alias `@own` may be provided for convenience:

```
let b = a@own              // equivalent to a@ownership
```

For **temporary / freshly-constructed values**, the metaproperty is not
required because they have no permanent owner:

```
let lst = delegate(List())   // List() is fresh — no @ownership needed
```

### Operator Semantics

`@` is a **postfix accessor** on expressions, binding tighter than `.`
(field/method access) but looser than call parentheses:

```
a@ownership.field     // interpreted as (a@ownership).field
                      // ownership is taken, then .field is accessed on the result
```

`@` can later be extended with other metaproperties such as:

- `a@type` — compile-time type information
- `a@size` — compile-time size of a value
- `a@align` — compile-time alignment

On the initial milestone, only `ownership` (and the optional alias `own`)
is required. Other metaproperties are deferred.

### Fresh-Value Exemption

Temporary values — literals, constructor calls, and expressions whose result
has not been bound to a variable — do not require `@ownership`:

```
let x = delegate(List())          // List() is fresh — no @ownership needed
let y = delegate([1, 2, 3])      // [1, 2, 3] is fresh — no @ownership needed
let items = List()
let z = delegate(items@ownership) // items is a variable — @ownership required
```

The compiler infers that a temporary has no other owner and implicitly
transfers it. This keeps the common case concise while preserving
explicitness for named bindings.

---

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Ownership Policy | Core — `@ownership` is the syntax for ownership transfer; the semantics are defined by `OWNERSHIP.md` |
| Lifetime Policy | Move invalidates the source binding; lifetime ends at the point of transfer |
| Allocation Policy | Move may require bytewise copy between arenas (see `REGION_BASED_MEMORY_MANAGEMENT.md`) |

---

## Design Principle

> **Ownership transfer is a compile-time property of a binding. `@ownership`
> reads as "give me the ownership of this value" — Data First, property
> second.**

The metaproperty approach applies uniformly to all ownership-transfer contexts:

- Variable-to-variable assignment
- Function arguments (including delegate construction)
- Return from `release` (ownership return from a delegate)
- Any future construct that consumes ownership

---

## Examples

### Basic transfer

```
let a = List()
a.add(1)
let b = a@ownership
// a.add(2)          // COMPILE ERROR: a was moved
b.add(2)              // OK
```

### Delegate construction

```
let lst = List()
lst.add(1)
let lst = delegate(lst@ownership)    // move into delegate, shadow with DelegatedContext
lst <- append(2)                     // delegated call via <-
```

### Release from delegate

```
let lst = release(lst@ownership)     // ownership returned, delegate destroyed
lst.add(3)                           // direct call via . restored
```

### Function argument

```
proc consume(data: List)
    // takes ownership of data

let source = List()
consume(source@ownership)
// source is invalid here
```

### Fresh temporaries (no `@ownership`)

```
let d1 = delegate(List())                  // OK — fresh
let d2 = delegate([1, 2, 3])              // OK — fresh
let items = List()
let d3 = delegate(items@ownership)         // @ownership required — items is a named binding
```

---

## Comparison with `move` Keyword and `$` Operator

| Aspect | `@ownership` (metaproperty) | `move` (keyword) | `$` (prefix operator) |
|--------|---------------------------|-------------------|----------------------|
| Visual weight | 11 characters (`@ownership`) / 4 (`@own`) | 4 characters + space | 1 character |
| Self-documenting | Yes — `ownership` is an English word | Yes — `move` is an English word | No — must be learned |
| Extensibility | `@` opens the door to `@type`, `@size`, etc. | Keyword per concept | Symbol per concept |
| Data First alignment | `a@ownership` — data, then property | `move a` — keyword, then data | `$a` — symbol, then data |
| Conflict with other languages | `@` used for annotations (Python), attributes (C#) | `move` is a keyword in Rust, C++11 | `$` used in string interpolation (JS, PHP, C#) |
| Core size impact | Introduces `@` operator into the language | Keyword addition | Operator addition |
| Discoverability | `@ownership` is grep-friendly | `move` is a common English word | `$` is hard to search for |

---

## Trade-offs

### Advantages

- **Self-documenting:** the word `ownership` directly explains intent
- **Extensible:** the `@` operator can become the standard way to access
  compile-time attributes, avoiding a new keyword for every concept
- **Data First:** `a@ownership` reads as "data, then its property," aligning
  with the Data First design principle
- **No symbol conflicts:** `@` does not conflict with widely-used symbols
  from other languages (`$` for string interpolation, `#` for macros)
- **Grep-friendly:** `@ownership` is easy to search for in code and logs

### Disadvantages

- **More verbose than `$`:** even the `@own` alias is 4 characters vs. 1
- **Core size:** introduces the `@` operator into the language, increasing
  the size of the minimal core
- **Metaproperty ambiguity:** if metaproperties proliferate, the distinction
  between them requires a strict specification to avoid confusion
- **Learnability:** the "metaproperty" concept may be less obvious to
  newcomers than a simple `move` keyword or `$` operator

---

## Related Concepts and Alternatives

- **`move` keyword** — baseline alternative; can coexist as a synonym for
  `@ownership` (`let b = move a` is equivalent to `let b = a@ownership`).
- **`$` prefix operator** — competing hypothesis (see
  `OWNERSHIP_TRANSFER_OPERATOR.md`); offers maximum conciseness at the cost
  of discoverability.
- **Postfix bare `@`** (`a@`) — not recommended due to ambiguity (ownership?
  type? reference?).
- **Built-in function `transfer(a)`** — breaks uniformity with delegates;
  looks clunky in idiomatic code.

---

## Open Questions

1. Should `@own` be a canonical short alias or deferred until metaproperty
   usage data is available?
2. What is the full list of planned metaproperties (`@type`, `@size`,
   `@align`, …)? Defining the set upfront prevents fragmentation.
3. How does `@ownership` interact with pattern matching and destructuring?
   Can you destructure a moved value?
4. Should `@ownership` be allowed in expression position only, or also in
   type position (e.g., `fn consume(x: T@ownership)`)?
5. What are the precedence rules when `@ownership` appears in complex
   expressions with method chains and arithmetic?

---

## Decision History

- **2026-07-26** — Initial hypothesis drafted as a competing alternative to
  `$` prefix operator (`OWNERSHIP_TRANSFER_OPERATOR.md`) and `move` keyword.
  The `@` metaproperty concept is filed as a research document; no EDR has
  been created yet. Formal Concept Design Review is deferred to Milestone 2.

---

### Affected Documents

- [ ] `OWNERSHIP.md` — the semantic model this syntax expresses
- [ ] `OWNERSHIP_TRANSFER_OPERATOR.md` — competing hypothesis
- [ ] `DELEGATE.md` — primary consumer of ownership transfer
- [ ] `DELEGATION.md` — delegation relies on ownership transfer
- [ ] `IDENTITY_BASED_SAFETY.md` — implicit move semantics
- [ ] `REGION_BASED_MEMORY_MANAGEMENT.md` — move as default in region-based allocation
- [ ] `GLOSSARY.md` — add "metaproperty" and "@ownership" entries
- [ ] `DESIGN_PRINCIPLES.md` — Data First principle alignment
