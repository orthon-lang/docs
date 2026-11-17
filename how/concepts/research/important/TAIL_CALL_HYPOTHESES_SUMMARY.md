# Tail-Call and Execution-Contract Hypotheses — Reasoning Summary

> **Status:** research/unverified.
> **Layer:** How — synthesis of linked research inputs, not accepted Orthon semantics.
> **Date:** 2026-11-17.
> No acceptance decision, human sign-off, or EDR is recorded here.

## The Classical Problem Cluster

The four hypotheses are connected by one question: how can a language preserve
the meaning of a computation while controlling the representation of its
continuation and its execution resources?

| Hypothesis | Classical PL problem | Narrow research claim |
|---|---|---|
| H1 TRE | Tail-position analysis, compiler lowering, operational equivalence | Direct tail self-calls may be lowered to iteration |
| H2 PTC | Continuation semantics, calling conventions, bounded-space execution | Eligible arbitrary tail calls may avoid an accumulating return chain |
| H3 Control Transfer | CPS, abstract machines, continuations, coroutines, exceptions | H1 and H2 may share an IR-level control operation |
| H4 Execution Contracts | Hoare logic, effects, refinement/resource types, sound composition | Execution properties may be declared independently of their implementation |

## Reasoning Chain

1. A tail call is identified by the continuation that remains after the call,
   not merely by the visual presence of `return`.
2. TRE handles a narrow case: a compiler can replace a direct self-call with a
   loop if evaluation, cleanup, ownership, and failure behavior are preserved.
3. PTC is broader: it concerns arbitrary tail-call chains, including mutual
   recursion and calls to unrelated functions. It therefore reaches into ABI,
   debugging, exceptions, and runtime representation.
4. Control transfer could unify these cases, but a new primitive is justified
   only if existing call semantics and IR composition cannot express them.
5. Contracts should state the property that users depend on. A structural
   `@tailrec` check and a resource promise such as `guarantees stack: constant`
   may overlap, but they are not equivalent without a precise scope.

## Why Mainstream Languages Often Choose Differently

Scheme made proper tail recursion a language guarantee. Other languages often
retain ordinary stack frames for predictable stack traces, cleanup and
destructor behavior, exception unwinding, FFI/ABI compatibility, and simpler
debugging. Some still perform best-effort tail-call optimization, while others
prefer explicit loops or compiler-generated state machines. This is a design
trade-off, not evidence that tail calls are obsolete.

## Consequences for the Orthon Research Agenda

The hypotheses should be tested in this order:

1. Define tail position and the exact observable behavior that must be
   preserved.
2. Establish whether H1 is useful without committing to H2.
3. Determine whether H3 belongs only in IR or needs user-visible semantics.
4. Define the scope and composition rules of any H4 guarantee.
5. Compare all candidates across strategies, debug traces, cleanup, dynamic
   dispatch, foreign calls, and higher-order functions.

No step accepts a syntax, a policy, or a semantic guarantee. The current
research result is a separation of concerns: analysis, optimization, control
representation, and user-visible guarantees must not be treated as one
mechanism.

## Related Hypotheses

- [H1 — TRE](TAIL_RECURSION_ELIMINATION.md)
- [H2 — PTC](PROPER_TAIL_CALLS.md)
- [H3 — Control Transfer](CONTROL_TRANSFER.md)
- [H4 — Execution Contracts](EXECUTION_CONTRACTS.md)
- [Accepted Contracts](../../../../what/concepts/CONTRACTS.md)
- [Effect Footprint hypothesis](../essential/EFFECT_FOOTPRINT.md)
