# Compiler-Managed Concurrency (Research Hypothesis)

> **Status:** research/unverified.
> **Layer:** How — concept research, not accepted Orthon semantics.
> **Date:** 2026-11-17.
> **Tier:** important — candidate simplification of existing concurrency mechanisms; no new primitive is presumed necessary.
> **Source:** Author discussion, "Atomicity of counter++", continued in this repository.
> No acceptance decision, human sign-off, or EDR is recorded here.

## Problem

Concurrent code often makes the programmer choose both the required behavior
and its machinery: an executor, asynchronous API, synchronization algorithm,
or contention-management technique. The research goal is to reduce these
mechanical choices while preserving explicit ownership, execution boundaries,
and observable behavior.

Pseudocode only; `counter++` is the motivating notation, not proposed Orthon syntax.

```text
exclusive local owner:       counter++
single-owner context:       submit increment to the owner
independent workers:        compute partial counts; explicitly combine results
arbitrary shared mutation:  counter++ does not establish a concurrency contract
```

These cases can have similar arithmetic but different ownership, ordering,
and observation requirements. An increment alone cannot determine whether
intermediate reads must observe every update, whether the old value is
returned, or whether several related fields must change together.

## Unified Hypothesis

**For programs expressed through Orthon's accepted Invocation and ownership
boundaries, compiler analysis and an explicitly selected Implementation
Strategy can remove most programmer choices about scheduling, suspension
machinery, and internal synchronization, using proven ownership and effect
information, while preserving the program's permitted observable behaviors
and keeping execution, result, and lifecycle boundaries explicit.**

The claim is conditional and falsifiable. Test whether the existing semantic
information is sufficient for a representative workload suite, whether the
source requires fewer mechanism-specific choices, and whether the resulting
implementations remain correct and practically competitive. The dimensions
below test this one hypothesis; they are not separate feature proposals.

## Examples From Other Languages

