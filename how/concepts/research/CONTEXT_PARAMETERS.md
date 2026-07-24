# Context Parameters (Dual Parameter Model)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Hypothesis status:** Proposed
> **Last updated:** 2026-07-24

## Hypothesis

> *If Orthon, at the semantic level, separates function parameters into two independent spaces — **data** ("what is being computed?") and **context** ("in what environment is the computation taking place?") — and provides a dedicated implicit-passing mechanism with automatic resolution (analogous to `given`/`using`) for context parameters, then code becomes more expressive, the routine threading of service objects through call chains disappears, and the compiler can guarantee environment correctness at build time.*

## Issue (Why)

In traditional languages, all function arguments are equal. This causes objects from two different categories to mix in function signatures:

- **Domain-level objects** that reflect the problem domain (e.g., order, product list, geometric shape).
- **Execution environment objects** that describe the running environment (logger, database connection, memory allocator, sort strategy, current transaction).

Context has no direct relationship to the function's logic, yet it is threaded through dozens of intermediate calls only because something deeper in the stack needs it. This creates fundamental problems:

- **Signal-to-noise ratio in signatures** — reading `f(a, b, logger, repo, metrics)` makes it impossible to tell what the computation is actually about vs. what is infrastructure plumbing.
- **High refactoring cost** — adding a new context parameter (e.g., tracing) forces signature changes across every function in the call chain.
- **Environment mismatch errors** — it is easy to pass the wrong logger or configuration, and such errors often manifest only at runtime.
- **Test pollution** — testing pure functions requires artificially creating and passing context objects, even when they do not affect the behavior under test.

Separating data from context lets the language distinguish the subject of computation from its environment.

## Proposal

Orthon introduces a fundamental split of parameters into two semantic spaces:

- **Data parameters** — ordinary parameters describing the objects the function operates on. They answer the question "**what** is being processed?"
- **Context parameters** — parameters declared in a special `using` block. They describe the environment in which the computation occurs. They answer the question "**in what environment**?"

Example syntax (one possible formulation — the concept matters, not the keywords):

```scala
// Function: data = list, context = ord
def sort[A](list: List[A])(using ord: Ord[A]): List[A]

// Class: no data in constructor, context = ord
class Sorter[A](using ord: Ord[A]):
    def sort(list: List[A]): List[A] = …
```

### Key Properties

- **Context is not passed explicitly.** If a matching `given` definition (an analogue of a context provider) is available in the lexical scope, the compiler plugs it in automatically.
- **Context resolution is static, at compile time.** Visibility and priority rules apply: local `given` overrides imported ones, a more specific type beats a more general one. Ambiguity produces a compile-time error, which the developer resolves by specifying the candidate explicitly.
- **Context can be captured via partial application.** `val sortInt = sort[Int](using summon[Ord[Int]])` turns a context-parameterized function into an ordinary data-only function. This provides natural currying over environment.
- **Context is not required to be immutable.** A transaction, metrics collector, or logger may mutate — they remain context because they describe the environment, not the entities being processed. What matters is the semantic role, not immutability.
- **Compatibility with effects and mutation.** Data parameters, context parameters, effects, and mutation become orthogonal function dimensions:

  ```
  proc draw(shape)        // data
      using renderer      // context
      throws IO           // effects
  ```

  This forms a coherent model of computational contexts.

Orthon does not merely "implement DI" — it rethinks the nature of function arguments. Context parameters are not just implicit arguments; they are an independent semantic axis, co-equal with data and effects.

## Related Concepts & Alternatives

- **Scala 3 `given`/`using`** — Direct predecessor. Orthon adopts best practices but makes the model more holistic, elevating it to a fundamental parameter space split rather than syntactic sugar for implicit arguments.
- **Haskell typeclasses** — Dictionaries passed implicitly solve the same problem, but hide the fact of passing entirely. In Orthon, the `using` block remains an explicit part of the signature, making the contract more readable.
- **Reader monad / effect systems** — The functional approach to threading environment through computations. Safe but verbose and requires writing monadic code. Orthon's context parameters provide comparable safety without monadic wrapping.
- **Container-based DI (Spring, Guice)** — External DI frameworks solve the threading problem but operate outside the language, losing static guarantees and adding reflection. Context parameters are a statically-checked, compiler-resolved dependency injection mechanism that is a natural part of the language.
- **Named parameters and default values** — Partially help with argument ordering but do not solve the core problem of threading context through dozens of functions that do not use it themselves. They only make call sites slightly cleaner without changing the semantic model.

## Trade-offs

- **Higher learning curve.** Developers must learn the two-space model, `given` visibility rules, priority resolution, and partial context application. Initially this is harder than threading everything manually.
- **Risk of "context magic."** Overuse of `using` can make it hard to trace where a particular context came from. The language should provide transparency tooling (e.g., compiler commands to show resolved `given` instances) and coding style should limit context to genuinely infrastructural concerns.
- **Compiler complexity.** The search and resolution mechanism for context candidates requires a full traversal of the type graph and lexical scopes, increasing implementation complexity and potentially slowing compilation of large projects.
- **Limited dynamism.** Context is determined at compile time. If runtime environment switching is required (e.g., plugin loading), explicit factories or wrappers are needed, breaking the automatic resolution.
- **Ecosystem ambiguity.** When using multiple libraries, `given` conflicts may arise requiring manual resolution. The language must provide flexible import and exclusion tools for specific `given` instances.

## Gate Criteria

- [ ] Define the syntax for `using` blocks in function signatures.
- [ ] Define `given` definition syntax and scoping rules.
- [ ] Define resolution priority rules (lexical, import, type-specificity).
- [ ] Define partial application semantics for context parameters.
- [ ] Document interaction with effect systems (throws, async, etc.).
- [ ] Document interaction with existing concepts (Data, Data Modifiers, Sequence).
- [ ] Add entry to `what/GLOSSARY.md` for "context parameter" and "given resolution."
- [ ] Create EDR for the Dual Parameter Model decision.
