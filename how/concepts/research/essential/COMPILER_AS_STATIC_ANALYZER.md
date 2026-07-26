# Compiler as Static Analyzer

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How much correctness verification should the compiler perform — and where is the line between a compiler error, a compiler warning, and a linter concern?

Traditional compilers check syntax and types. Modern compilers go further:

- **Rust** — ownership, borrowing, lifetimes, Send/Sync, exhaustiveness, dead code, unused results.
- **Haskell (GHC)** — type-level proofs, totality checking, pattern exhaustiveness, termination checking (with extensions).
- **Zig** — comptime assertions, undefined-behaviour detection, overflow checking.
- **ATS / Liquid Haskell** — full dependent types and refinement types; the compiler proves theorems about program behaviour.

The core problem: **every static check is a trade-off between compile-time catch (earlier, cheaper) and expressiveness restriction (some valid programs will be rejected)**. A compiler that checks everything rejects programs that are correct but not provably correct; a compiler that checks nothing shifts the burden entirely to testing and production.

For LLM-native design, this trade-off is amplified: an LLM generates code that *looks* plausible but may contain subtle errors. A compiler that catches those errors without executing the code is the fastest feedback loop for an LLM (generate → compile → fix, no runtime needed).

## Principles

1. **Progressive verification layers** — The compiler applies checks in layers, from cheapest/least-restrictive to most-expensive/most-restrictive. Early layers fail fast; later layers catch deeper properties.

2. **No undefined behaviour** — The compiler rejects all programs where behaviour is not fully specified by the language semantics. No platform-defined traps, no unspecified evaluation order, no implicit conversions that lose information.

3. **Explicitness enables analysis** — Orthon's design principle of explicit semantics (effects, mutations, error handling) gives the compiler more information to verify. An explicit `proc fn` makes it trivial to check that no mutation occurs in a `fun fn`.

4. **Verification is not a separate tool** — Static analysis is built into the compiler pipeline, not relegated to an external linter. The `wvy check` command runs the full verification suite. External linters may add project-specific rules but cannot relax compiler checks.

5. **LLM-friendly error output** — Every diagnostic includes a machine-readable error code, the exact location, the inferred/expected type or state, and a repair hint. The Schema Provider exposes diagnostics for LLM consumption.

6. **Verified by default, opt-out for prototyping** — All checks are enabled by default. A `--fast` or `--relaxed` mode may skip expensive checks during early development, but code must pass all checks before release.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Verification Layer Policy | Defines the ordered layers of static checks and which are enabled by default vs. opt-in. |
| Soundness Policy | Determines whether the compiler is fully sound (all accepted programs are correct) or pragmatic (some false negatives accepted for expressiveness). |
| Undefined Behaviour Policy | Specifies which operations the compiler rejects vs. defines as implementation-defined vs. leaves undefined. |
| Effect Tracking Policy | Verification of declared effect boundaries (mutability, allocation, I/O, async). |
| Completeness Policy | Exhaustiveness checking for pattern matching; totality checking for recursive functions. |
| Diagnostic Policy | Structure and content of compiler diagnostics: human-readable message, machine-readable code, location, repair hint. |

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

Each layer that passes increases confidence in the program without executing it.

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

// Layer 4 failure — borrow violation
let data = [1, 2, 3]
let ref = &data[0]
data.push(4)     // ERROR: cannot borrow `data` as mutable while borrowed
// Code: E401 | Severity: ERROR | Note: immutable borrow at line 2

// Layer 5 failure — effect violation
fun fn compute() -> Int
    // This function declares no mutation
    state.counter += 1   // ERROR: mutation not allowed in `fun` context
// Code: E501 | Severity: ERROR | Note: declare as `proc` to allow mutation

// Layer 6 failure — non-exhaustive match
match option_value
    case Some(v) -> process(v)
    // ERROR: missing case `None` — `Option<T>` has two variants
