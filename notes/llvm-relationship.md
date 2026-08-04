# LLVM and Orthon — Relationship Note

> **Note:** Exploratory analysis from 2026-08-04 conversation.
> Captures how LLVM relates to Orthon: a potential code-generation
> backend and execution target, not part of the language semantics.
> Records the ABI nuance — LLVM implements ABI mechanics but does not
> own the ABI.
>
> **Status:** Exploratory — no design decision made. Action at M4
> (Implementation), where a concrete backend is selected.
>
> **See also:**
> [`../what/EXECUTION_MODEL.md`](../what/EXECUTION_MODEL.md) (§ Execution Targets),
> [`../how/architecture/ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md) (§ Compiler),
> [`../how/architecture/IR.md`](../how/architecture/IR.md),
> [`../when/ROADMAP.md`](../when/ROADMAP.md) (§ M4, § Phase 7),
> [`./abi-and-calling-convention.md`](./abi-and-calling-convention.md),
> [`./c-abi-interop-consideration.md`](./c-abi-interop-consideration.md)

---

## Where LLVM Sits in the Orthon Stack

LLVM is not a language feature — it belongs to the **Execution
Environment**, at the Compiler/Platform layer:

```
Core Language (semantics, implementation-independent)
        ↓
Implementation Strategy (Policies: Allocation, Evaluation, ...)
        ↓
Execution Environment
   ├── Compiler  ← LLVM (potential backend)
   ├── Runtime
   └── Platform  (OS, WASM runtime, bare metal)
```

In the terms of [`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md),
the compiler translates Core semantics into a concrete execution form;
LLVM is the mechanism that lowers the typed IR to machine code for a
wide range of CPUs. Analogous to Rust/Swift/Zig: the language semantics
are defined independently, and LLVM is the lowering mechanism.

## What the Spec Says Today

- [`EXECUTION_MODEL.md`](../what/EXECUTION_MODEL.md) § Execution Targets
  lists supported execution environments: Interpreter, AOT, JIT, WASM,
  Container.
- [`ROADMAP.md`](../when/ROADMAP.md) § Phase 7 names the full target set
  explicitly: Interpreter, AOT, JIT, LLVM, WASM, Container.
- In practice this means AOT/JIT native compilation would most likely
  go through LLVM, but the spec does not mandate it.

## Why the Spec Is Not Bound to LLVM

1. **Implementation independence.** IR Invariant #1
   ([`IR.md`](../how/architecture/IR.md)): the IR must not reference any
   Policy value or strategy-specific concept. A backend choice is a
   transformation *over* the IR, not baked into it.
2. **Execution Program.** Semantics are decoupled from execution
   strategy — the same artifact can be interpreted, compiled,
   containerized, or built for WASM. LLVM is one point in that graph,
   not its definition.
3. **Specification ≠ implementation.** This repository is
   documentation-only; implementation lives in a separate repository
   (M10). Concrete backend selection happens there.

## ABI Nuance: LLVM Implements, Platform Defines

The relationship between LLVM and the ABI is frequently misstated. The
correct framing is layered:

| Layer | Who defines | Examples |
|-------|-------------|----------|
| ABI definition | Platform / OS / hardware | System V AMD64, Windows x64 (MSVC), ARM AAPCS, Itanium C++ ABI |
| ABI interpretation for a language | Frontend (Clang) | `TargetInfo` in Clang: C/C++ ABI rules, mangling, exceptions |
| ABI implementation in machine code | LLVM backend | argument passing, stack alignment, unwind info |

What the LLVM backend actually implements:

- **Calling conventions** — where arguments and return values live,
  caller/callee-saved registers, stack cleanup. Machinery:
  `CallingConvLower` / `CCAssignFn` + `TargetLowering::LowerFormalArguments`
  / `LowerCall` / `LowerReturn`. x86 has two paths: SysV and MSVC.
- **DataLayout** — pointer sizes and alignment, ABI/pref alignment,
  expressed as the module `DataLayout` string.
- **Stack alignment and frame layout** — including pre-call alignment
  rules (SysV requires 16-byte alignment).
- **Stack unwinding** — frame layout so the stack can be walked for
  exceptions and debugging (Itanium / MSVC personality, `.eh_frame`).
- **Partially name mangling and exceptions** — mostly frontend
  (Clang); LLVM supplies the machinery (personality functions).

What the language must still decide (LLVM does not decide it):

- **Struct layout / representation** — LLVM provides `DataLayout`, but
  the choice of representation (field order, `repr`) is the language's.
- **Name mangling** — own scheme or adopting the C scheme.
- **Own ABI vs C ABI** — a language may supply custom `CCAssignFn` rules
  for its own calling convention, but that creates a one-way FFI
  "island" (see [`c-abi-interop-consideration.md`](./c-abi-interop-consideration.md)).

## Implications for Orthon

- The Orthon specification is **ABI-agnostic** — ABI lives at the
  Implementation Strategy / tooling level, not in the semantic core.
- The C ABI consideration proposes the **platform C ABI as the native
  ABI**. If the implementation uses LLVM, the mechanics (argument
  passing, alignment, unwinding) come correct "out of the box" — but
  Orthon must still explicitly fix its data layout and mangling as part
  of its strategy.

## Open Questions

1. Should LLVM be named explicitly as a target in
   `EXECUTION_MODEL.md`, or kept implicit under AOT/JIT?
2. ABI policy at M4: adopt the platform C ABI as native, or define an
   internal ABI with an adaptor/trampoline shim?
3. IR interop: does IR need SSA form or serialization to lower cleanly
   into LLVM IR (see `IR.md` open questions)?
4. Does the LLM-native toolchain profile
   ([`LLM_STRATEGY.md`](../how/strategies/LLM_STRATEGY.md)) constrain the
   backend choice, or is backend selection orthogonal?
