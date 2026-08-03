# Voldemort Types (Opaque Return Types)

> **⚠️ DRAFT — This document is a preliminary research analysis.**
> It has not passed Concept Design Review.
>
> **Source:** D language idiom analysis — "Voldemort types" are types
> declared inside a function, returned via `auto`, whose concrete name
> is invisible to callers.
>
> **Last updated:** 2026-08-04

## Issue (Why)

How does a function return a value of a complex, locally-defined,
or chain-of-iterators type without exposing its internal structure
in the function signature?

Consider a function that returns a filtered, mapped iterator chain:

```orthon
// Without opaque return types: the programmer must either:
// Option A: Expose the full concrete type (brittle, leaks internals)
fn search(query: String) -> Filter<Map<Iter<[Item]>, fn(Item) -> bool>, fn(Item) -> Item>
    items.iter().map(transform).filter(predicate)

// Option B: Box into dyn Trait (runtime overhead)
fn search(query: String) -> dyn Iterator[Item]
    items.iter().map(transform).filter(predicate).collect()  // heap allocation
```

**Option A** breaks encapsulation — changing the internal implementation
changes the public API signature. **Option B** introduces runtime overhead
(vtable dispatch, heap allocation) for a purely compile-time concern.

The core problem: **type-level encapsulation at API boundaries is sacrificed
to performance, or performance is sacrificed to encapsulation.** There is
no way to say "this function returns *some type* that implements `Iterator[Item]`,
but the concrete type is an implementation detail."

This is the problem that D's Voldemort types, Rust's `impl Trait`, C++'s
`auto` return, and Swift's `some View` all solve in different ways.

### Real-world scenarios

| Scenario | Without opaque types | With opaque types |
|----------|---------------------|-------------------|
| Iterator chains | Expose `Filter<Map<Iter<...>>>` in signature | Return `impl Iterator[Item]` |
| Builder patterns | Expose internal builder state type | Return `impl Builder` |
| Factory functions | Expose concrete implementation type | Return `impl Service` |
| Async combinators | Expose `AndThen<Map<Future<...>>>` in signature | Return `impl Future[Output = T]` |
| Proxy/delegate wrappers | Expose wrapper struct type | Return `impl Delegate` |

## Principles

Which principles must not be violated? Reference: [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md).

1. **Explicitness** — "The meaning of code should be apparent from its surface form." A signature `fn foo() -> auto` reveals nothing about the return type. A signature `fn foo() -> impl Iterator[Item]` reveals the trait contract but hides the concrete type. Which level of explicitness is sufficient?
2. **Declarative With Static Guarantees** — the compiler must still verify that the returned type satisfies the declared interface. Opaque types are not dynamic types — they are statically known to the compiler, just hidden from the caller.
3. **Orthogonality** — opaque return types must compose with generics, traits, error handling, and all other language constructs without special cases.
4. **LLM Generability** — can an LLM generate correct code using `impl Trait` return types? The LLM knows the trait contract (from the signature), knows the concrete type (from the function body it generates), and can verify the match. This is LLM-generable with proper tooling.
5. **Transparency** — "the compiler must not hide optimization decisions that affect performance." An opaque return type should not introduce hidden allocations or dynamic dispatch — it is a zero-cost abstraction (monomorphised at compile time).

## Five Models: Comparative Analysis

### Overview

| Aspect | D (Voldemort) | Rust (`impl Trait`) | C++ (`auto`) | Swift (`some`) | Kotlin |
|--------|---------------|---------------------|---------------|----------------|--------|
| **Mechanism** | `auto` return + locally-defined struct | `impl Trait` in return position | `auto` return type deduction | `some Protocol` opaque result type | No direct equivalent |
| **Constrained?** | No — `auto` is unconstrained | Yes — `impl Trait` constrains to a trait | No — `auto` is unconstrained | Yes — `some Protocol` constrains to a protocol | N/A |
| **Caller knows trait?** | No — only via UFCS on returned methods | Yes — trait is visible in signature | No — caller must inspect body or rely on `decltype` | Yes — protocol is visible in signature | N/A |
| **Zero-cost?** | Yes — monomorphised, no allocation | Yes — monomorphised (return-position `impl Trait`) | Yes — monomorphised | Yes — compiler-resolved | N/A |
| **Multiple return types?** | No — single concrete type per call | No — single concrete type per call | No — single concrete type per call | No — single concrete type per call | N/A |
| **LLM-generable?** | ⚠️ LLM can't determine type from `auto` alone | ✅ LLM knows trait contract from signature | ⚠️ same as D | ✅ LLM knows protocol from signature | N/A |

### D: Voldemort Types

