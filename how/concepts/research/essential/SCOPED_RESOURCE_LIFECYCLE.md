# Scoped Resource Lifecycle

> **⚠️ SUPERSEDED — [EDR-085](../../../decision_records/architecture/EDR-085-execution-context-invocation.md) (Execution Context Invocation), 2026-08-06.**
> This hypothesis is superseded. Resource management is the same
> Invocation pattern as coroutines, actors, and threads: `using x = expr`
> is pure syntactic sugar over context + scope + deterministic destructor
> (desugars to `delegate(expr)` + block + scope-bound destruction), with
> no new semantics. Retained for historical reference (the four-model
> comparative analysis remains useful background).
>
> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-resource-management.md`
> **Last updated:** 2026-08-06 — superseded by EDR-085.

## Issue (Why)

Manually opening and closing files, connections, and handles forces the
programmer to write `try/finally`-style cleanup code that easily misses
`close()` in case of an exception, leading to resource leaks. Every
manual `close()` call is a potential leak site.

Orthon should eliminate resource leaks by providing a built-in scoped
resource management construct that guarantees release at scope exit.

### Historical context

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `f = open(...); try: ... finally: f.close()` | `with open(...) as f:` |
| Java | `br = null; try { ... } catch {...} finally { if(br != null) br.close(); }` | `try (BufferedReader br = ...)` |

## Principles

Which principles must not be violated? Reference: [`DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md).

1. **Explicitness** — the cleanup must be syntactically visible. Hidden control flow (magic destructors with no surface syntax) is acceptable only if the programmer explicitly opts into it.
2. **Deterministic Behavior** — cleanup must happen at a well-defined, predictable point. No GC-finalizer-style "eventually" semantics.
3. **Semantic Purity** — the cleanup construct must have exactly one meaning, independent of context.
4. **Orthogonality** — the cleanup mechanism must compose freely with error handling, ownership, concurrency, and all other language constructs. No special cases.
5. **LLM Generability** — an LLM must be able to reliably generate correct cleanup code. The construct must not introduce hidden state or non-local reasoning requirements.

## Four Models: Comparative Analysis

Four languages solve the scoped-resource-lifecycle problem in fundamentally different ways. This section compares them to inform Orthon's choice.

### Overview

| Aspect | D | Zig | Go | Rust |
|--------|---|-----|----|------|
| **Mechanism** | `scope(exit)` / `scope(success)` / `scope(failure)` | `defer` + `errdefer` | `defer` | RAII via `Drop` trait |
| **Number of variants** | 3 (exit, success, failure) | 2 (defer, errdefer) | 1 (defer) | 1 (Drop::drop) |
| **Granularity** | Per-block, explicit | Per-block, explicit | Per-block, explicit | Per-type, implicit |
| **Error-aware?** | Yes — `scope(success)` / `scope(failure)` distinguish | Yes — `errdefer` runs only on error | No — `defer` always runs | No — `Drop::drop` always runs |
| **Requires wrapper type?** | No — arbitrary code block | No — arbitrary code block | No — arbitrary code block | Yes — must implement `Drop` trait |
| **Ordering** | LIFO (last-in, first-out) | LIFO | LIFO | Field declaration order (reverse) |
| **Interaction with returns** | Runs after return value evaluation, before caller receives it | Runs after return value evaluation | Runs after return value evaluation, before caller receives it | Runs when value goes out of scope |
| **Interaction with moves** | N/A (GC-based, no move semantics) | N/A (manual memory, no compiler-enforced move) | N/A (GC-based, no move semantics) | `Drop` runs at move destination's end-of-scope, not at move source |

### D: `scope(exit)` / `scope(success)` / `scope(failure)`

D provides three scope-guard statements that execute code at scope exit:

```d
void processFile(string filename) {
    auto file = File(filename, "r");
    scope(exit) file.close();       // always runs
    scope(success) writeln("OK");   // runs only if no exception thrown
    scope(failure) stderr.writeln("FAILED: ", filename); // runs only on exception

    auto data = file.readln();
    // if readln throws, scope(failure) fires, then scope(exit) fires
    // file.close() is guaranteed
}
```

**Key characteristics:**
- **Three-variant model:** `scope(exit)` always, `scope(success)` on normal return, `scope(failure)` on exception.
- **LIFO ordering:** Multiple `scope` statements in the same block execute in reverse declaration order.
- **No wrapper type needed:** Any cleanup code can be written inline — no need to define a `Drop`-equivalent type.
- **GC coexistence:** D has GC, so `scope(exit)` covers non-memory resources (file handles, locks, sockets). Memory is GC-managed separately.
- **Syntax:** `scope(exit) statement;` or `scope(exit) { block; }`.

**Strengths:**
- Granular error-aware cleanup without type-system ceremony.
- `scope(failure)` enables cleanup-that-only-matters-on-error (e.g., logging, rollback) without wrapping logic in `catch` blocks.
- Reads naturally: the cleanup is declared near the acquisition, preserving locality.

**Weaknesses:**
- Requires programmer discipline — nothing forces you to write `scope(exit)` (unlike Rust's `Drop`, which the compiler can warn about).
- No ownership integration — D doesn't track resource ownership, so `scope(exit)` is a convention, not a safety guarantee.
- Three variants can be confusing: when does `scope(success)` fire if there's a `scope(failure)` that throws?

### Zig: `defer` + `errdefer`

Zig provides two scope-guard statements:

```zig
fn processFile(allocator: std.mem.Allocator, filename: []const u8) !void {
    var file = try std.fs.cwd().openFile(filename, .{});
    defer file.close();            // always runs
    errdefer std.log.err("FAILED: {s}", .{filename}); // runs only on error

    var buffer = try allocator.alloc(u8, 1024);
    defer allocator.free(buffer);  // always runs

    _ = try file.readAll(buffer);
    // on error: errdefer fires, then both defers fire (LIFO: free first, close second)
    // on success: both defers fire (LIFO), errdefer skipped
}
```

**Key characteristics:**
- **Two-variant model:** `defer` (always) and `errdefer` (only on error).
- **No `scope(success)` equivalent:** Zig lacks D's on-success variant. Success-only cleanup must be done with a boolean flag or by restructuring code.
- **Error-aware via error unions:** `errdefer` triggers when the function returns an error (`!void`), not an exception — Zig has no exceptions.
- **Manual memory:** `defer` is the primary RAII substitute in a language without `Drop` traits. Every allocation is paired with a `defer allocator.free(...)`.
- **LIFO ordering:** Same as D.

**Strengths:**
- Simpler than D's three-variant model — two variants cover most cases.
- Pairs naturally with Zig's explicit error handling (`try`, error unions).
- No type-system overhead — `defer` is a statement, not a trait.

**Weaknesses:**
- No success-only variant — D's `scope(success)` use cases require workarounds.
- Verbose: every allocation needs an explicit `defer` line. Compare Rust: `Drop` is automatic.
- No compiler enforcement — forgetting `defer` on an allocation is a leak, and the compiler won't catch it (unlike Rust's borrow checker).

### Go: `defer`

Go provides a single scope-guard statement:

```go
func processFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()  // always runs

    data, err := io.ReadAll(file)
    if err != nil {
        return err  // file.Close() still runs
    }
    // file.Close() runs here
    return nil
}
```

**Key characteristics:**
- **Single variant:** `defer` always runs. No error-aware variant.
- **Argument evaluation at `defer` site:** arguments to the deferred function are evaluated immediately, but the call is executed later. This is a common source of bugs.
- **LIFO ordering:** Same as D and Zig.
- **GC coexistence:** Like D, Go has GC, so `defer` is for non-memory resources.

