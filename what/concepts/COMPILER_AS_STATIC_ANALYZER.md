# Compiler as Static Analyzer

> **✅ ACCEPTED — [EDR-030](../how/decision_records/architecture/EDR-030-compiler-as-static-analyzer.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`GLOSSARY.md`](../GLOSSARY.md) § Verification Layer,
> [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md),
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md)

---

## Issue (Why)

How much correctness verification should the compiler perform — and where is the line between a compiler error, a compiler warning, and a linter concern?

Traditional compilers check syntax and types. Modern compilers go further — Rust checks ownership, borrowing, lifetimes, exhaustiveness; Zig checks comptime assertions and undefined-behaviour detection; Haskell (with extensions) checks totality and termination.

The core trade-off: **every static check is a trade-off between compile-time catch (earlier, cheaper) and expressiveness restriction (some valid programs will be rejected)**. A compiler that checks everything rejects programs that are correct but not provably correct; a compiler that checks nothing shifts the burden entirely to testing and production.

For LLM-native design, this trade-off is amplified: an LLM generates code that *looks* plausible but may contain subtle errors. A compiler that catches those errors without executing the code is the fastest feedback loop for an LLM (generate → compile → fix, no runtime needed).

Orthon's answer: **the compiler IS the static analyzer.** Verification is built into the compiler pipeline in progressive layers, not relegated to external tools. The `wvy check` command runs the full verification suite.

## Principles

1. **Progressive verification layers** — The compiler applies checks in layers, from cheapest/least-restrictive to most-expensive/most-restrictive. Early layers fail fast; later layers catch deeper properties.
2. **No undefined behaviour** — The compiler rejects all programs where behaviour is not fully specified by the language semantics.
3. **Explicitness enables analysis** — Orthon's explicit semantics (effects, mutations, error handling) give the compiler more information to verify.
4. **Verification is not a separate tool** — Static analysis is built into the compiler pipeline, not relegated to an external linter. External linters may add project-specific rules but cannot relax compiler checks.
5. **LLM-friendly error output** — Every diagnostic includes a machine-readable error code, exact location, inferred/expected type or state, and a repair hint.
6. **Verified by default, opt-out for prototyping** — All checks are enabled by default. A `--relaxed` mode may skip expensive checks during early development, but code must pass all checks before release.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Verification Layer Policy | Defines the ordered layers of static checks and which are enabled by default vs. opt-in |
| Soundness Policy | Determines whether the compiler is fully sound (all accepted programs are correct) or pragmatic (some false negatives accepted for expressiveness) |
| Undefined Behaviour Policy | Specifies which operations the compiler rejects vs. defines as implementation-defined vs. leaves undefined |
| Effect Tracking Policy | Verification of declared effect boundaries (mutability, allocation, I/O, async) |
| Completeness Policy | Exhaustiveness checking for pattern matching; totality checking for recursive functions |
| Diagnostic Policy | Structure and content of compiler diagnostics: human-readable message, machine-readable code, location, repair hint |

## Model (What)

### Layers of Verification

The compiler applies checks in a defined order. Each layer depends on the previous:

```
Layer 1: Syntax & Parsing
  ├── Well-formed tokens
  ├── Valid grammar
  └── No syntactic ambiguity

Layer 2: Name Resolution
  ├── All symbols resolve to a declaration
  ├── No duplicate definitions in scope
  ├── Visibility rules respected
  └── Cyclic dependencies detected (where illegal)

Layer 3: Type Checking
  ├── All expressions have valid types
  ├── Function arguments match parameter types
  ├── Generic instantiation is valid
  ├── Trait bounds are satisfied
  └── Type inference completes (no ambiguous types)

Layer 4: Ownership & Borrowing
  ├── Every value has exactly one owner
  ├── Borrows do not outlive their referent
  ├── No simultaneous mutable and immutable borrows
  ├── Moves of borrowed values are prevented
  └── Destructors run at correct scope

Layer 5: Effect Verification
  ├── Mutation only in `proc` functions
  ├── Allocation only where policy permits
  ├── I/O only where declared
  ├── Async operations only in `async` contexts
  └── Error handling is complete (no unhandled errors)

Layer 6: Exhaustiveness & Completeness
  ├── Pattern matches cover all variants
  ├── All enum variants are handled
  ├── All function return paths return a value
  └── `Option<T>` and `Result<T, E>` are handled

Layer 7: Contract Verification (optional)
  ├── Precondition checks (where feasible at compile time)
  ├── Postcondition validation (where trivially checkable)
  ├── Loop invariant consistency
  └── Type-level constraint satisfaction
```

### Error Model

```orthon
// Layer 1 failure — syntax error
let x = 42 +     // ERROR: incomplete expression, expected operand
// Code: E001 | Severity: ERROR | Repair: provide right operand

// Layer 2 failure — unresolved symbol
print(z)         // ERROR: `z` not declared in this scope
// Code: E201 | Severity: ERROR | Candidates: `x`, `y` in current scope

// Layer 3 failure — type mismatch
fn greet(name: String)
    print(name)  // OK

greet(42)        // ERROR: expected `String`, found `Int`
// Code: E301 | Severity: ERROR | Note: `Int` does not implement `Stringifiable`
```

### LLM-Friendly Diagnostics

Every diagnostic includes structured fields for LLM consumption:

```json
{
    "code": "E401",
    "severity": "ERROR",
    "message": "cannot borrow `data` as mutable while borrowed",
    "location": { "file": "src/sort.orthon", "line": 8, "column": 5 },
    "repair_hint": "Use `data.clone().push(4)` or restructure to end the borrow before mutation",
    "related_locations": [
        { "file": "src/sort.orthon", "line": 6, "note": "immutable borrow introduced here" }
    ]
}
```

## Default Strategy

Seven verification layers enabled by default. Errors prevent compilation. Warnings are displayed but do not block builds. A `--relaxed` mode skips layers 6–7 for prototyping. Machine-readable diagnostic output is the default for CI and LLM toolchain consumption.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Minimal compiler, external linters** | C, Go | Fast compilation but most verification delegated to optional tools |
| **Full dependent types** | ATS, Coq, Liquid Haskell | Maximum verification but extreme annotation burden |
| **Sound type system only** | Haskell, OCaml | Type-level safety without ownership or effect tracking |

## Open Questions

1. Should there be an `@unsafe` escape hatch that disables specific verification layers for a scope?
2. Should the compiler cache verification results across incremental compilations?
3. How do verification layers interact with different Implementation Strategies?

## Decision History

- **2026-07-27:** Accepted via EDR-030. Compiler IS the static analyzer — progressive verification layers built into the compiler pipeline. Guaranteed analyses and extension analyses distinguished. LLM-friendly diagnostic format adopted.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_STRATEGIES.md`
