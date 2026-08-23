# System Error (Research — graduated)

> **📚 RESEARCH SOURCE — provenance for an accepted concept.**
> Home: `how/concepts/research/essential/` — semantic bedrock tier.
> **Graduated 2026-08-23:** accepted via
> [EDR-089](../../../decision_records/architecture/EDR-089-system-error-taxonomy.md);
> the accepted concept lives at [`what/concepts/SYSTEM_ERROR.md`](../../../what/concepts/SYSTEM_ERROR.md).
> This research file is retained as traceable provenance per `what/concepts/README.md`.
>
> **Last updated:** 2026-08-23
>
> **Related:** `ERROR_HANDLING.md` (EDR-020), `ERROR_UNION.md` (EDR-023),
> `COMPILER_AS_STATIC_ANALYZER.md` (EDR-030), `ALLOCATION.md` (EDR-034),
> `../../../../what/EXECUTION_MODEL.md`, `../../../../what/GLOSSARY.md`

## Issue (Why)

Orthon's error model (`Result<T, E>`, Error Union `!T` — EDR-020, EDR-023)
defines what happens for **expected, recoverable** failures: they are
values, declared in the function contract, and the compiler enforces
handling. But the model is silent about the complement: what happens when
a failure is **not representable as a value** — when the program violates
its own invariant, when the runtime fails to honour the language contract,
or when the platform cannot provide what the program needs?

Today the vocabulary is incomplete and inconsistent:

- **Panic** exists only as an unnamed alternative strategy inside
  `ERROR_HANDLING.md` — an escape hatch with no diagnostic contract and no
  definition of *whose* fault it is.
- **Memory exhaustion** is unspecified entirely — `ALLOCATION.md` defines
  the allocation *mechanism* but not the *behaviour on exhaustion*.
- **Runtime or platform defects** have no name at all.

Without a taxonomy, three things suffer: (1) the **LLM toolchain** cannot
rely on a deterministic contract for terminal conditions (EDR-030 requires
machine-readable diagnostics for every failure), (2) the **process
contract** (exit codes, signals, termination semantics) is undefined, and
(3) **programmer reasoning** conflates bug, environment, and runtime
defect — three different causes that demand three different responses.

The core problem: **draw the boundary between a domain error (a value the
program handles) and a system error (a defined terminal condition the
program does not handle), and give every system error a name that says
whose contract was violated.**

## Principles

1. **Domain errors are values; system errors are not.** A failure is a
   *domain error* if and only if it is representable in the program's type
   system as declared fallibility (`Result<T, E>` / `!T`). A *system
   error* is a failure that is not representable as a value — its contract
   is defined diagnostic + defined termination, never recovery.
2. **One owner, one class.** Each system-error class names the layer whose
   contract was violated — program, runtime, or platform — mapping directly
   onto the Execution Environment of `ARCHITECTURE.md` (Compiler · Runtime ·
   Platform) and the semantics/execution decoupling of `EXECUTION_PROGRAM.md`.
3. **System errors are not catchable.** There is no `try-catch` for system
   errors (EDR-020, EDR-023). Recovery is impossible by construction — a
   program cannot handle a platform that vanished or a runtime that
   corrupted itself. "Handling" means a defined diagnostic and defined
   termination semantics, not continuation.
4. **Classification follows policy, not language.** The same nominal
   condition may be a domain error under one Implementation Strategy and a
   system error under another (memory exhaustion is a recoverable domain
   error under `Arena`/`Static` and a terminal `PLATFORM_ERROR` under
   `Heap`/GC). This is a deliberate consequence of strategy decoupling
   (EDR-006): semantics are stable, classification follows the active
   Policy.
5. **Every system error has a defined diagnostic.** Machine-readable code,
   severity, location, and repair hint — consistent with the diagnostic
   contract of `COMPILER_AS_STATIC_ANALYZER.md` (EDR-030). No undefined
   behaviour, no silent corruption.
6. **LANGUAGE_ERROR must not occur.** A language defect is a spec defect,
   resolved in-process (EDR, gate), never a program-visible failure. A
   runtime that reports a language error is itself defective (an
   `EXECUTION_ERROR`).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Error Propagation Policy | System errors are not propagated as values; each class has defined termination semantics |
| Soundness Policy | Determines which conditions are *defined* (never UB) vs *forbidden* (LANGUAGE_ERROR) |
| Undefined Behaviour Policy | Every system-error condition has defined observable behaviour |
| Diagnostic Policy | Structure of system-error diagnostics (code, severity, location, repair hint) |
| Allocation Policy | Decides allocation-failure classification — recoverable domain error vs terminal `PLATFORM_ERROR` |
| Verification Layer Policy | COMPILE_ERROR is the ERROR severity of the seven verification layers (EDR-030) |