Java virtual threads provide the motivating analogy: a runtime can decouple
a logical thread from an OS thread so that sequential-looking request code
scales for workloads with substantial waiting. Oracle describes the benefit
as throughput rather than reduced latency; see the primary
[Java 24 virtual-thread documentation](https://docs.oracle.com/en/java/javase/24/core/virtual-threads.html).
This is evidence for hiding execution machinery, not evidence that arbitrary
shared increments become atomic or that CPU-bound work becomes faster.

For Orthon, investigate lightweight tasks, state machines, fibers, and
worker pools as alternative implementations of existing contexts. No Java
API, universal virtual-thread runtime, or new context constructor follows
from the analogy. Existing comparative research is in
[Concurrency Approaches and Trends](../CONCURRENCY_APPROACHES_AND_TRENDS.md).

## Existing Decisions and Compatibility Boundaries

The authoritative baseline is
[EDR-085](../../../decision_records/architecture/EDR-085-execution-context-invocation.md).
Older research is provenance, not an alternative active specification.

| Area | Existing baseline and constraint on this hypothesis |
|---|---|
| Async/await | EDR-085 supersedes EDR-047 and removes the `async` modifier and coloured functions. `defer(obj)` and named `await` materialisation remain; the hypothesis does not restore async function types. |
| Immediate calls | `fn(args)` stays synchronous and inline. No ambient context inference, automatic task creation, or newly implicit semantic suspension points. A changed suspension model would require a separate decision. |
| State ownership | `delegate(obj)` and `defer(obj)` own one object and serialise submitted invocations in order through `<-`. The compiler cannot silently replace this with independent shared writers. |
| Distributed work | `spawn()` and `fork()` dispatch stateless work; captured data must meet `Send` and `Move` respectively. Distribution's glyph remains provisional under EDR-085. |
| Results and lifetime | Context-specific materialisation and explicit resolution of pending worker work remain. No implicit join, detach, retry, or cancellation on ordinary scope exit. |
| Libraries | [Concurrency utilities](../../../../what/concepts/CONCURRENCY.md) are accepted as StdLib under EDR-049. Its older delegate examples must be interpreted through EDR-085, not used to revive `act` or the superseded concurrency model. |
| Ownership analysis | [Ownership research](../essential/OWNERSHIP.md) distinguishes accepted single-owner/move/borrow rules from unresolved details. [Borrowing and exclusivity](../essential/OWNERSHIP_BORROWING_EXCLUSIVITY.md) leaves analysis precision open. No assumed perfect alias proof. |
| Effects | [Effect Footprint](../essential/EFFECT_FOOTPRINT.md) is an open hypothesis. Its read/mutate/consume model is a candidate proof input, not an accepted annotation system or sufficient proof of independence. |
| Optimization | [Design Principles](../../../DESIGN_PRINCIPLES.md) requires Semantics Before Optimization and Explicit Optimization. Adaptive performance strategies require intentional selection; automatic analysis alone does not authorize silent optimization. The [Optimization Model](../../../../what/OPTIMIZATION_MODEL.md) remains a Phase 7 placeholder. |

An implementation may park a physical worker only where doing so preserves
the accepted execution contract. Releasing that worker must not release a
state owner's exclusivity or allow otherwise forbidden reentrancy.

## Mechanisms to Investigate

| Candidate mechanism | Proof obligation or boundary |
|---|---|
| Plain local mutation | Prove exclusive access for the relevant interval, including aliases and callbacks; do not merely count syntactic writers. |
| Single-owner mailbox execution | Preserve submission order, isolation, effects, and extraction behavior. Logical serialization does not require one dedicated OS thread per owner. |
| Lightweight waiting tasks | Preserve explicit materialisation and resource lifetime. Measure foreign/blocking calls that retain a physical worker. |
| Internal atomic RMW, CAS, or locks | May implement runtime queues or an already defined contract; cannot legalize source-level shared mutation rejected by ownership rules. |
| Combining or batching | Prove equivalence for returned values, intermediate observations, overflow, errors, and external effects. Same final count alone is insufficient. |
| Adaptive switching | If supported by the selected Strategy, prove safe publication, quiescence or another valid transition protocol, and preservation of in-flight work. Include transition cost in measurements. |

No mechanism is selected. With insufficient proof, retain a known-correct
implementation or diagnose unsupported source; do not silently weaken its
contract. A retrying CAS loop cannot repeat an effectful operation merely
because its arithmetic resembles an increment.

## Open Questions

1. What minimum information is already available from ownership, context
   boundaries, declarations, and capture rules? Where would additional
   contracts actually be necessary, particularly under separate compilation,
   dynamic dispatch, callbacks, or FFI?
2. How far can suspension machinery be hidden within EDR-085's cooperative,
   explicit-extraction model? Which blocking libraries need adapters or
   dedicated workers, and which proposed transformations would change semantics?
3. At what granularity can a proof remove internal synchronization: one
   operation, invocation, owner lifetime, or execution phase? How is the
   proof invalidated when data escapes?
4. Is owner serialization sufficient for practical counters and services?
   If a workload genuinely needs shared linearizable state, relaxed metrics,
   or transactions, can library composition express it? Such contracts are
   alternatives to investigate, not accepted concurrency types.
5. Which observations prevent combining, including intermediate reads,
   returned old values, multi-field invariants, and noncommutative effects?
6. Can runtime adaptation repay its measurement and transition costs without
   making latency or debugging unpredictable? What evidence justifies a JIT
   requirement rather than an optional Strategy?
7. How do errors, cancellation, and cleanup constrain each transformation?
   These remain coordinated with the existing
   [error propagation](../essential/EXECUTION_CONTEXT_ERROR_PROPAGATION.md),
   [cancellation](../essential/EXECUTION_CONTEXT_CANCELLATION.md), and
   [structured lifecycle](../essential/EXECUTION_CONTEXT_STRUCTURED_LIFECYCLE.md)
   hypotheses; this document chooses no policy for them.
8. Can the approach remain useful on embedded targets without a JIT or rich
   scheduler, with unsupported capabilities diagnosed rather than hidden?

## Verification Plan and Refutation Criteria

This repository contains design documents only. The following is a future
research protocol, not a claim that executable validation has occurred.

Before evaluating candidates, freeze a workload suite, hardware and resource
budgets, permitted observations, and numerical performance tolerances. Include
exclusive counters, contended owner counters, waiting I/O fan-out, CPU-bound
workers, partial-result aggregation, effectful updates, FFI, and cleanup.
Compare each candidate with a simple correct implementation and a manually
tuned implementation of the **same** contract. Do not compare a relaxed metric
against a linearizable counter as if their semantics were identical.

| Test dimension | Evidence and failure condition |
|---|---|
| Semantic preservation | Compare allowed histories: results, intermediate reads, effect order, ownership transfers, destruction, and failures. Use bounded interleaving exploration and transition stress. Any forbidden history rejects that mechanism. For a linearizable comparison contract, check a legal sequential history consistent with real-time order. |
| Negative cases | Overlapping mutable captures, unknown aliases, effectful retries, and unresolved worker work must remain rejected or conservatively handled. Accepting an unsafe case refutes soundness. |
| Mechanism burden | Count mechanism-specific types, annotations, bridges, locks, executor decisions, and configuration steps across the whole application. Proposed target: eliminate more than half of the baseline choices on at least two thirds of the frozen workloads, with no hidden equivalent configuration. Failure refutes the claim of removing most choices for that suite. |
| Cost and scalability | Record throughput, p50/p99 latency, memory per pending task, OS-thread count, compilation cost, and adaptation overhead across contention and waiting scales. Missing predeclared tolerances rejects practical competitiveness for the affected target. |
| Composability | Repeat with callbacks, separate compilation, owner extraction, cancellation, and resource cleanup. A required new semantic exception is evidence against sufficiency of the existing model. |
| Strategy portability | Test at least a lightweight-task implementation and a restricted non-JIT implementation against the same semantic cases. Document capability limits and avoid claiming identical performance. |

Partial success narrows the hypothesis: hiding scheduling may succeed while
adaptive synchronization or batching fails. Record that boundary rather than
turning an unsuccessful mechanism into a new language guarantee.

## Decision Pipeline — Preliminary Triage

| Question | Research assessment |
|---|---|
| 1. Problem? | Reduce mechanical concurrency choices while retaining locally understandable correctness obligations. |
| 2. Language or library? | Start with compiler/runtime Strategy and StdLib composition; no demonstrated need for Core extension. |
| 3. Existing primitives? | Invocation, Execution Context, ownership, and lifetime are the baseline to test. |
| 4. Principle conflict? | Invisible semantic changes and unselected adaptive optimization would conflict with explicitness. |
| 5. New semantics? | Mechanism substitution must preserve semantics; new consistency or suspension guarantees would require separate review. |
| 6. Composition? | Test owner contexts plus independent workers and explicit result aggregation first. |
| 7. Sugar? | No new syntax or shorthand proposed. |
| 8. Optimization? | Candidate lowering and adaptation belong to implementation research under explicitly selected Strategies. |
| 9. Compatibility? | Preserve EDR-085, ownership checks, observable effects, and explicit worker resolution. |
| 10. Worth adding? | Unverified; retain one research hypothesis until the protocol provides evidence. |

## Alternatives, Trade-off, and Route to Decision

A universal atomic `counter++` hides too little information to express owner,
ordering, and lifecycle intent. A new actor keyword or restored `async`
modifier duplicates superseded mechanisms. Conversely, requiring application
code to select every low-level synchronization mechanism defeats the research
goal. Retain these as comparison baselines, not adopted designs.

The accepted trade-off under investigation is reduced application machinery
in exchange for more compiler/runtime work. The new contradiction is that
simpler source may require harder proofs and less predictable performance;
conservative analysis may then return complexity to users through diagnostics
or extra contracts. Measure that relocated cost explicitly.

Before any acceptance, apply the
[Decision Pipeline](../../../process/DECISION_PIPELINE.md) and
[Language Design Gate](../../../gates/_language-design.md) to the concrete
result. Pending evidence includes minimality, observable equivalence,
orthogonality, Strategy independence, and explicit optimization selection.
Any demonstrated need to change EDR-085 or ownership rules must be reviewed
as a separate semantic proposal. This research adds no syntax choice, so no
new Syntax Decision Queue entry is required. An acceptance EDR requires the
author's human sign-off.
