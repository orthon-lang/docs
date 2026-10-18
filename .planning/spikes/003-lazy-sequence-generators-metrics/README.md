---
spike: 003
idea: semantic-simplicity-metrics
name: lazy-sequence-generators-metrics
type: standard
validates: "Given the accepted LAZY_SEQUENCE_GENERATORS concept, when its constructs are derived over the 10 primitives, then the metrics expose duplicate forms and the suspension gap."
verdict: VALIDATED
related: [001, 002, 004]
tags: [simplicity, metrics, generators, control-flow, derivation]
---

# Spike 003: LAZY_SEQUENCE_GENERATORS metrics

Source: `what/concepts/LAZY_SEQUENCE_GENERATORS.md` (EDR-021). Metric definitions: see Spike 004.

## What This Validates

Given LAZY_SEQUENCE_GENERATORS, when `emit` and the generator function are derived over the kernel, then the metrics separate "three spellings of one idea" from "one idea that needs a mechanism the kernel lacks".

## Investigation Trail

1. The concept lists three "canonical forms" of one operation: `emit v`, `return sequence(v)`, `return v ->`. All three are followed by more statements in the body, so `return` here does **not** terminate. In the same concept, an explicit `return` in a generator "exhausts the iterator" (terminates). The keyword `return` therefore has three meanings in one concept.
2. Derived `emit`: store the value and the resume point, hand control back to the consumer, continue after the last `emit` on the next `next()`. Storing is `assignment`; **suspend and resume** has no kernel element.
3. The two documents disagree on how to classify it: the concept's decomposition row says "state-machine transformation ... beyond primitive composition", `PRIMITIVE_BLOCKS.md` section 10.8 says "the generator protocol is `function` + `call` composition". A third reading exists: a generator is a coroutine, i.e. `execution_context` (`defer`, EDR-085). The derivation table forces a decision between the three.
4. The same suspension idea appears in ITERATION_LOOP (loop/break/continue) and in the `?` and `?.` early exits of ERROR_HANDLING and NULL_SAFETY. Hypothesis for Phase 04.4: these are one control-transfer family (escape, repeat, one-shot resume), possibly one primitive.

## Ideas, forms and derivations

| # | Idea | Grammar production(s) | Derivation over the kernel | Kernel elements |
|---|------|----------------------|----------------------------|-----------------|
| 1 | Lazy production of a value | `emit v`; `return sequence(v)`; `return v ->` | store value and resume point; hand control to consumer; resume on `next()` | assignment, call, pack (`Some(v)`), **suspend/resume (gap)** |
| 2 | Completion | falling off the end; `return` | `None` at the next `next()` | pack/literal (`None`) |
| 3 | Generator yields an `Iterator<T>` | none (a typing rule: a function containing `emit` has type `Iterator<T>`) | the function returns a value with a `next` method | function, scope |

Composition with combinators (`.map`, `.filter`, `.take`) is delegated to ITERATOR_PROTOCOL; infinite sequences are not an idea but a permitted case of `loop` + `emit`.

## Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Ideas | 3 (+1 delegated) | |
| P, grammar productions introduced | 3 | `emit`, `return v ->`, and `return sequence(v)` as a special non-terminating use of `return` |
| Fdup, ideas with more than one production | 1 | emit has 3 spellings, 2 of them redundant; `yield` was already removed (S2) |
| K, kernel elements used | 6 of 10 | function, call, assignment, identifier, scope, pack/unpack |
| G, kernel gaps | 1 candidate (suspend/resume) | same family as the ITERATION_LOOP gap |
| R, residue | 3 | generator typing rule; undecided: `emit` ownership (move or borrow, OQ3); undecided: `emit` outside a generator (OQ1) |
| D, derivation weight | 7 | |
| C, curated depends_on / depended_on_by | 1 / 6 | in: COMPOSABLE_COLLECTION_OPS, EMIT_AS_INTERMEDIATE_RESULT, GENERATORS, ITERATION_LOOP, ITERATOR_PROTOCOL, PUSH_STREAMS (curated and mined agree) |
| Instability, curated | 0.14 | the most stable of the three: many concepts stand on it |
| Symbol meanings | `return` has 3 | terminate, emit via `sequence(...)`, emit via `->` |

## Results

Verdict: **VALIDATED**. Three results:

- Fdup = 1 and the meanings-per-symbol observation show a violation of the project's own Manifesto ("one syntactic element, one responsibility"). Candidates for removal: `return sequence(v)` and `return v ->`, keeping `emit`. Precedent: EDR-095 withdrew the equivalent forms of `emit` in EMIT_AS_INTERMEDIATE_RESULT.
- G = 1 with the same family as spike 002: the control-transfer family is the largest candidate gap in the kernel. This is the strongest argument for resolving the todo `fix-control-flow-primitive-contradiction` first in Phase 04.4.
- Instability 0.14: depended on by six concepts, so its grade is bounded from below by theirs. A concept with six dependents cannot be graded 3 whatever its sugar content. Fan-in is what encodes the "cost of deferral" most directly.
