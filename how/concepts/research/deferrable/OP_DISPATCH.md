# opDispatch — Compile-Time Method Interception

> **⚠️ DRAFT — This document is a preliminary research analysis.**
> It has not passed Concept Design Review.
>
> **Source:** D language idiom analysis — `opDispatch` is a template-based
> mechanism that intercepts calls to non-existent methods at compile time,
> analogous to Python's `__getattr__` but resolved during compilation.
>
> **Last updated:** 2026-08-04

## Issue (Why)

How does a type handle method calls whose names are not known until
compile time — enabling proxy patterns, API forwarding, and dynamic
dispatch without inheritance or runtime reflection?

Consider a proxy that forwards all method calls to an inner object:

```orthon
// Without opDispatch: every method must be explicitly forwarded
type Proxy
    inner: Service

    fn do_work(self, x: Int) -> Result[String, Error]
        return self.inner.do_work(x)

    fn get_status(self) -> Status
        return self.inner.get_status()

    fn reset(self) -> Result[None, Error]
        return self.inner.reset()

    // ... 50 more forwarding methods — pure boilerplate
```

Every method on `Service` requires a corresponding forwarding method on
`Proxy`. This is mechanical, error-prone, and scales linearly with the
size of the proxied interface. The same problem appears in:

- **Decorator/wrapper types** that add logging, metrics, or validation
- **API compatibility layers** that rename or reshape methods
- **Mock/test doubles** that intercept all calls for recording
- **Remote procedure call stubs** that serialize method calls

The core problem: **the set of methods a type must respond to is known
at compile time, but writing each forwarding method by hand is pure
boilerplate.** A mechanism to intercept *any* method call at compile time
would collapse dozens of identical forwarding methods into a single
template.

## Principles