// Code: E601 | Severity: ERROR | Repair: add `case None -> ...`
```

### Warnings vs. Errors

Not every check needs to be a hard error. The compiler distinguishes:

| Severity | Behaviour | Examples |
|---|---|---|
| **ERROR** | Compilation fails. No artifact produced. | Type error, unresolved symbol, borrow violation, unhandled error. |
| **WARNING** | Compilation succeeds. Warning displayed. | Unused variable, dead code, redundant cast, missing documentation. |
| **INFO** | Compilation succeeds. Info shown only on request (`wvy check --verbose`). | Inferred type materialisation, unreachable branch tolerance, minor style suggestions. |
| **SUPPRESSIBLE** | Warning that can be suppressed with explicit annotation (`@allow(unused)`). | Most warnings; never errors. |

### LLM-Friendliness

Every diagnostic includes structured fields for LLM consumption:

```json
{
    "code": "E401",
    "severity": "ERROR",
    "message": "cannot borrow `data` as mutable while borrowed",
    "location": {
        "file": "src/sort.orthon",
        "line": 8,
        "column": 5
    },
    "repair_hint": "Use `data.clone().push(4)` or restructure to end the borrow before mutation",
    "related_locations": [
        {"file": "src/sort.orthon", "line": 6, "note": "immutable borrow introduced here"}
    ]
}
```

The Schema Provider and Compiler Introspection API expose diagnostics in this structured form for programmable consumption.

## Default Strategy

Seven verification layers (Syntax → Name Resolution → Type Checking → Ownership → Effects → Exhaustiveness → Contracts), all enabled by default. Errors prevent compilation. Warnings are displayed but do not block builds. A `--relaxed` mode skips layers 6–7 for prototyping. Machine-readable diagnostic output is the default for CI and LLM toolchain consumption.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Minimal compiler, external linters** | C, Go | Fast compilation. Most verification delegated to tools that may not run in every workflow. Catches fewer errors before test. |
| **Full dependent types** | ATS, Coq, Liquid Haskell | Maximum verification: the compiler proves theorems about behaviour. Extremely high annotation burden; many correct programs rejected because proofs cannot be automated. |
| **Contract-only verification** | Eiffel, Ada/SPARK | Verification focused on pre/post/invariant contracts. Powerful for critical systems but leaves ownership and effect gaps. |
| **Sound type system only** | Haskell, OCaml | Type-level safety without ownership or effect tracking. Works well for pure languages but incomplete for systems programming. |
| **Pluggable linter ecosystem** | Rust (`clippy`), Python (`ruff`) | Rich ecosystem of additional checks outside the compiler. Flexible but requires separate toolchain awareness and CI configuration. |

## Open Questions

1. Should the compiler support user-defined static assertions beyond contracts (e.g., compile-time property tests)?
2. How should verification interact with the LLM generation loop — should the compiler provide a "diff patch" as a repair hint?
3. Should there be an `@unsafe` escape hatch that disables specific verification layers for a scope?
4. At what point should contract verification (Layer 7) become a hard error vs. a warning?
5. Should the compiler cache verification results across incremental compilations for faster feedback?
6. How do verification layers interact with different Implementation Strategies — do embedded targets require stricter verification?

## References

- [`TYPE_INFERENCE.md`](./TYPE_INFERENCE.md) — type checking layer details
- [`OWNERSHIP.md`](./OWNERSHIP.md) — ownership and borrowing verification
- [`PATTERN_MATCHING.md`](./PATTERN_MATCHING.md) — exhaustiveness checking
- [`ERROR_HANDLING.md`](./ERROR_HANDLING.md) — error handling completeness
- [`CONTRACTS.md`](../important/CONTRACTS.md) — contract verification layer
- [`NULL_SAFETY.md`](./NULL_SAFETY.md) — null safety checks
- [`COMPILE_TIME_CONCURRENCY_SAFETY.md`](../deferrable/COMPILE_TIME_CONCURRENCY_SAFETY.md) — Send/Sync-style verification
- [`IDENTITY_BASED_SAFETY.md`](./IDENTITY_BASED_SAFETY.md) — ownership checker as core safety mechanism
- [`EXCLUSIVE_DECLARATIONS.md`](./EXCLUSIVE_DECLARATIONS.md) — effect declaration and verification
- [`LLM_NATIVE_TOOLCHAIN.md`](../deferrable/LLM_NATIVE_TOOLCHAIN.md) — Static Analyser as toolchain component