Named after the Harry Potter villain because "the type that must not be named" — the type exists, has a name, but callers cannot utter it.

```d
auto makeCounter(int start) {
    struct Counter {
        int value;
        int next() { return value++; }
    }
    return Counter(start);
}

void main() {
    auto c = makeCounter(10);
    writeln(c.next());  // 10
    writeln(c.next());  // 11
    // c's type is makeCounter.Counter — but we can't write it in source
}
```

**Key characteristics:**
- **Locally-defined struct returned via `auto`:** The struct is defined inside the function body. The return type is deduced by the compiler.
- **No trait constraint:** Callers interact with the returned value through UFCS or duck-typing. There is no declared interface — if the caller calls `.next()`, the compiler checks that the concrete type has `.next()`.
- **Full type information at compile time:** The compiler knows the concrete type; it just hides the name from the programmer.
- **Monomorphised:** No vtable, no heap allocation — the compiler generates code as if the type were named.

**Strengths:**
- Maximum encapsulation — the concrete type is truly hidden.
- Zero syntactic overhead — `auto` is one word.
- Composes with D's template metaprogramming — Voldemort types can be used in generic code.

**Weaknesses:**
- **Unconstrained:** The signature `auto makeCounter(int)` tells the caller nothing about what methods the returned type supports. Tooling and documentation must inspect the function body.
- **LLM-unfriendly:** An LLM generating code that calls `makeCounter` would need to read the function body to know what methods are available.
- **No API contract:** Changing the returned type's interface is a silent breaking change — no trait bounds to check.

### Rust: `impl Trait`

Rust constrains opaque return types with a trait bound:

```rust
fn make_counter(start: i32) -> impl Iterator<Item = i32> {
    // The concrete type is an anonymous struct generated by the compiler
    // from the iterator chain. Callers only see `impl Iterator<Item = i32>`.
    (start..).into_iter()
}

fn main() {
    let mut c = make_counter(10);
    println!("{}", c.next().unwrap());  // 10
    println!("{}", c.next().unwrap());  // 11
}
```

**Key characteristics:**
- **Trait-constrained:** `impl Trait` in return position guarantees the returned type implements the specified trait(s).
- **Single concrete type:** All return paths must return the same concrete type. The compiler verifies this.
- **Zero-cost:** Monomorphised — the compiler generates code for the concrete type. No heap allocation, no vtable.
- **Not dynamically dispatched:** Unlike `Box<dyn Trait>`, `impl Trait` is static dispatch. The caller's code is monomorphised against the concrete type.

**Strengths:**
- **API contract preserved:** The trait bound is visible in the signature. Tooling and documentation can show the available methods.
- **LLM-generable:** An LLM generating calling code sees `impl Iterator<Item = i32>` and knows the available methods without reading the function body.
- **Zero-cost:** No allocation, no dynamic dispatch — purely a type-system abstraction.
- **Compiler-verified:** The compiler checks that the returned type actually implements the declared trait.

**Weaknesses:**
- **Single concrete type:** Cannot return different concrete types from different branches (e.g., `if condition { TypeA } else { TypeB }`). Requires `Box<dyn Trait>` or `enum` for heterogeneous returns.
- **Trait bound required:** You cannot return a completely unconstrained opaque type. There must be a declared trait.
- **Limited trait composability:** `impl TraitA + TraitB` is supported, but the syntax can become unwieldy for complex bounds.

### C++: `auto` Return Type

C++14 introduced `auto` return type deduction:

```cpp
auto makeCounter(int start) {
    struct Counter {
        int value;
        int next() { return value++; }
    };
    return Counter{start};
}

int main() {
    auto c = makeCounter(10);
    std::cout << c.next();  // 10
}
```

**Key characteristics:**
- **Unconstrained, like D:** `auto` reveals nothing about the interface.
- **`decltype` for callers:** Callers can use `decltype(makeCounter(10))` to name the type, but this is fragile.
- **Trailing return type alternative:** `auto foo() -> SomeType` is a different feature (explicit return type, just with `auto` as a placeholder).

**Strengths:**
- Simple syntax — `auto` is already used for variable type deduction.
- Works with lambdas (which have unnameable types) — the primary motivation.

**Weaknesses:**
- Same LLM-unfriendliness as D — no trait constraint visible.
- `decltype` is an escape hatch, not a first-class feature.

### Swift: `some Protocol`

Swift's `some` keyword provides constrained opaque result types:

```swift
func makeCounter(start: Int) -> some IteratorProtocol {
    // Returns an opaque type that conforms to IteratorProtocol
    return (start...).makeIterator()
}

let c = makeCounter(start: 10)
print(c.next())  // 10
```

