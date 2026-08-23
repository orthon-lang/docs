# EDR-089: System Error Taxonomy — Domain/System Boundary and Fault-Owner Classification

**Status:** Accepted

**Date:** 2026-08-23

**Category:** Architecture

**Scope:** Subsystem (Error Handling & Execution Semantics)

---

### Context

Orthon's error model — `Result<T, E>` (EDR-020) and the Error Union `!T`
(EDR-023) — defines what happens for **expected, recoverable** failures:
they are values, declared in the function contract, and the compiler
enforces handling. The model is silent about the complement: what happens
when a failure is **not representable as a value** — when the program
violates its own invariant, when the runtime fails to honour the language
contract, or when the platform cannot provide what the program needs.

Today the vocabulary is incomplete and inconsistent:

- **Panic** exists only as an unnamed alternative strategy inside
  `ERROR_HANDLING.md` — an escape hatch with no diagnostic contract and no
  definition of *whose* fault it is.
- **Memory exhaustion** is unspecified entirely — `ALLOCATION.md` (EDR-034)
  defines the allocation *mechanism* but not the *behaviour on exhaustion*.
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

The Decision Pipeline returned **ACCEPT** as a **Language** decision
(D-03). The allocation-failure classification is a Policy sub-rule (D-04)
inside the concept. The Concept Design Review (8 steps) completed
2026-08-23; Step 7 (Convergence Check) passed with explicit Human
Sign-off.

**Research & pipeline trail:**
[`SYSTEM_ERROR.md`](../../concepts/research/essential/SYSTEM_ERROR.md)
(research; allocation-failure rule folded from the former SYSMEM_ERROR
hypothesis 2026-08-23);
[`DECISION_LOG.md`](../../gates/DECISION_LOG.md) § Entry: SYSTEM_ERROR
(Decision Pipeline, per-question reasoning).

---

### Decision

1. **Boundary rule.** A failure is a *domain error* if and only if it is
   representable in the program's type system as declared fallibility
   (`Result<T, E>` / `!T`). Any failure that is not representable as a
   value is a *system error*: its contract is defined diagnostic + defined
   termination, never recovery. The predicate is binary and decidable at
   the type level.

2. **The SYSTEM_ERROR family, classified by fault owner.** Each class
   names the layer whose contract was violated, mapping directly onto the
   Execution Environment (Compiler · Runtime · Platform):

   | Class | Fault owner | Example |
   |---|---|---|
   | **PROGRAM_ERROR** (ex-Panic) | program | forced unwrap `!` on `None` (EDR-018/028), out-of-bounds access that escaped static analysis, unreachable state |
   | **EXECUTION_ERROR** | runtime / strategy | codegen bug, allocator/scheduler strategy failure, GC corruption |
   | **PLATFORM_ERROR** | OS / hardware | stack overflow, hardware fault, signal-delivered termination, missing platform capability, true out-of-memory under `Heap`/GC |

   One owner, one class. No `try-catch` for system errors — recovery is
   impossible by construction (EDR-020, EDR-023).

3. **Diagnostic contract.** Every system error carries a machine-readable
   diagnostic — code, severity, location, repair hint — consistent with the
   diagnostic contract of `COMPILER_AS_STATIC_ANALYZER.md` (EDR-030). No
   undefined behaviour, no silent corruption.

4. **Classification follows policy, not language.** The same nominal
   condition may be a domain error under one Implementation Strategy and a
   system error under another. Allocation failure: recoverable domain
   error (`error.OutOfMemory`) under `Arena`/`Static`, terminal
   `PLATFORM_ERROR` under `Heap`/GC, impossible by construction under
   `no_alloc`. Never UB (EDR-030).

5. **COMPILE_ERROR and LANGUAGE_ERROR are not runtime family members.**
   `COMPILE_ERROR` is the ordinary ERROR severity of the seven static
   verification layers (EDR-030) — a phase, not a system-error class.
   `LANGUAGE_ERROR` is a forbidden meta-state: a spec defect resolved
   in-process (EDR/gate), never surfaced to a running program.

6. **No core expansion.** This is a naming + contract layer over the
   existing error model. No new primitive, no new syntax, no Core change.

---

### Consequences

- **Positive:**
  - Deterministic terminal contract for the LLM toolchain and the process
    (EDR-030 machine-readable diagnostics for every failure).
  - Programmer reasoning separates bug, environment, and runtime defect —
    three causes that demand three different responses.
  - `Panic` receives a home (`PROGRAM_ERROR`) with a defined diagnostic
    contract and fault-owner attribution.
  - Strengthens Minimal Core (no new primitive) and Explicitness (the
    undefined complement becomes a defined, explicit contract).
  - The allocation-failure rule closes the unspecified memory-exhaustion
    gap without introducing a MEMORY_ERROR class.
- **Negative:**
  - The `Panic` → `PROGRAM_ERROR` rename is a sequenced amendment to
    accepted documents (`ERROR_HANDLING.md`, EDR-020) requiring its own
    Human Sign-off (AGENTS.md §7.4).
  - Adds vocabulary a programmer must learn (three class names).
  - Process-level specifics (exit codes, signals, termination timing) and
    the FFI/plugin application of the boundary are deferred as open
    questions (see `SYSTEM_ERROR.md` Open Questions).

