# EDR-064: Span — Compiler-Recognized Memory View

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Memory Model)

---

### Context

How do you provide safe, non-owning access to a contiguous region of memory? A `Span<T>` is a lightweight view over a contiguous sequence — non-owning, lifetime-tracked, bounds-checked.

Span requires compiler support for:
1. **Lifetime tracking** — the view cannot outlive its backing storage.
2. **Bounds checking** — access through a span is checked.
3. **Slicing protocol** — slicing produces a span, not a copy.

These are compiler-level guarantees — a library type wrapping pointer+length cannot provide lifetime tracking or guarantees about bounds checking without compiler support.

---

### Decision

Span is a **Language** construct:
- `Span<T>` is a compiler-recognized memory view type.
- Slicing a collection produces a `Span` (reference, no copy).
- Span access is bounds-checked (debug and release-safe modes).
- Lifetime of a span is statically verified against its source.
- Mutable variant via `mut Span<T>`.
- Uniform construction from arrays, slices, and contiguous buffers.

---

### Consequences

- **Positive:**
  - Safe, non-owning memory views without copying.
  - Lifetime tracking prevents dangling views.
  - Bounds checking prevents out-of-bounds access.
- **Negative:**
  - Compiler must recognize `Span<T>` as a special type for lifetime tracking.
  - Bounds checking has runtime cost in debug mode.

---

### Compliance

- The compiler must verify span lifetime against source data lifetime.
- All span access must be bounds-checked in debug and release-safe modes.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Library-only span | Cannot provide compiler-level lifetime tracking or bounds checking guarantees. |
| Copying slice (Python style) | Creates unnecessary copies — defeats the purpose of a view. |
| Unsafe pointer+length | No safety guarantees — violates Orthon's design principles. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes slicing, gets safety without copies. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Lifetime rules are a direct application of ownership model. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept: non-owning view with lifetime. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Span composes with existing collection and slice patterns. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Span semantics independent of lifetime tracking mechanism. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood from Rust `&[T]` and C++ `std::span`. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Obvious semantics — LLMs correctly generate slicing expecting a view. |
