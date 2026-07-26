# Custom Operators

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

Should a language allow programmers to define their own symbolic operators, or should operators be limited to a fixed set built into the language?

Languages fall into two camps:

1. **Fixed operator set** (Java, Go, Python, Rust) — the language defines a fixed set of operators (`+`, `-`, `*`, `/`, `==`, etc.). Operators can be overloaded for user-defined types (Rust traits, Python `__add__`), but no new operator symbols can be created.

2. **Custom operators** (Swift, Haskell, Scala, C++) — programmers can define new operators, specify their precedence, associativity, and sometimes their fixity (infix, prefix, postfix).

The core tension:

- **Against custom operators**: New operators create a **readability tax**. A codebase using domain-specific operators (`|>`, `>>=`, `<|>`) is opaque to newcomers. Every operator is a new syntax element the reader must learn. This is the Java/Go philosophy: restrict syntax to minimise cognitive load.

- **For custom operators**: Domain-specific notations can dramatically improve **expressiveness** for specialised domains — mathematics (matrix multiplication `*+`), functional programming (composition `>>>`, pipe `|>`), DSLs. If the language is used for mathematical computation or symbolic manipulation, custom operators prevent verbose method chains.

Swift allows defining custom operators with a declaration of precedence and associativity:

```swift
// Swift — custom infix operator
infix operator +++: AdditionPrecedence
func +++<T: Numeric>(lhs: T, rhs: T) -> T { lhs + rhs }
```

Haskell goes further, allowing arbitrary symbolic operators:

```haskell
-- Haskell — custom operator
(|>>) :: a -> (a -> b) -> b
x |>> f = f x   -- pipeline operator (like F# |>)
```

## Examples

| Language | Custom operators | Precedence control | New symbols allowed? | Overload existing? |
|---|---|---|---|---|
| **Java** | No | N/A | No | No (built-in only) |
| **Go** | No | N/A | No | No |
| **Python** | No | N/A | No | Yes (`__add__`, etc.) |
| **Rust** | No | N/A | No | Yes (`impl Add for T`) |
| **Swift** | Yes | Custom groups | Yes (any Unicode symbol) | Yes |
| **Haskell** | Yes | Fixity declaration | Yes (any symbol) | Yes |
| **C++** | Yes | No (precedence fixed by symbol) | Limited (existing symbols) | Yes |
| **Scala** | Implicit | No (starts with symbol) | Yes (symbolic method names) | Yes (via implicit conversion) |
| **F#** | Yes | Declared | Limited | Yes |

### Swift: Operator Declaration

Swift requires operators to be declared at file scope with an explicit precedence group:

```swift
// Swift — operator declaration
precedencegroup PowerPrecedence {
    associativity: right
    higherThan: MultiplicationPrecedence
}

infix operator **: PowerPrecedence
func **<T: Numeric>(base: T, power: Int) -> T { ... }
```

The precedence group system prevents the "operator soup" problem: every operator's place in the evaluation order is explicitly documented at the declaration site.

### Haskell: Fixity Declarations

```haskell
infixr 5 ++   -- right-associative, precedence 5
infixl 6 +, - -- left-associative, precedence 6
infixl 7 *, / -- left-associative, precedence 7
```

### Rust: Operator Overloading (no custom symbols)

```rust
use std::ops::Add;

impl Add for Vector {
    type Output = Vector;
    fn add(self, rhs: Vector) -> Vector { ... }
}
```

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Syntax Policy | Determines whether custom operators are allowed and what characters form valid operators |
| Parsing Policy | Governs how operator precedence and associativity are resolved |
| Standard Library Policy | Determines which operators are defined in the standard library vs language core |

## Implications for Orthon

1. **Tension with LLM Generability** — Custom operators are harder for an LLM to generate correctly unless the operator is common (like `|>`). The LLM must know the operator's name, precedence, and expected behaviour. This weights against a fully open custom operator system.

2. **Prefer named over symbolic** — Following the principle from [`how/DESIGN_PRINCIPLES.md`](../DESIGN_PRINCIPLES.md) "prefer named over symbolic", Orthon should favour named alternatives. A `pipe()` or `compose()` function is more discoverable than `|>` or `.` for most readers.

3. **Limited custom operators** — If Orthon supports custom operators, they should require explicit declaration of precedence and associativity (Swift model), not be implicit (Scala model). This makes the operator's behaviour transparent to both humans and LLMs.

4. **Operator overloading for built-in operators** — Orthon should support overloading of a fixed set of built-in operators (`+`, `-`, `*`, `/`, `==`, `<`, etc.) via trait implementation, following the Rust model. This is uncontroversial and expected.

## Open Questions

1. Should Orthon support custom operators at all, or limit to overloading built-in operators (Rust model)?
2. If custom operators are allowed, should they require a special syntax (Swift `operator` keyword) or be definable via traits?
3. Should the standard library define common operators (pipe, compose, optional chaining) as built-in to avoid users needing to define them?
4. How should operator precedence interact with the parser design in [`how/architecture/PARSER.md`](../architecture/PARSER.md)?
5. Should custom operators be module-scoped or globally visible? Swift requires file-level scope; Haskell allows local operator definitions.