---

### Compliance

The decision is followed when:

- Every failure not representable as a value names exactly one class
  (`PROGRAM_ERROR`, `EXECUTION_ERROR`, `PLATFORM_ERROR`).
- No `try-catch` exists for system errors; recovery is never attempted for
  a non-representable failure.
- Every system error produces a structured diagnostic (code, severity,
  location, repair hint) per EDR-030.
- Allocation-failure classification follows the active Allocation Policy
  and is never undefined behaviour.
- `LANGUAGE_ERROR` never appears in a running program; `COMPILE_ERROR`
  remains the output of the static verification pipeline.

---

### Alternatives Considered

> Carried verbatim from Concept Design Review Step 2 (Alternatives).

Candidate set (canonical pipeline exit paths + one wild candidate):

| # | Candidate | Description |
|---|-----------|-------------|
| A | **Language construct** | System-error taxonomy as language-level semantic contract: named classes by fault owner, boundary rule (representable-as-value → domain; not → system), defined diagnostic + defined termination per class. Allocation-failure classification is a Policy sub-rule (D-04) inside the concept. |
| B | **Syntactic sugar** | Desugar system-error handling to existing primitives — e.g., `throw`-like sugar mapped to `Result`. |
| C | **Library / composition** | Express the taxonomy as a standard-library convention — a `SystemError` enum + helpers, no new language semantics. |
| D | **Do nothing** | Accept the current gaps: `Panic` stays an unnamed escape hatch, memory exhaustion unspecified, runtime/platform defects unnamed. |
| E | **Wild — single catch-all `SystemError` class** | One undifferentiated system-error class instead of a family named by fault owner. |

Scoring against the fixed validation-gate criteria (Design Principles,
minimality, orthogonality, LLM Generability Gate) plus two
problem-specific criteria (deterministic terminal contract; "whose
contract was violated"):

| Criterion | A — Language | B — Sugar | C — Library | D — Do nothing | E — Catch-all |
|---|---|---|---|---|---|
| **Solves the stated problem** (boundary + named complement) | ✅ | ❌ sugar renames nothing — the complement has no existing primitive to desugar to | ❌ a library cannot define runtime/platform termination, process contract, or compiler-enforced boundary | ❌ gaps remain; v0.1 not self-consistent (pipeline Q10) | ⚠️ names the complement but answers no *whose contract* — re-creates the conflation |
| **Design Principles** | ✅ strengthens Minimal Core (no new primitive; names existing layers) | ⚠️ violates Explicit Semantics if it implies hidden sugar | ✅ | ❌ violates Explicit Semantics / Defined Diagnostics by leaving undefined behaviour unnamed | ❌ violates Orthogonality — one class conflates three distinct owners |
| **Minimality** | ✅ no new primitive; classes map onto existing Execution Environment layers | ✅ but vacuous | ✅ | ✅ zero cost | ✅ zero new vocabulary |
| **Orthogonality** | ✅ composes with `Result`/`!T` (boundary is complementary, not overlapping) | ✅ | ⚠️ overlapping with language semantics; boundary is not enforceable from a library | ✅ | ❌ violates one-owner-one-class |
| **LLM Generability Gate** (EDR-030: machine-readable diagnostics for every failure) | ✅ deterministic per-class diagnostic + termination contract | ❌ | ⚠️ convention is not compiler-enforced; LLM cannot rely on it | ❌ no contract at all | ⚠️ ambiguous repair hint — LLM cannot distinguish bug/environment/runtime |
| **Answers "whose contract was violated"** | ✅ class names the owner | ❌ | ❌ | ❌ | ❌ |

**Selection: A — Language construct**, with the allocation-failure sub-rule
at Policy level (D-04). B is rejected because there is nothing to desugar —
the problem is semantic, not syntactic; C because the boundary and
termination semantics are outside a library's reach; D because v0.1 is not
self-consistent without the taxonomy (pipeline Q10); E because a catch-all
class defeats the purpose — it names the class but not whose contract was
violated.

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | Solves a real programmer need: undefined complement, LLM/process terminal contract, cause separation |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | Boundary is a decidable binary predicate; family names existing layers; no contradiction with EDR-020/EDR-023 |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | No new primitive; three classes; parsimonious (fewest entities for the diagnostic power) |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Strengthens Minimal Core and Explicitness; no closed principle violated (Principle Check, Step 4) |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass | Classification follows the active Strategy (EDR-006); semantics stable across implementations |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | One documented family; deferred refinements (process specifics, FFI) do not alter fundamental semantics |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Deterministic per-class diagnostic + termination contract; LLM can rely on the shape (EDR-030) |

**Gates not applied:** none — all seven gates apply and pass.

**Detailed reasoning:** See `DECISION_LOG.md` § Entry: SYSTEM_ERROR (Decision
Pipeline) and § Entry: SYSTEM_ERROR → EDR-089 (Concept Design Review) for
the per-gate reasoning trail.

---

> **Human Sign-off:** `Reviewed-by: mniedre · Date: 2026-08-23 · Verdict: LOCKED`
> Required before the EDR is finalized (AGENTS.md §7.4 — non-automatable).
> Only the solo author's explicit confirmation counts.
