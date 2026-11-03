# ADT Variant Separator Syntax

> **Status:** Phase 5 input — syntax hypothesis; not a concept under current review.
> The current `|` spelling remains accepted by [EDR-039](../decision_records/architecture/EDR-039-algebraic-data-types.md).
> No alternative spelling is accepted by this document.
> Home: [`how/syntax/`](README.md).
> **Opened:** 2026-11-03.

## Question

How should an ADT declaration separate newly introduced variant constructors
without giving one syntactic element two context-dependent responsibilities?

The [ADT concept](../../what/concepts/ALGEBRAIC_DATA_TYPES.md) defines a named,
tagged, closed set of variants with exhaustive matching. [Structural union
types](../../what/concepts/UNION_INTERSECTION_TYPES.md) combine existing types
without a tag or an exhaustiveness guarantee. Both accepted concepts currently
use `|` in a `type` declaration. The [syntax exceptions
registry](../SYNTAX_EXCEPTIONS.md) records this as an accepted exception.

The author proposes `||` as a possible ADT separator to reduce that exception.
However, [smart-cast analysis](../decision_records/architecture/EDR-060-smart-cast.md)
already uses `||` for short-circuit logical OR. The proposal therefore moves
the collision unless logical OR is also reconsidered. The `||` collision is
registered as **OPEN CONFLICT**; this file owns its syntax review.

## Semantic Boundary

The choice of separator must preserve the accepted distinction:

| Form | Operands | Result | Match guarantee |
|------|----------|--------|-----------------|
| ADT declaration | New variant constructor names, optionally with fields | Named, tagged, closed ADT | Exhaustive |
| Structural union | Existing named or literal types | Untagged union type | No exhaustiveness guarantee |
| Logical OR | Boolean expressions | Boolean result with short-circuit evaluation | Not applicable |

A spelling change cannot turn an ADT variant into a standalone type, add a tag
to a structural union, or change short-circuit evaluation. Any such change
belongs in the semantic concept pipeline, not this syntax hypothesis.

## Candidates for Review

The examples below are **notation candidates**, not accepted Orthon code.

| Candidate | ADT notation | Effect on current collision | New question |
|-----------|--------------|-----------------------------|--------------|
| A. Retain `\|` | `type Color = Red \| Green` | Keeps the registered `\|` exception. | Is the shared "choice" reading sufficiently uniform despite different operands and guarantees? |
| B. Use `\|\|` for ADTs | `type Color = Red \|\| Green` | Separates ADTs from union `A \| B`. | What replaces logical `\|\|`, or is a new `\|\|` exception justified? |
| C. Use a named ADT marker or separator | For example, a proposed `variant` keyword | Could reserve `\|` for union types and `\|\|` for logical OR. | Which form stays concise, unambiguous, and consistent with declarations and multiline layout? |
| D. Change the union spelling instead | Preserve `type Color = Red \| Green`; choose a distinct type-union form | Could remove the registered `\|` exception. | What is the impact on union annotations, literal-type unions, and existing examples? |

No candidate receives a verdict here. In particular, `or` as a replacement
for logical `||` is another proposed spelling, not a settled decision.

## Cases the Review Must Discriminate

1. A payload-free ADT beside an existing named type: if `Red` already names a
   type, can a declaration still introduce a `Red` variant? The parse and
   name-resolution rule must not depend silently on lookup order.
2. A payload-bearing ADT beside a union of named types: compare
   `Circle(radius: Float)` with an existing `Circle` type, including generic
   and recursive declarations.
3. A union of literal types (`"GET" | "POST"`) beside a payload-free ADT.
   Both are closed lists in source text but have different match guarantees.
4. Logical OR inside a type-adjacent expression, especially a condition whose
   first operand narrows the second through short-circuit evaluation.
5. Multiline declarations, nested type positions, and lexical recognition of
   `|`, `||`, and the accepted invocation operator `|>`.
6. Existing documentation and examples: count the migration surface for each
   candidate before weighing readability or LLM generation claims.

## Decision Pipeline Pre-Filter

| Question | Preliminary answer |
|----------|--------------------|
| Problem and layer | A **syntax** collision between already accepted language concepts; no library or implementation-policy solution applies. |
| Existing semantics, composition, and sugar | ADTs, structural unions, and logical OR already have distinct meanings. Changing the separator should be surface notation only. |
| Principle compliance | Candidate A retains a registered exception. Candidate B creates an open `\|\|` conflict. Candidates C and D need a full symbol and keyword sweep. |
| Optimisation and compatibility | No optimisation. The language is pre-v1.0, but the edit spans accepted concept examples, EDRs, syntax records, and future grammar. |
| Worth deciding | Yes: [Syntax Principle 2](../../what/SYNTAX.md#syntax-principles) and the existing registry entry make the overload explicit. This table does not select a candidate. |

This is preliminary Stage 2 input. The full ten-question run, Stage 3 collision
sweep, syntax principles, acceptance gates, and relevant language-design gate
criteria follow the [Syntax Pipeline](../SYNTAX_PIPELINE.md). A choice requires
the author's [human sign-off](../SYNTAX_PIPELINE.md#stage-6b--human-sign-off)
before an Architecture EDR or accepted syntax record is written.

## Coupling and Open Questions

- Does the author want `||` to retain its logical-OR role? If not, what
  context-independent logical-OR spelling replaces it?
- Is the `|` exception acceptable because both type forms express some kind
  of choice, or do the tag and exhaustiveness differences make that reading
  misleading?
- Should a distinct ADT form introduce a named marker, change the separator,
  or instead change the structural-union notation?
- What grammar and name-resolution rule distinguishes variant introduction
  from references to existing types without semantic lookup during parsing?
- Which option minimizes total exceptions and ambiguity across accepted
  concepts after the entire symbol inventory is checked?

## Acceptance Work

1. Run the full [Decision Pipeline](../process/DECISION_PIPELINE.md) for the
   syntax choice, recording all ten answers here.
2. Sweep accepted syntax and concepts for collisions, including `||` in
   smart casts and `|>` in invocation; record the Stage 3 outcome for each
   candidate in the [registry](../SYNTAX_EXCEPTIONS.md).
3. Compare the candidates against all five
   [Syntax Principles](../../what/SYNTAX.md#syntax-principles) and the
   applicable [language-design gate](../gates/_language-design.md).
4. Test representative declarations, annotations, conditions, nesting, and
   match sites for parsing, type checking, readability, and LLM generation.
5. Record the author's LOCKED or changes verdict before filing an EDR.
6. If accepted, update the relevant EDRs, concepts, registry, accepted syntax
   record, syntax hub, and parser grammar with reciprocal links.
