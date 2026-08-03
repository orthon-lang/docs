# ABI and Calling Convention

> Machine-code-level contract between parts of a compiled program, and why it
> is one of the hardest topics for a language implementer.

---

## Overview

| Level | Contract | What it fixes |
|-------|----------|---------------|
| **API** | Source-code level | Function signatures, data types |
| **ABI** | Compiled-binary level | How bytes are laid out in memory and registers at call boundaries |

The **Calling Convention** is the part of the ABI that governs function calls:
where arguments live, where the return value lives, who cleans the stack, and
which registers a callee must preserve.

For a language developer (building a compiler or a JIT interpreter), this is a
hard, non-negotiable layer: the code generator must follow the target
platform's rules exactly, or the program crashes.

---

## What the Calling Convention specifies (in detail)

When a compiler emits machine code for a function call, it must obey the target
platform's rules:

- **Where arguments live.** The first N arguments pass through CPU registers
  (e.g. System V AMD64: `rdi, rsi, rdx, rcx, r8, r9`), the rest through the
  stack. In classic x86 (cdecl) *all* arguments pass through the stack.
- **Where the return value lives.** Usually `rax` (integers) or the `rdx:rax`
  pair (128-bit). For large structs, the compiler may implicitly pass a hidden
  pointer to the result.
- **Stack cleanup.**
  - **Caller cleanup (cdecl):** the caller removes the arguments after the
    call returns — required for variadic functions (`printf`).
  - **Callee cleanup (stdcall):** the callee cleans its own stack — more
    compact, faster; used by the WinAPI.
- **Caller-saved vs callee-saved registers.** The ABI states which registers a
  function must preserve across the call and which it may clobber freely.
  Critical for the compiler's optimizer.
- **Stack unwinding.** How the stack frame is laid out so the system can walk
  the call stack on exception (`throw`) or during debugging.

---

## Why a language developer needs to know this

### A. Code generation (backend)

The compiler must emit `mov` / `push` instructions in a strict order. Put the
5th argument in a register where the platform expects it on the stack, and the
program dies with `Segmentation Fault`. Forget to clean the stack, and
everything breaks.

### B. Foreign Function Interface (FFI) — calling C libraries

Modern languages (Python, Rust, Zig, Go) actively call dynamic libraries
(`.dll`, `.so`). To call `printf` from C, the language must generate exactly
the instruction sequence and stack alignment the C compiler expects on that OS.
If the convention does not match the system one (SysV or MSVC), data is read
from the wrong place.

### C. Linking and compatibility

When a compiler emits object files (`.o`), the linker merges them with code
written in C++ or Rust. For this to work, function names must be **mangled**
according to the ABI — otherwise the linker cannot find the function.

### D. Garbage collection and exceptions

The ABI describes how to find GC roots on the stack. A JIT compiler needs to
know where callee-saved register values are so it can scan them for pointers
into the heap.

---

## Main pain points

- **Hard platform binding.** The ABIs for Linux x86_64, Windows x64, and
  ARM64 (AArch64) differ drastically:
  - **System V (Linux/macOS):** integer and float arguments use *different*
    register sets.
  - **Windows x64:** strictly 4 arguments in `RCX, RDX, R8, R9` (different
    order!) and a 32-byte **shadow space** must be reserved on the stack
    before the call, even with fewer arguments. If the compiler does not do
    this, Windows crashes the process.
- **ABI stability over time.** A language cannot change the ABI of public
  (exported) functions between versions without breaking every dynamic library
  written in that language. Like changing the physical shape of a power outlet
  — old plugs no longer fit.

---

## Modern trends

Many modern languages (Rust, Swift, Zig) let the developer attach attributes
that change the convention for a specific function:

```rust
#[link(name = "c")]
extern "C" fn foo() { ... }        // Use the system C ABI

extern "stdcall" fn bar() { ... }  // For Windows
```

Some languages (e.g. Go, early Nim) use their **own ABI** (not C-compatible)
for internal calls — fewer callee-saved registers, simpler stack, faster
goroutines. The price: when calling system libraries they must build a
trampoline/adaptor shim.

---

## Takeaway

For a language developer, ABI and Calling Convention are the **"traffic rules"
at the assembly level**. They cannot be ignored — otherwise the program cannot
talk to the OS, call `malloc`, or even run `main`. This is the low-level layer
where compiler abstraction meets physical hardware and a specific OS.

---

## Relevance to Orthon

- Orthon is currently a spec-only repository (implementation is a future,
  separate repo — Milestone 10). This note captures the backend-knowledge
  surface that the future compiler must design against.
- Any Orthon FFI story (the language calls C/OS libraries, or C calls into
  Orthon) will have to commit to a concrete platform ABI per target
  (SysV vs Windows x64 vs AArch64) and record it as an Engineering Decision
  Record.
- The Execution Program model (semantics decoupled from execution strategy)
  implies that calling-convention decisions live in the implementation
  strategy layer (`how/strategies/`), not in the Core Language semantics.