**Key characteristics:**
- **Protocol-constrained, like Rust's `impl Trait`:** `some Protocol` is Swift's equivalent.
- **Primary use case: SwiftUI:** `var body: some View` is the canonical example — the concrete view type is an implementation detail.
- **Compiler-resolved:** The compiler knows the concrete type; the programmer and caller do not.

**Strengths:**
- Clean syntax — `some` is intuitive ("some type that conforms to...").
- Well-proven at scale — SwiftUI depends on it.

**Weaknesses:**
- Same single-concrete-type limitation as Rust.
- No equivalent for unconstrained opaque types — must always specify a protocol.

### Kotlin

Kotlin has no direct equivalent. The closest patterns are:
- **Inline functions with reified type parameters** — type is known at call site, not hidden.
- **Sealed classes/interfaces** — the caller sees all possible subtypes; not opaque.
- **`by` delegation** — delegates to another object; the delegate's type is visible.

Kotlin's absence of opaque return types means iterator chains expose their full concrete type (`FilteringSequence<MappingSequence<...>>`) or require boxing into a common interface.

## Orthon-Specific Analysis

### Conflict with TYPE_INFERENCE (EDR-027)

TYPE_INFERENCE requires **explicit annotations at public API boundaries:**

> "Explicit annotations at public API boundaries (parameters and return types)."

Voldemort types (`auto` return) directly violate this rule — the return type
is not explicitly annotated; it is inferred from the function body.

`impl Trait` (Rust model) partially resolves the conflict: the trait bound
is an explicit annotation, but the concrete type is still hidden. The question
is whether a trait bound is "explicit enough" for Orthon's API boundary rule.

**Proposed resolution:** Accept `impl Trait` as satisfying the API boundary
rule, on the grounds that the trait contract is the API — the concrete type
is an implementation detail. Reject `auto` (unconstrained) as insufficiently
explicit.

### Decision Pipeline (Q1–Q10)

Per [`how/process/DECISION_PIPELINE.md`](../../process/DECISION_PIPELINE.md):

1. **What problem are we solving?** Type-level encapsulation at API boundaries — returning complex types without exposing internal structure or sacrificing performance.
2. **Is this a language problem or a library problem?** Language. Requires type-system support for existential types at return positions.
3. **Can it be solved with existing primitives?** No. `dyn Trait` (dynamic dispatch) sacrifices performance and requires heap allocation. `@derive` macros can generate wrapper types but cannot hide them. No existing primitive creates an opaque type alias.
4. **Does it violate any Design Principle?** Yes — conflicts with Explicitness if unconstrained (`auto`). Partially conflicts if constrained (`impl Trait`) — the concrete type is still hidden. See Explicitness analysis above.
5. **Does it add new semantics (vs. syntactic sugar)?** Yes. Existential types at return positions are new type-system semantics — the compiler must track an opaque type identity across call sites while hiding its concrete name.
6. **Can it be expressed through composition?** No. Opaque types are a type-system feature, not expressible through composition of existing constructs.
7. **Can it be syntactic sugar over existing primitives?** No. `impl Trait` is more than sugar — it creates an existential type that the caller cannot name.
8. **Is this an optimisation, not semantics?** No. The semantics (type hiding) are not an optimisation — they change the type system's visibility rules.
9. **Does it affect backward compatibility?** N/A — Orthon is v0.1.
10. **Is it worth adding at all?** Yes — the encapsulation problem is real and affects library design, iterator chains, and async code. However, the form matters: `impl Trait` (constrained) is worth adding; `auto` (unconstrained) is not.

### LLM Generability Gate

| Variant | LLM Generability | Rationale |
|---------|-----------------|-----------|
| `auto` return (D/C++) | ❌ Low | LLM cannot determine available methods from `fn foo() -> auto` alone — must read function body |
| `impl Trait` return (Rust) | ✅ High | LLM sees `fn foo() -> impl Iterator[Item]` and knows the trait contract — can generate calling code |
| `some Protocol` return (Swift) | ✅ High | Same as `impl Trait` — protocol is visible in signature |

**Recommendation:** If Orthon adopts opaque return types, require a trait bound. Reject unconstrained `auto`.

### Interaction with Existing Concepts