Which principles must not be violated? Reference: [`DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md).

1. **Explicitness** — "The meaning of code should be apparent from its surface form." A call `proxy.unknownMethod()` must not silently succeed via hidden dispatch — the dispatch mechanism must be syntactically visible at the type declaration.
2. **LLM Generability** — an LLM must be able to determine what methods are available on a type. If `opDispatch` intercepts *all* unknown methods, the LLM cannot enumerate the effective interface.
3. **Semantic Purity** — each syntactic construct has exactly one meaning. Method calls (`obj.method()`) should not sometimes resolve statically and sometimes go through `opDispatch` depending on context.
4. **Orthogonality** — does `opDispatch` compose with traits, generics, error handling, and pattern matching without special cases?
5. **Minimal Core** — can the use cases be served by existing mechanisms (DELEGATE, AST_MACROS, PATTERN_MATCHING_DISPATCH)?

## Comparative Analysis

### D: `opDispatch`

D provides `opDispatch` as a template member function that intercepts
compile-time method resolution:

```d
struct Proxy(T) {
    T target;

    // Intercepts ALL method calls not explicitly defined on Proxy
    auto opDispatch(string name, Args...)(Args args) {
        import std.stdio;
        writeln("Calling ", name, " with ", args.length, " args");
        return mixin("target." ~ name ~ "(args)");
    }
}

void main() {
    auto p = Proxy!MyService(service);
    p.doWork(42);      // "Calling doWork with 1 args" — forwarded to service
    p.getStatus();     // "Calling getStatus with 0 args" — forwarded to service
    p.anyMethod(1, 2); // Still works — opDispatch catches everything
}
```

**Key characteristics:**
- **Compile-time resolution:** The method name is available as a compile-time string template parameter. The compiler resolves `obj.method()` by: (1) check for explicit method `method`, (2) if not found, check for `opDispatch`, (3) if `opDispatch` exists, instantiate it with `"method"`.
- **String-based method name:** `opDispatch` receives the method name as a `string` template parameter. This enables string manipulation and `mixin`-based code generation.
- **Template-based, not trait-based:** `opDispatch` is a template, not a trait implementation. Any struct with `opDispatch` automatically handles all unknown methods.
- **No trait constraint:** There is no `Dispatchable` trait — `opDispatch` is duck-typed at the template level.

**Strengths:**
- Eliminates forwarding boilerplate — one `opDispatch` replaces N forwarding methods.
- Compile-time — no runtime reflection overhead. All dispatch is resolved and inlined.
- Composes with D's template metaprogramming — method name manipulation at compile time.

**Weaknesses:**
- **LLM-hostile:** An LLM cannot determine what methods a type supports by reading its declaration — `opDispatch` is a black box that accepts everything.
- **String-based, not typed:** D uses `mixin(string)` for code generation — Orthon rejects string-based metaprogramming in favor of typed AST macros (see EDR-029).
- **No API contract:** `opDispatch` provides no type-level guarantee about which methods will succeed. A typo like `proxy.doWrok()` silently becomes `opDispatch("doWrok")` — no compile error.
- **Hidden control flow:** The method resolution path is invisible to the caller.

### Python: `__getattr__`

Python's runtime equivalent provides dynamic attribute access:

```python
class Proxy:
    def __init__(self, target):
        self._target = target

    def __getattr__(self, name):
        # Called only when normal attribute lookup fails
        print(f"Calling {name}")
        return getattr(self._target, name)

p = Proxy(service)
p.do_work(42)    # __getattr__("do_work") → service.do_work(42)
p.typo_method()  # __getattr__("typo_method") → AttributeError from service
```

**Key characteristics:**
- **Runtime, not compile-time:** Method interception happens at call time, not compilation. No static verification.
- **String-based:** Like D, the method name is a runtime string.
- **Fallback-only:** `__getattr__` is called only after normal attribute lookup fails. Explicitly defined methods take priority.

**Relevance to Orthon:** Low. Orthon is statically typed and has no runtime reflection. Python's model is informative for the *semantics* of method interception, but not the mechanism.

### Ruby: `method_missing`

Ruby's equivalent is even more dynamic:

```ruby
class Proxy
  def initialize(target)
    @target = target
  end

  def method_missing(name, *args)
    puts "Calling #{name}"
    @target.send(name, *args)
  end
end
```

**Relevance to Orthon:** Low. Ruby's model is fully runtime, fully dynamic. Orthon's comptime model (see COMPILE_TIME_EXECUTION, EDR-031) is the compile-time alternative.

### Rust

Rust has no `opDispatch` equivalent. The closest mechanisms:

| Mechanism | Covers | Not Covered |
|-----------|--------|-------------|
| `Deref` trait | Automatic forwarding to inner type via `*` | Only one level; method name must exist on target |
| `DerefMut` trait | Same with mutable access | Same limitations |
| Proc macros (`#[derive(...)]`) | Generate forwarding methods at compile time | Must know method names at macro-write time; cannot intercept *arbitrary* calls |
| `async_trait` / dynamic dispatch | Runtime dispatch through vtable | Runtime overhead; not compile-time |

Rust's philosophy is that method dispatch should always be explicit and verifiable. There is no mechanism for "handle any method call." The absence of `opDispatch` in Rust — a language that values explicitness and compile-time safety — is instructive.

### Kotlin

Kotlin provides class delegation via the `by` keyword:

```kotlin
interface Service {
    fun doWork(x: Int): String
    fun getStatus(): Status
}

class Proxy(target: Service) : Service by target {
    // All Service methods automatically forwarded to target
    // Override specific methods as needed
    override fun getStatus(): Status {
        log("getStatus called")
        return target.getStatus()
    }
}
```

**Key characteristics:**
- **Trait-bounded, not unbounded:** Delegation is constrained to a specific interface. `by target` only forwards methods declared in `Service`, not arbitrary methods.
- **Compile-time:** The compiler generates forwarding methods for all interface methods not explicitly overridden.
- **LLM-friendly:** The interface is visible in the type declaration. An LLM can enumerate available methods from the interface.

## Orthon-Specific Analysis

### What problem does opDispatch solve that existing concepts don't?

Orthon already has several mechanisms that address parts of the forwarding/decoration problem:

| Existing Concept | What It Covers | Gap (what opDispatch would add) |
|-----------------|----------------|-------------------------------|
| **DELEGATE model (EDR-033)** | Isolated execution contexts with message passing | Delegate is for concurrency, not method forwarding on a single type |
| **COMMAND_PATTERN_VIA_DELEGATE** | Command pattern via delegate + function | Specific command pattern, not general method interception |
| **AST_MACROS (EDR-029)** | Code generation at compile time via typed AST functions | Can generate forwarding methods, but must know method names at macro-write time; cannot intercept *arbitrary* unknown method calls |
| **PATTERN_MATCHING_DISPATCH (EDR-026)** | Multimethod dispatch on argument types | Dispatches on *arguments*, not *method names* |
| **TRAITS (EDR-019)** | Behavioural contracts with explicit `impl` | Requires explicit method declarations; no "catch-all" |
| **EXTENSION_FUNCTIONS** | Add methods to external types | Adds known methods, not arbitrary interception |

**The gap:** None of these allow a type to handle *any* method name without declaring it explicitly. The proxy/decorator use case — "forward all methods on `Service` to `self.inner`" — still requires boilerplate or macro-based code generation that knows the method names in advance.

### Decision Pipeline (Q1–Q10)

Per [`how/process/DECISION_PIPELINE.md`](../../../process/DECISION_PIPELINE.md):

1. **What problem are we solving?** Eliminating forwarding boilerplate for proxy, decorator, and API-compatibility patterns without sacrificing static dispatch performance.
2. **Is this a language problem or a library problem?** Language. Method resolution is a compiler concern — the compiler decides what `obj.method()` means. A library cannot alter method dispatch semantics.
3. **Can it be solved with existing primitives?** Partially. AST_MACROS can generate forwarding methods if method names are known at macro-write time. DELEGATE covers the concurrency use case. But neither covers "intercept any method call."
4. **Does it violate any Design Principle?** **Yes — multiple:**
   - **Explicitness:** `proxy.anyMethod()` succeeds without `anyMethod` being declared anywhere visible to the caller.
   - **LLM Generability:** An LLM cannot enumerate the effective interface of a type with `opDispatch`.
   - **Semantic Purity:** Method calls have context-dependent resolution — sometimes static, sometimes through `opDispatch`.
5. **Does it add new semantics (vs. syntactic sugar)?** Yes. Method resolution fallback to a comptime-evaluated handler is new compiler semantics.
6. **Can it be expressed through composition?** No — method dispatch is a primitive compiler operation.
7. **Can it be syntactic sugar over existing primitives?** No.
8. **Is this an optimisation, not semantics?** No. It changes which method calls are valid — semantic, not optimisation.
9. **Does it affect backward compatibility?** N/A — Orthon is v0.1.
10. **Is it worth adding at all?** **Defer.** The proxy/decorator use case is real, but the cost to Explicitness and LLM Generability is high. A trait-bounded delegation mechanism (Kotlin's `by` model) would cover the primary use case without sacrificing these principles. Revisit in v0.2+.

### LLM Generability Gate — Deep Analysis

This is the critical gate for `opDispatch`. Method dispatch is the primary
way LLMs interact with types. If an LLM cannot determine what methods a
type supports, it cannot generate correct code.

| Scenario | Without opDispatch | With opDispatch |
|----------|-------------------|-----------------|
| LLM generates `proxy.doWork(x)` | Compiler checks: does `Proxy` have `doWork`? Yes/no. | Compiler checks: does `Proxy` have `doWork`? No → falls through to `opDispatch("doWork")` → what happens? LLM cannot predict. |
| LLM generates `proxy.doWrok(x)` (typo) | Compile error: `Proxy has no member 'doWrok'` | `opDispatch("doWrok")` — silently accepted, likely a runtime error from the target |
| IDE autocomplete on `proxy.` | Lists all declared methods | Cannot list methods — `opDispatch` accepts everything |
| Documentation generation | Lists declared methods from source | Cannot enumerate effective interface |

**Verdict:** `opDispatch` in its full D form (unbounded, string-based) is **incompatible with Orthon's LLM Generability Gate.** A constrained form (trait-bounded delegation) would pass, but that is a different feature — closer to Kotlin's `by` than D's `opDispatch`.

### A Typed-AST Alternative

If Orthon were to explore this space, the mechanism would need to differ
from D's approach in two fundamental ways:

1. **Typed, not string-based:** D uses `mixin(string)` for code generation. Orthon would use typed AST nodes (consistent with AST_MACROS, EDR-029). The method name would be a comptime-known symbol, not a string.
2. **Trait-bounded, not unbounded:** Rather than intercepting *all* unknown methods, the mechanism should be constrained to methods declared in a specific trait. This preserves the API contract and LLM Generability.

Example of a hypothetical trait-bounded form:

```orthon
// Hypothetical — not accepted
type Proxy(inner: Service) delegates Service
    // Compiler generates forwarding methods for all Service methods
    // unless explicitly overridden
    fn get_status(self) -> Status
        log("get_status called")
        return self.inner.get_status()  // override with logging
```

This is closer to Kotlin's `by` keyword than D's `opDispatch`. It would be
a separate concept — **Trait Delegation** — not `opDispatch`.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Method Resolution Policy | Determines whether method resolution can fall through to a comptime handler |
| Dispatch Policy | Static (monomorphised) — the comptime handler is inlined at each call site |
| Interface Visibility Policy | Controls whether IDE tooling can enumerate the effective interface of a type |

## Model (What)

**D's `opDispatch` is NOT recommended for Orthon v0.1.** The LLM Generability
and Explicitness conflicts are fundamental, not superficial. The primary use
case — forwarding boilerplate elimination — is better served by a trait-bounded
delegation mechanism (Kotlin's `by` model), which would be a separate concept.

If revisited for v0.2+, the mechanism must be:
1. **Typed-AST-based,** not string-based (consistent with EDR-029).
2. **Trait-bounded,** not unbounded (to satisfy LLM Generability Gate).
3. **Explicitly declared** on the type (to satisfy Explicitness).

## Default Strategy

**Do not include `opDispatch` in Orthon v0.1.** The use cases are served by AST_MACROS for known-method codegen and by a potential future Trait Delegation concept for forwarding boilerplate.

## Alternative Strategies

| Strategy | Description | When to Use | Trade-offs |
|----------|-------------|-------------|------------|
| **AST_MACROS (current)** | Generate forwarding methods via `@macro` functions | Method names known at macro-write time | Cannot intercept unknown method names |
| **Trait Delegation (future)** | `type Proxy(inner: Service) delegates Service` — auto-forwards all trait methods | Proxy/decorator patterns with known interface | Requires trait declaration; not fully dynamic |
| **Full opDispatch (D model)** | Compile-time interception of any method call | Maximum flexibility for metaprogramming | Violates Explicitness and LLM Generability |
| **Kotlin-style `by`** | Class delegation constrained to a specific interface | Clean forwarding with override capability | Requires explicit trait per delegated interface |

## Open Questions

1. Should a trait-bounded delegation mechanism (Kotlin's `by` model) be researched as a separate concept for v0.2+?
2. Can AST_MACROS be extended to support "generate forwarding methods for all methods of trait T" without knowing method names in advance? This would require comptime reflection over trait method lists.
3. If `opDispatch` is permanently rejected, does the gap between AST_MACROS and full `opDispatch` leave any use case unaddressed that DELEGATE + COMMAND_PATTERN_VIA_DELEGATE cannot cover?

## Decision History

- 2026-08-04: Initial research document created from D language idiom analysis. Comparative analysis of D, Python, Ruby, Rust, and Kotlin. Pipeline Q1–Q10 answered inline. Classification: **deferrable tier** — real use case (proxy/decorator forwarding), but fundamental conflicts with Explicitness and LLM Generability Gate. D's `opDispatch` model (string-based, unbounded) is incompatible with Orthon's design principles. A future trait-bounded delegation mechanism (Kotlin `by` model) would be a separate concept. Recommend deferring to v0.2+.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md` — if eventually accepted or formally rejected
- [ ] `how/concepts/research/essential/AST_MACROS.md` — alternative mechanism
- [ ] `how/concepts/research/essential/COMPILE_TIME_EXECUTION.md` — comptime model
- [ ] `how/concepts/research/essential/PATTERN_MATCHING_DISPATCH.md` — related dispatch concept
- [ ] `how/concepts/research/essential/DELEGATE.md` — concurrency delegate model
- [ ] `how/concepts/research/important/COMMAND_PATTERN_VIA_DELEGATE.md` — command dispatch
- [ ] `what/GLOSSARY.md` — if terms introduced
- [ ] `how/gates/_language-design.md` — LLM Generability Gate analysis