## Model (What)

### The domain/system boundary

```
representable as a value (Result<T,E> / !T)   →  DOMAIN ERROR   (handled in code)
not representable as a value                  →  SYSTEM ERROR   (defined termination)
```

The boundary is **representability in the program's type system**. Anything
the program can name as a value is recoverable by construction and belongs
in the domain model. Anything it cannot is a system error.

### The SYSTEM_ERROR family

`SYSTEM_ERROR` is the family name for all non-domain, unrecoverable
failures, classified by **fault owner** — whose contract was violated:

| Class | Fault owner | Condition | Example |
|---|---|---|---|
| **PROGRAM_ERROR** (ex-Panic) | program | program violated its own invariant — a bug | forced unwrap `!` on `None` (EDR-018/028), out-of-bounds access that escaped static analysis, unreachable state |
| **EXECUTION_ERROR** | runtime / strategy | the engine failed to honour the language contract | codegen bug, allocator/scheduler strategy failure, GC corruption |
| **PLATFORM_ERROR** | OS / hardware | the environment failed the program | stack overflow, hardware fault, signal-delivered termination, missing platform capability, true out-of-memory (unrecoverable `Heap`/GC exhaustion) |

### Allocation failure: classification follows the Allocation Policy

Memory exhaustion has no fixed class — it follows the active Allocation
Policy (EDR-034, EDR-006):

| Allocation Policy | Exhaustion predictability | Class | Mechanism |
|---|---|---|---|
| `Arena` (Default) | forecastable (fixed region) | **domain error** | `Result<_, AllocError>` / Error Union tag `error.OutOfMemory` |
| `Static` (Embedded) | forecastable / compile-time | **domain error** or impossible | fixed sizes; no runtime allocator |
| `Heap` (High-Performance) | not forecastable | **PLATFORM_ERROR** | defined abort + structured diagnostic |
| GC (High-Performance) | not forecastable | **PLATFORM_ERROR** | defined abort + structured diagnostic |
| `no_alloc` mode (OQ3 of `ALLOCATION.md`) | — | **impossible by construction** | any allocation is a compile error |

Recoverable exhaustion is a *domain error* — a value the program handles
(`error.OutOfMemory`), never an implicit abort. Unrecoverable exhaustion is
a *system error* classified as **PLATFORM_ERROR**: the fault owner is the
environment that cannot provide memory. Exhaustion is never undefined
behaviour (EDR-030) — every outcome is either a typed `Error` value or a
defined abort with a structured diagnostic (code, severity, location,
repair hint). The `ALLOCATION.md` implicitness tension remains open:
recoverable exhaustion requires a syntactically visible fallible point
(explicit allocator API returning `Result` vs a marked fallible scope).

**Alternative allocation-failure strategies considered:**

| Strategy | Description | Trade-off |
|---|---|---|
| **Always terminal** (Rust default) | OOM always aborts, even for arena/embedded | Simple, but discards recoverability exactly where exhaustion is forecastable |
| **Always recoverable** (Zig style) | Explicit allocator everywhere; every allocation can fail | Maximum control, but high ceremony and contradicts `ALLOCATION.md`'s implicit model |
| **Policy-dependent (proposed)** | Class follows the Allocation Policy | Matches Orthon's strategy decoupling; requires resolving the explicitness tension |
| **Silent / UB** | Exhaustion is undefined behaviour | Rejected: violates EDR-030, creates corruption |

### Distinct positions: not members of the runtime family

| Name | Position | Why it is separate |
|---|---|---|
| **COMPILE_ERROR** | Not a runtime failure at all | The normal outcome of the seven verification layers (EDR-030): the program failed to satisfy the language contract and never ran. It is a phase, not a system-error class |
| **LANGUAGE_ERROR** | Forbidden meta-state | The language spec/compiler is defective. Never a program-visible failure; resolved via the EDR/gate process. Its occurrence signals a spec bug, not a program bug |

### Who "handles" what

| Class | Program handles it? | Entity with a defined obligation |
|---|---|---|
| Domain error | Yes — `match`/combinators, compiler-enforced | program code |
| PROGRAM_ERROR | No — bug, fix the code | compiler/runtime produce the diagnostic; no recovery |
| EXECUTION_ERROR | No — impossible for a correct program to trigger | runtime produces the diagnostic; it is an implementation defect |
| PLATFORM_ERROR | No — nothing to run on | platform/runtime produce the diagnostic; termination (recoverable allocation failure is handled by the program as a domain error) |

