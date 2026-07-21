# Functions

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

Functions are the primary unit of computation in most programming languages.
The core problem: **how does a language define, compose, and invoke units of
computation** while supporting multiple styles (named functions, anonymous
closures, generators, async coroutines) without fragmenting the programmer's
mental model?

In many languages, functions, lambdas, methods, and closures are separate
concepts with separate syntax and separate rules. This fragmentation makes
it harder to refactor code (turning a named function into a closure requires
syntax changes) and harder for LLMs to generate correct code (multiple
syntactic patterns for the same semantic concept).

## Principles

1. **First-class** — Functions are first-class values. They can be assigned
   to variables, passed as arguments, and returned from other functions.
2. **Uniform call syntax** — Named functions and anonymous closures use the
   same call syntax. Converting between forms requires no call-site changes.
3. **Closure capture is explicit** — Variables captured by a closure must be
   explicitly listed or syntactically visible. No silent capture of the
   surrounding scope.
4. **Named before anonymous** — Named function declarations are preferred
   for top-level and module-level computation. Anonymous closures are
   available for scoped and higher-order use.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Evaluation Policy | Defines when function bodies are evaluated (eager, lazy, or hybrid) |
| Lifetime Policy | Controls how long captured variables live in closures |
| Allocation Policy | Affects where closure environments are allocated (stack vs. heap) |
| Mutability Policy | Governs whether a function can mutate captured state |

## Model (What)

Functions are first-class values with a single unified call syntax.

```orthon
# Named function
fn add(a: Int, b: Int) -> Int
    a + b

# Anonymous closure assigned to variable
multiply = fn (a: Int, b: Int) -> Int
    a * b

# Closure with explicit capture
counter = fn [count: Mutable(0)] () -> Int
    count += 1
    count

# Calling — same syntax regardless of declaration form
add(3, 4)        # 7
multiply(3, 4)   # 12
counter()        # 1
```

All functions have a type signature: `(ArgType1, ArgType2) -> ReturnType`.
Closures are functions that capture variables from their enclosing scope.

## Default Strategy

Functions are compiled to anonymous, non-closure forms when possible
(no captures). When captures are present, the compiler allocates a closure
environment on the heap (for flexibility) with escape analysis to promote
to stack allocation where safe.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Stack allocation | Closure environment allocated on stack | Short-lived closures that don't escape their scope |
| Closure flattening | Inline captured variables into the caller | Performance-critical closures with few captures |
| Tail-call optimization | Recursive calls reuse stack frame | Deeply recursive functional code |
| No-closure mode | Compile error on captures | Embedded or safety-critical contexts |

## Open Questions

1. What is the exact syntax for explicit closure capture lists? (`fn [...] (...)` or `fn [...](...)`?)
2. Should Orthon support variadic functions? If so, what is the syntax?
3. Should functions have named/default parameters?
4. How do functions interact with the type system — are function types
   nominal or structural?

## Decision History

- **First-class functions:** Adopted. Closures and named functions share the
  same call syntax. Alternatives considered: methods-only (rejected: limits
  functional composition), C-style function pointers (rejected: no closures).
- **Explicit capture:** Adopted over implicit closure capture (JavaScript model).
  Rationale: Explicitness principle — captured variables must be visible in the
  function declaration.
- **Deferred to Phase 3:** Full closure capture semantics, variadic parameter
  design, named/default parameters, function type subtyping.