**Strengths:**
- Simplest model — one keyword, one behavior. Easy to learn.
- Pairs well with Go's explicit error returns (`if err != nil`).

**Weaknesses:**
- No error-aware variant. Common pattern: use a separate error-handling path or a flag variable.
- Argument evaluation semantics are surprising: `defer fmt.Println(i)` captures `i` at the `defer` site, not at execution time.
- At scale, many `defer` calls obscure the happy path.

### Rust: RAII via `Drop` Trait

Rust provides no scope-guard statement at all. Cleanup is automatic via the `Drop` trait:

```rust
fn process_file(filename: &str) -> Result<(), io::Error> {
    let file = File::open(filename)?;  // File implements Drop
    let mut reader = BufReader::new(file);

    let mut data = String::new();
    reader.read_to_string(&mut data)?;
    // file is dropped here — Drop::drop() closes the handle
    // No defer, no scope(exit), no manual close()
    Ok(())
}
```

**Key characteristics:**
- **Implicit, type-driven:** Cleanup is defined on the type (`impl Drop for File`), not at the use site. The programmer never writes a `defer` line.
- **Compiler-enforced:** The borrow checker ensures resources are not used after drop. Forgetting to clean up is impossible — the compiler guarantees it.
- **Move-aware:** When a value is moved, `Drop` responsibility transfers. The old location is invalidated.
- **No error-awareness:** `Drop::drop()` takes `&mut self` and returns nothing — it cannot signal failure. Error-aware cleanup (e.g., flushing a buffer that might fail) must be done explicitly before drop.
- **Field declaration order:** Drop order is the reverse of struct field declaration order, not LIFO of acquisition.

**Strengths:**
- **Guaranteed by construction** — the compiler ensures cleanup. No discipline required.
- Zero boilerplate at use sites — acquisition implies cleanup.
- Composes with ownership — move semantics transfer cleanup responsibility.
- LLM-generable: an LLM doesn't need to remember `defer`; the type system enforces it.

**Weaknesses:**
- No error-aware cleanup — `Drop::drop()` is infallible. Flushing a file that might fail requires explicit `flush()` before drop.
- Requires a wrapper type for ad-hoc cleanup — if you want to run arbitrary code at scope exit, you must define a struct, implement `Drop`, and instantiate it. This is ceremony for one-off cleanup.
- No success/failure distinction — D's `scope(failure)` logging requires a separate mechanism.
- Drop order is structural (field order), not temporal (acquisition order). This can surprise: resources acquired later may be dropped before resources acquired earlier if they are in different struct fields.

## Model (What)

The hypothesis: Orthon can eliminate `try/finally`-style cleanup by providing a built-in scoped resource management construct (RAII via ownership, context-manager blocks, or `defer` statements) that guarantees release at scope exit.

### Key design tension

Two fundamentally different models are in tension:

| | RAII (`Drop` trait) | Explicit scope guard (`defer`/`scope(exit)`) |
|---|---|---|
| **Where cleanup is defined** | On the type (centralized) | At the use site (decentralized) |
| **Boilerplate at use site** | None — acquisition implies cleanup | One line per acquired resource |
| **Boilerplate for one-off cleanup** | Must define a wrapper type | Inline code block |
| **Error-awareness** | No — `drop()` is infallible | Yes — `scope(failure)` / `errdefer` |
| **Compiler enforcement** | Yes — borrow checker guarantees drop | No — programmer must remember |
| **Interaction with moves** | Ownership transfer carries drop responsibility | N/A (manual memory model) |

Neither model strictly dominates the other. The question is which to make **primary** in Orthon, and whether the other should exist as a secondary mechanism.

### Orthon-specific considerations

**Interaction with OWNERSHIP model:** Orthon's ownership model (see [`OWNERSHIP.md`](OWNERSHIP.md)) is Rust-style: single owner, move semantics, borrow checking. This naturally aligns with RAII — `Drop` is the cleanup mechanism for owned values. Adding `defer`/`scope(exit)` on top of RAII creates a dual-cleanup model where the programmer must understand both mechanisms and their interaction.