| Concept | Interaction |
|---------|------------|
| **TRAITS (EDR-019)** | `impl Trait` depends on TRAITS for the trait bound. The opaque type is checked against the trait at compile time. |
| **TYPE_INFERENCE (EDR-027)** | Conflict — TYPE_INFERENCE requires explicit return types. Resolution: accept `impl Trait` as sufficiently explicit; reject `auto`. |
| **GENERICS (EDR-024)** | `impl Trait` is a form of existential quantification, complementary to universal quantification (`<T: Trait>`). Both are type-level abstractions. |
| **ALGEBRAIC_DATA_TYPES (EDR-039)** | ADTs are an alternative for heterogeneous returns: `type Result = TypeA \| TypeB`. But ADTs name all variants — opaque types hide the concrete type entirely. |
| **dyn Trait** | `impl Trait` is the static-dispatch alternative to `dyn Trait`. Both hide the concrete type; `impl Trait` is zero-cost, `dyn Trait` has runtime overhead. |
| **AST_MACROS (EDR-029)** | Macros cannot create opaque types — the macro output is visible at the expansion site. Opaque types require compiler-level type hiding. |

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Visibility Policy | Controls whether concrete return types can be hidden at API boundaries |
| Dispatch Policy | Determines whether opaque types use static (monomorphised) or dynamic dispatch. Default: static |
| Inference Boundary Policy | Determines whether type inference crosses module boundaries for opaque types |

## Model (What)

If accepted, Orthon's opaque return type would follow the Rust model:

```orthon
fn make_counter(start: Int) -> impl Iterator[Int]
    // The concrete type is a compiler-generated state machine.
    // Callers only see `impl Iterator[Int]`.
    return (start..).iter()

fn main()
    let c = make_counter(10)
    // c: impl Iterator[Int] — concrete type hidden, trait visible
    let first = c.next()  // Some(10)
```

**Semantics:**
- `impl Trait` in return position creates an **existential type**: "there exists some concrete type `T` such that `T: Trait`, and this function returns `T`."
- The concrete type is known to the compiler but hidden from the caller.
- All return paths must return the **same** concrete type. Returning different types from different branches is a compile error.
- Dispatch is **static** (monomorphised) — no vtable, no heap allocation.
- The compiler verifies that the returned type satisfies the declared trait bound(s).

## Default Strategy

**`impl Trait` only — no unconstrained `auto`.** This preserves the API contract (trait bound visible in signature) while providing type-level encapsulation. The trait bound satisfies the Explicitness requirement for API boundaries.

## Alternative Strategies

| Strategy | Description | When to Use | Trade-offs |
|----------|-------------|-------------|------------|
| **No opaque types (current)** | All return types are fully concrete or `dyn Trait` | v0.1 — keeps the type system simple | Sacrifices encapsulation or performance |
| **`auto` return (D model)** | Unconstrained opaque return type | Maximum flexibility | Violates Explicitness; LLM-unfriendly |
| **`impl Trait` (Rust model)** | Trait-constrained opaque return type ✓ | Library APIs, iterator chains, async | Requires trait declaration for every opaque type |
| **`some Protocol` (Swift model)** | Same as Rust with different keyword | Same use cases | `some` vs `impl` — naming choice, not semantic difference |
| **Full existential types** | `exists T. T: Trait` as a first-class type | Advanced use cases beyond return types | Significant type-system complexity; defer to v0.2+ |

## Open Questions

1. Should Orthon support opaque types in v0.1, or defer to v0.2+? The feature is useful but not essential for v0.1 — `dyn Trait` + ADTs cover many use cases.
2. If accepted: `impl Trait` or `some Trait` keyword? Rust uses `impl`; Swift uses `some`. Orthon's guiding principle is "Named Before Symbolic" — `impl` is more explicit.
3. Should opaque types be allowed in `let` bindings (not just return types)? Rust supports `let x: impl Trait = ...` in some contexts.
4. Should opaque types support multiple trait bounds (`impl TraitA + TraitB`)? Rust supports this. It adds complexity but is sometimes necessary.
5. How do opaque types interact with Orthon's Schema Provider? The Schema Provider should reveal the opaque type's trait bounds but not its concrete type.
6. Should there be a way to name an opaque type for reuse? Rust does not allow this; the opaque type is truly anonymous.

## Decision History

- 2026-08-04: Initial research document created from D language idiom analysis. Comparative analysis of D, Rust, C++, Swift, and Kotlin. Pipeline Q1–Q10 answered inline. Classification: **important tier** — real problem, conflict with Explicitness is resolvable via `impl Trait`. Recommend deferring acceptance decision to Milestone 2 (Concept Design Review).

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/concepts/TYPE_INFERENCE.md` — may need amendment to API boundary rule
- [ ] `what/concepts/TRAITS.md` — `impl Trait` depends on TRAITS
- [ ] `what/concepts/GENERICS.md` — existential vs universal quantification
- [ ] `what/GLOSSARY.md` — new term: "opaque type", "existential type"
- [ ] `how/DESIGN_PRINCIPLES.md` — Explicitness analysis
- [ ] `how/gates/_language-design.md` — gate entry if pursued