## Default Strategy

Program bugs surface as **PROGRAM_ERROR** (bounds checks, forced unwrap).
Environment failures surface as **PLATFORM_ERROR** (including true
out-of-memory under `Heap`/GC). Runtime defects surface as
**EXECUTION_ERROR**. Memory exhaustion follows the active Allocation
Policy — recoverable as a domain error (`error.OutOfMemory`) under
`Arena`/`Static`, terminal as **PLATFORM_ERROR** under `Heap`/GC. Every
system error produces a structured diagnostic (code, severity, location,
repair hint); none are catchable.
COMPILE_ERROR is the ordinary ERROR severity of the static verification
pipeline. LANGUAGE_ERROR is tracked as a spec-quality defect, never
surfaced to a running program.

## Alternative Strategies

| Strategy | Description | Trade-off |
|---|---|---|
| **Catchable system exceptions** | `try-catch` around system errors | Rejected: hidden control flow, violates EDR-020's no-exceptions decision |
| **Flat "panic" with no taxonomy** | Current state | Insufficient: no diagnostic contract, no fault-owner attribution, muddles bug vs environment vs runtime |
| **Single catch-all `SystemError`** | One name for all terminal failures | Loses fault-owner attribution; weakens LLM diagnostics and process contracts |
| **Verified totality** | Dependent/refinement types prove absence of runtime system errors | Maximum safety but extreme annotation burden; rejects too many correct programs; rejected for cost |
| **Taxonomy as documentation only** | Classes exist in spec, no runtime contract | Fails the LLM-toolchain requirement (EDR-030): no deterministic machine-readable contract |

## Open Questions

1. Is `SYSTEM_ERROR` one documented family (in `what/EXECUTION_MODEL.md`)
   or a set of per-member concepts (`PROGRAM_ERROR.md`, etc.)?
2. Should **PROGRAM_ERROR** replace "Panic" as the term in
   `ERROR_HANDLING.md` (EDR-020) — i.e., the rename amendment? Does it need
   a concept of its own, or is a terminology update to the accepted concept
   sufficient?
3. Can a *correct* program ever observe **EXECUTION_ERROR**? If not, is it
   purely a developer/LLM-facing diagnostic with no program-visible
   semantics?
4. What is the **process-level contract** per class — exit codes, signals,
   and termination timing?
5. Should **LANGUAGE_ERROR** be tracked as a spec-quality metric (count of
   spec defects) for governance, mirroring `how/gates/`?
6. How does the taxonomy interact with the **Execution Program**
   (EDR-036) — should the Execution Descriptor declare which system-error
   classes a given deployment can surface?
7. Does the boundary (representability) hold for **FFI / plugin**
   boundaries, where a foreign failure is neither a typed domain error nor
   a defined Orthon system error?

### Allocation-failure questions (folded from the former SYSMEM_ERROR hypothesis)

1. Does the **implicit-allocation model** (`ALLOCATION.md`) allow a *visible*
   recoverable allocation, or is recoverability limited to an explicit
   allocator API returning `Result`?
2. Should the terminal allocation-failure diagnostic carry a **repair hint**
   (e.g., "increase arena size", "switch to `Static` policy", "adopt
   `no_alloc`")?
3. Does **`no_alloc` mode** (OQ3 of `ALLOCATION.md`) make allocation failure
   impossible by construction, and should that be a compile-time guarantee?
4. What is the **entrypoint's obligation** under the recoverable regime —
   must exhaustion of the top-level arena be handled at `entry:` (unhandled
   `Result` is a compile error per EDR-020)?
5. Under GC, can allocation failure ever be **recoverable** (e.g., a
   pre-abort compaction pass), or is it always terminal?
6. Should allocation failure carry **per-policy tags** (`error.ArenaExhausted`
   vs `error.OutOfMemory`) or a single name shared across policies?
7. How does allocation failure interact with **region inference**
   (`REGION_BASED_MEMORY_MANAGEMENT.md`) — does arena liveness analysis
   eliminate most exhaustion paths?

## Decision History

- **2026-08-23 — Graduated.** Accepted via [EDR-089](../../../decision_records/architecture/EDR-089-system-error-taxonomy.md)
  (Concept Design Review, 8 steps; Human Sign-off LOCKED). The full Decision
  History of the accepted concept is recorded in
  [`what/concepts/SYSTEM_ERROR.md`](../../../what/concepts/SYSTEM_ERROR.md)
  § Decision History. This research file is retained as provenance.