**Interaction with ERROR_HANDLING:** Orthon uses `Result<T, E>` and `!T` error unions (see EDR-020, EDR-023) — not exceptions. `scope(failure)` in D and `errdefer` in Zig are exception/error-aware. Orthon's equivalent would need to trigger on `Error` variant returns.

**LLM Generability Gate:**
- RAII: ✅ LLM-generable. The type system enforces cleanup. Generated code is correct by default.
- `defer`/`scope(exit)`: ✅ LLM-generable but ⚠️ requires the LLM to remember to write the `defer` line. Forgetting it is a silent leak, not a compile error.
- `scope(failure)`/`errdefer`: ⚠️ More complex LLM reasoning — the LLM must understand which cleanup is error-only.

## Default Strategy

**RAII via `Drop` trait as primary mechanism.** Ownership-based cleanup is compiler-enforced, zero-boilerplate at use sites, and aligns with Orthon's ownership model. This is the Rust model.

Resources (files, sockets, locks, allocators) implement `Drop`. The compiler guarantees `drop()` is called exactly once, at end of scope or move.

## Alternative Strategies

| Strategy | Description | When to Use | Trade-offs |
|----------|-------------|-------------|------------|
| **Explicit `defer` as secondary** | Orthon also provides `defer` for ad-hoc scope-level cleanup without defining a wrapper type | One-off cleanup that doesn't justify a `Drop` impl; error-aware cleanup (`errdefer`) | Dual model: programmer must understand both RAII and defer. Risk of using `defer` when `Drop` would be safer |
| **D-style three-variant scope guard** | `scope(exit)`, `scope(success)`, `scope(failure)` as built-in statements | Rich error-aware cleanup without wrapper types | Three variants increase cognitive load. `scope(success)` has unclear interaction with early returns |
| **Zig-style two-variant defer** | `defer` + `errdefer` only | Error-aware cleanup in `Result`/`!T` error model | Simpler than D, but no success-only variant |
| **RAII-only (Rust model)** | No explicit scope guard at all. All cleanup via `Drop` | Maximum safety, minimal surface area | Requires wrapper types for ad-hoc cleanup. No error-aware cleanup in `Drop` |

## Open Questions

- Should RAII (ownership-based) or explicit scope blocks (`defer`) be primary? Both?
- If both: what is the ordering when a scope has both `defer` blocks and `Drop` values? (Proposed: all `defer` blocks execute first in LIFO, then all `Drop` values in field order.)
- Should Orthon have an error-aware variant (`errdefer` / `scope(failure)`)? If so, how does it interact with `Result<T, E>` and `!T`?
- How to handle resources that outlive their scope intentionally (e.g., returning a file handle from a function)?
- Should `scope(success)` exist? D is the only language with this variant. Is the use case compelling enough?
- If `defer` exists, should arguments be evaluated at `defer` site (Go) or at execution time (D/Zig)?

## Decision History

- Initial hypothesis derived from `imperative-crutch-resource-management.md` — no decisions recorded.
- 2026-08-04: Four-model comparative analysis added (D, Zig, Go, Rust). The analysis surfaces the RAII-vs-explicit tension and the error-awareness question. No decision yet — this remains an open design question for Concept Design Review (Milestone 2).
- 2026-08-06: **Superseded by EDR-085.** `using` is pure sugar over context + scope + deterministic destructor; resource management is the same Invocation pattern as coroutines/actors/threads. The RAII-vs-explicit question is resolved: context-scoped deterministic destruction, with `using` as the declarative form.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/OWNERSHIP.md`
- [ ] `essential/EXECUTION_PROGRAM.md`
- [ ] `essential/ERROR_HANDLING.md`
- [ ] `what/GLOSSARY.md`
