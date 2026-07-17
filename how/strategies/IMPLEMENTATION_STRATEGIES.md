# Implementation Strategies

> This document maps each separable language concern to its possible
> strategy implementations. New strategies can be added as they emerge
> without changing the architecture document.

## Scope of the Pattern

The following table catalogues every mechanism where the
interface/implementation separation (defined in
[`ARCHITECTURE.md`](../architecture/ARCHITECTURE.md#scope-of-the-pattern))
applies, and lists known concrete strategies for each.

| Aspect | Interface (Language / StdLib) | Implementation (Strategy) |
|---|---|---|
| **Memory allocation** | Declarative allocation semantics | Heap, arena, linear, or GC-backed allocator |
| **Expression evaluation** | Expression semantics and composition | AST walker, bytecode VM, JIT, or interpreter |
| **Algorithm selection** | What the program expresses | Sorting, searching, hashing strategy implementations |
| **Object representation** | Data model and structural contracts | Struct-of-fields, SOA, tagged unions, or pointer-based layout |
| **Concurrency model** | Task and communication semantics | Thread pool, work-stealing, event loop, or hardware dispatch |
| **Error handling** | Result and error propagation model | Stack unwinding, error codes, or return-value branching |