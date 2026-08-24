# Classical PLT Problems — Source Catalog

> Working source catalog of classical programming-language theory (PLT)
> problems, distilled from a survey of the field. This is reference
> material for forming entries in
> [`what/LANGUAGE_PROBLEMS.md`](../what/LANGUAGE_PROBLEMS.md): each
> problem below is written so that its statement, scientific grounding,
> and cross-language analogies can be lifted into a registry entry. It
> intentionally does NOT contain Orthon mechanisms or EDRs — those are
> added during the problem-mapping phase, not here.
>
> **See also:** [`what/LANGUAGE_PROBLEMS.md`](../what/LANGUAGE_PROBLEMS.md)

## Lambda Calculus and Foundations of Computability

### The Halting Problem

**Problem.** No algorithm can determine, for an arbitrary program and
input, whether the program will terminate or run forever. This is a
fundamental limit for every programming language.

**Scientific grounding.** A classical undecidability result — one of the
best-known unsolvable problems in computability theory.

**Analogies / notes.** Applies universally; no language can provide a
general termination oracle. Total languages (e.g., Coq, Agda) sidestep it
by restricting expressiveness.

### The Program Equivalence Problem

**Problem.** It is impossible to algorithmically decide whether two
arbitrary programs are equivalent — that is, whether they compute the
same function for all possible inputs.

**Scientific grounding.** A classical undecidability result.

**Analogies / notes.** Feeds optimisers (which must be conservative) and
verification (which must be undecidable in the general case).

### Fixed Point and the Y Combinator

**Problem.** Express recursion in the untyped lambda calculus, which has
no built-in mechanism for recursive definitions. The solution is the
Y combinator, which makes it possible to define recursive functions such
as factorial.

**Scientific grounding.** Classic lambda-calculus result; recursion via
fixed points.

**Analogies / notes.** The same fixed-point machinery underpins
denotational semantics (CPO fixed points) and is implicit in how
languages provide recursion as a built-in rather than as an encoding.

### Quine (Self-Reproducing Program)

**Problem.** Write a program that outputs its own source code without
reading it from a file.

**Scientific grounding.** Tied to fundamental self-reference theorems
for programming languages.

**Analogies / notes.** A construction exercise rather than a design
target — a language makes Quines trivial or awkward depending on how
transparently its runtime exposes its own representation.

## Formal Semantics

### Language Semantics (Formal Models of Meaning)

**Problem.** Develop rigorous mathematical models that precisely describe
the meaning of every language construct. The classical approaches are:

- **Operational semantics** — describes how a program executes step by
  step on an abstract machine.
- **Denotational semantics** — maps each syntactic construct to a
  mathematical object (a function) describing its meaning; the key
  challenge is correctly giving semantics to recursive functions, which
  led to **partially ordered sets (CPOs)** and **fixed points**.
- **Axiomatic semantics** — e.g., **Hoare logic**, a formal system for
  proving correctness of imperative programs using preconditions,
  postconditions, and loop invariants.

**Scientific grounding.** Operational / denotational / axiomatic
semantics are the three classical semantics frameworks; CPO + fixed
points resolve recursive definitions; Hoare logic is the canonical
axiomatic system.

**Analogies / notes.** Every language spec implicitly chooses among
these; most real languages document operational semantics informally.

## Type Systems

### Type Inference

**Problem.** Automatically determine the types of expressions without
explicit programmer annotations. The best-known example is the
**Hindley-Milner** algorithm for polymorphic languages.

**Scientific grounding.** Hindley-Milner type inference (Damas-Milner
algorithm), the classic result for ML-style polymorphism.

**Analogies / notes.** Haskell, OCaml, SML (full inference); Rust
(inference with annotations); Java, C# (local inference only).

### Type Checking

**Problem.** Design algorithms that verify whether the types of
expressions in a program conform to the rules of the language's type
system.

**Scientific grounding.** Core static-analysis problem of type theory.

**Analogies / notes.** Static vs. dynamic languages draw the line
differently; gradual typing occupies the middle.

### Subtyping

**Problem.** Determine the relations between types (e.g., whether a
value of type `S` can be used where a type `T` is expected). The classic
example is proving the existence of a type `S` for which
`S <: S -> S` holds — a type that is a subtype of its own function type.

**Scientific grounding.** Directly related to the harder problem of
**recursive subtyping** and the theory of subtyping with recursive types.

**Analogies / notes.** OO languages (nominal subtyping via inheritance);
structural subtyping in TypeScript / Go; recursive subtyping appears in
languages with recursive object types.

## Compilation and Execution

### Lexical and Syntax Analysis

**Problem.** The classical first stages of a compiler: write a
**lexer** and a **parser** for the language's grammar.

**Scientific grounding.** Formal language theory; regular languages and
context-free grammars.

**Analogies / notes.** LALR / LL / PEG / Pratt parsing; parser
generators vs. hand-written recursive descent.

### Translation and Code Generation

**Problem.** Design compilation schemes that transform high-level
constructs into low-level code (e.g., abstract syntax into code for a
register machine).

**Scientific grounding.** Compiler construction; intermediate
representations and backend code generation.

**Analogies / notes.** AST -> IR -> machine code; tree-walking
interpreters, bytecode VMs, and native codegen are different points on
the same continuum.

### Control Flow

**Problem.** Implementation of exceptions, **continuations**, and
**tail calls** (tail recursion).

**Scientific grounding.** Control-flow semantics; CPS (continuation-
passing style); tail-call optimisation.

**Analogies / notes.** Languages differ sharply: C has no exceptions or
closures; functional languages guarantee tail-call optimisation; Scheme
is built on continuations.

## Other Fundamental Concepts

### Turing Completeness

**Problem.** Prove that a programming language has computational power
equivalent to a Turing machine.

**Scientific grounding.** Computability theory; equivalence of
computational models.

**Analogies / notes.** Most general-purpose languages are Turing
complete; some DSLs deliberately are not.

### Abstract Data Types (ADT)

**Problem.** Implement and use data structures (e.g., stacks, queues,
sets) with the internal representation hidden.

**Scientific grounding.** Classic abstraction / encapsulation result in
programming methodology.

**Analogies / notes.** Classes, modules, and traits all serve ADT-style
encapsulation in different forms.

---

## Cross-Cutting Notes

Many of these problems intersect and serve as the foundation for more
advanced topics: concurrency, effects, and dependent types.
