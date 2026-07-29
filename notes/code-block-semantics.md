# Code Block Semantics — Open Question

> **Status:** Open (2026-07-29)
> **Source:** Post-Phase 4 gap analysis
> **Affects:** Primitive Blocks (`scope`), Functions (`function` concept)
> **See also:** [`CONCEPT_PIPELINE.md`](../how/CONCEPT_PIPELINE.md) § Post-Acceptance Gaps (Type B),
> [`PRIMITIVE_BLOCKS.md`](../what/PRIMITIVE_BLOCKS.md),
> [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) § Evaluation

---

## Problem

After Phase 4, the following question remains open: is a `block` a
separate language construct (analogous to an HOF-block — a callable
unit with parameter bindings) or simply an expression block (a
`scope` primitive used for expression grouping)?

A related question: should the language distinguish between
**expression lambda** and **statement block**?

---

## Context

- The Semantic Model defines `scope` as a primitive block for
  grouping expressions — it evaluates them in sequence and yields
  the last expression's value.
- `function` is a callable unit with explicit `return`.
- It is unclear whether a block that represents a callable entity
  (delegate/closure/function) is the *same* construct as an expression
  block, or a distinct syntactic concept.

**Key principle:** *Show All Canonical Forms* (`DESIGN_PRINCIPLES.md`) —
all equivalent forms must be documented. If expression lambda and
statement block are semantically equivalent, they must be shown as
variants of one construct.

**Existing rule:** Any block that represents a callable entity
(delegate/closure/function) uses explicit `return`. This is already
established in function semantics.

---

## Options

### Option 1: Block = Expression Block (scope), Lambda = Syntactic Sugar

Block is simply `scope` (the primitive). Any block representing a
callable entity uses explicit `return`. Lambda is syntactic sugar
for a function definition.

```orthon
do { expr; expr; }        // expression block — evaluates to last expression
|x| expr                   // expression lambda — sugar for fn(x) { return expr }
|x| -> Type { stmt; ret } // typed statement block with explicit return
```

**Pros:**
- Minimal — no new construct added to the core
- Orthogonal — block ≠ function; the distinction is clear
- Consistent with existing rule (explicit `return` in callable blocks)

**Cons:**
- Requires explicit `return` in lambda bodies, which can be verbose
  for single-expression lambdas

---

### Option 2: Block = HOF-Block (Separate Language Construct)

Block is a distinct language construct representing a callable
entity. It differs from `scope` by having parameter bindings and
can be passed as an argument.

```orthon
{ |x| expr }               // HOF-block: block with parameters
{ stmt; return expr }     // statement block
{ expr; expr }            // expression block (implicit last-expr return)
```

**Pros:**
- Explicit distinction between scope and callable block
- Familiar to developers from Kotlin/Swift/Groovy

**Cons:**
- Adds a new language construct — violates minimal core principle
- Overlaps with `function` semantics (both are callable)
- Blurred boundary with `scope`

---

### Option 3: Three Distinct Constructs

Explicitly distinguish three separate constructs:

1. **Expression block** — `do { expr; expr }` — grouping, evaluated
   to last expression
2. **Lambda expression** — `|x| expr` — anonymous function, is an
   expression
3. **Statement block** — `{ stmt; return expr }` — block with
   statements

**Pros:**
- Each construct has a clear, single purpose
- No ambiguity for the programmer

**Cons:**
- Three constructs instead of one — significant core language burden
- Increased learnability tax
- Violates *Show All Canonical Forms* if these turn out to be
  semantically equivalent

---

## Decision Pipeline Check

| Question | Answer |
|----------|--------|
| **Q1 (Problem)** | Yes — ambiguity: developers cannot tell which construct to use when |
| **Q2 (Language vs Library)** | Language — block semantics affect the evaluation model |
| **Q5 (New semantics vs sugar)** | Options 1 and 3 add semantics; Option 2 also adds semantics |
| **Q10 (Worth adding?)** | Open — requires further analysis; preliminary recommendation below |

---

## Preliminary Recommendation

**Option 1** is the most orthogonal choice:

- `scope` remains the sole primitive construct for grouping
- Lambda is syntactic sugar: `|x| expr` ≡ `fn(x) -> T { return expr }`
- `do { ... }` is a named expression block (Tier 4 inline decision)
- Explicit `return` is mandatory in any callable entity — this is
  already established in function semantics

**Open sub-questions that need resolution:**

1. Does the language need a `do { ... }` syntax, or is bare
   `{ ... }` in expression position automatically an expression block?
2. How do `for`, `if`, `while` interact with block syntax — do they
   introduce an implicit scope, or require explicit braces?
3. LLM Generability: can an LLM consistently distinguish expression
   position from statement position to generate correct block syntax?

---

## Related Concepts

- `scope` primitive — [`PRIMITIVE_BLOCKS.md`](../what/PRIMITIVE_BLOCKS.md)
- `function` concept — [`what/concepts/FUNCTIONS.md`](../what/concepts/FUNCTIONS.md)
- `return` semantics — [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) § Evaluation
- Expression position vs statement position — [`SYNTAX.md`](../what/SYNTAX.md) (Phase 5)

---

## Resolution

<!-- To be filled after discussion -->
| Field | Value |
|-------|-------|
| **Decision date** | |
| **Chosen option** | |
| **Rationale** | |
| **EDR / inline ref** | |
| **Implemented in** | |