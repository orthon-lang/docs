# Hypothesis: LLM as a `delegate` Implementation, Not a Language Concept

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-29
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).
>
> **Related:** [`DELEGATE.md`](../../essential/DELEGATE.md) — delegate as an execution policy,
> [`COMMAND_PATTERN_VIA_DELEGATE.md`](../COMMAND_PATTERN_VIA_DELEGATE.md) —
> command pattern elimination via delegate,
> [`LLM_NATIVE_TOOLCHAIN.md`](../../deferrable/LLM_NATIVE_TOOLCHAIN.md) —
> LLM-native toolchain (separate concern: tooling, not language semantics).

---

## Issue (Why)

Modern applications increasingly use LLMs as part of business logic. Today,
such calls are typically plain HTTP requests with a string prompt and manual
response parsing. This leads to several problems:

- **No typing of inputs and outputs.** The prompt is a raw string. The response
  is a raw string. The contract between caller and LLM exists only in the
  programmer's head — or in a doc comment that no tool verifies.

- **Prompts exist outside the language model.** There is no way to reference
  a prompt from the type system, validate it statically, or compose it with
  other language constructs.

- **No static analysis.** The compiler cannot check whether the expected
  response shape matches the code that consumes it. A field rename in the
  output schema silently breaks the parser at runtime.

- **Testing and mocking are ad-hoc.** Every codebase invents its own LLM mock.
  There is no standard way to replace an LLM call with a deterministic
  implementation for tests.

- **Weak integration with tracing and observability.** LLM calls are invisible
  to the language's execution trace. Latency, token usage, cost — all must be
  measured through external tooling glued to HTTP clients.

- **Provider lock-in.** The calling code is coupled to a specific provider's
  API (OpenAI, Anthropic, etc.). Switching providers requires rewriting
  integration code.

The deeper question: **should LLM invocation be a language concept, or an
implementation detail of an existing concept?**

---

## Hypothesis

**Do not introduce LLM as a new language concept.** Instead, treat LLM as
**one implementation (`delegate`) of an abstract action (`act`).**

```
act summarize(text: String) -> Summary

impl llm {

    model gpt-5

    prompt """
    Summarize:

    {{text}}
    """
}
```

From the language's perspective, `act` remains an ordinary contract — a
callable entity with typed inputs and outputs. The `impl llm` block is a
**delegate implementation** that happens to use an LLM as its execution
backend.

Other implementations are equally valid:

- `native` — a hand-written function
- `remote` — an RPC call
- `http` — a REST endpoint
- `sql` — a database query
- `llm` — an LLM invocation
- `mock` — a deterministic test double

The compiler, aware of the `llm` implementation kind, can:

- **Generate structured response schemas** from the return type, so the LLM
  is prompted to produce valid JSON matching the contract.
- **Validate results** against the return type at runtime (or statically
  when the LLM supports constrained decoding).
- **Auto-generate mock implementations** for testing — a `mock` delegate
  that returns canned responses matching the contract.
- **Include LLM calls in execution traces** — latency, token count, model,
  and cost appear as part of the standard execution trace, not as a
  separate observability silo.
- **Support replay and snapshot testing** — record LLM responses and replay
  them deterministically.

Thus, LLM becomes an **execution policy**, not part of the language's
computational model.

---

## Principles

1. **Orthogonal core.** The language core defines *what* is computed. *How*
   it is computed — locally, remotely, or via LLM — is a delegate concern.
   No LLM-specific syntax or semantics enter the core language.

2. **Explicitness.** The `impl llm` block makes the LLM dependency visible
   at the declaration site. It is not hidden behind a library call or a
   string prompt buried in application code.

3. **Uniform execution model.** An `act` with an `llm` delegate has the same
   calling convention, error model, and lifetime as any other delegate.
   Callers do not know or care which backend executes the contract.

   > **⚠️ Determinism carve-out.** The calling convention is uniform, but
   > the *determinism guarantee* is not. Pure-language delegates (`native`,
   > `mock`) satisfy Orthon's Deterministic Behavior principle (§
   > `DESIGN_PRINCIPLES.md`): the same inputs always produce the same
   > observable output. An `llm` delegate does not — two calls with
   > identical arguments may return different results. This is an inherent
   > property of LLM inference and is not fixable at the language level.
   >
   > **Consequence for callers:** Code that depends on deterministic
   > behaviour (e.g., idempotent retry, memoisation, repeatable
   > computation) should not assume `act` calls are deterministic unless
   > the delegate kind is known to be so. This is a semantic constraint
   > that lives at the call site, not in the type system.

4. **Testability by substitution.** Replacing an `llm` delegate with a
   `mock` or `native` delegate requires changing only the `impl` block —
   not the call sites.

5. **Provider independence.** The `model` declaration names a capability
   ("gpt-5"), not an HTTP endpoint. Provider resolution is a deployment
   concern, not a language concern.

---

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Execution Policy (`delegate`) | LLM is one delegate kind among many (`native`, `remote`, `http`, `sql`, `llm`, `mock`). |
| Contract Enforcement Policy | The return type of an `llm` delegate defines the schema the LLM must produce. Validation is automatic. |
| Observability Policy | LLM calls participate in standard execution tracing (latency, tokens, model, cost). |
| Testing Policy | Mock delegates are auto-generable from the contract. Replay/snapshot testing is a first-class capability. |
| Provider Resolution Policy | Model names are symbolic; mapping to endpoints is deployment configuration. |

---

## Model (What)

### The `act` Contract

An `act` declares a typed, named contract — a callable interface with inputs
and outputs:

```
act classify(text: String) -> Sentiment
act translate(text: String, target: Language) -> String
act generate_image(prompt: String, style: Style) -> Image
```

The contract says nothing about *how* it is fulfilled.

### Delegate Implementations

A delegate binds a contract to an execution backend:

```
// Native implementation — hand-written code
impl native for classify {
    // ordinary Orthon code
    if text.contains("happy") { return Sentiment::Positive }
    else { return Sentiment::Negative }
}

// LLM implementation — prompt-driven
impl llm for classify {
    model claude-4
    prompt """
    Classify the sentiment: {{text}}
    Respond with exactly one of: Positive, Negative, Neutral.
    """
}

// Mock implementation — deterministic test double
impl mock for classify {
    return Sentiment::Positive
}
```

### The `llm` Delegate Block

An `llm` delegate block contains:

| Element | Purpose | Required |
|---------|---------|----------|
| `model` | Which model to use (symbolic name) | Yes |
| `prompt` | The prompt template with `{{placeholder}}` interpolation | Yes |
| `temperature` | Generation temperature | No (default: provider default) |
| `max_tokens` | Response token limit | No (default: inferred from return type) |
| `output_format` | Constrained decoding format | No (default: JSON matching return type) |

### Call Site

Call sites are identical regardless of delegate kind:

```
let result = classify(text)    // Could be native, llm, remote — caller doesn't care
```

---

## Default Strategy

The default strategy for `llm` delegates:

1. **Schema generation.** The compiler generates a JSON Schema from the
   return type and injects it into the prompt as a formatting instruction.

2. **Validation.** The LLM response is parsed and validated against the
   return type. Validation failure produces a typed error (not a raw
   string).

3. **Execution tracing.** Every LLM call emits a trace span with:
   - Model name
   - Prompt (trimmed to N characters)
   - Response (trimmed to N characters)
   - Latency
   - Token count (prompt + completion)
   - Estimated cost

4. **Error taxonomy.** LLM-specific errors (timeout, rate limit, content
   filter, invalid JSON) map to standard Orthon error types. The caller
   handles them through the normal error-handling mechanism.

---

## Nondeterminism Semantic Consequences

This section documents what nondeterminism means for code that calls an
`llm` delegate — and what the language and tooling should provide in
response.

### Relationship to Orthon's Deterministic Behavior Principle

Orthon's [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) states:

> **Deterministic Behavior** — The same source code should behave
> identically across optimization levels and implementations.

The `llm` delegate model creates a **carve-out** to this principle:
pure-language code (expressions, control flow, native delegates) remains
fully deterministic. Nondeterminism is confined to `llm` delegate
invocations, which are gated behind an explicit `impl llm` declaration.
The principle applies to **language semantics**; LLM inference is an
**external service call** whose behaviour is outside the language's
semantic guarantees.

This carve-out has precedent in the project: `PARALLELISM_EXECUTOR_MODEL.md`
already identifies intra-task determinism vs. inter-task nondeterminism
as a necessary qualification to the principle.

### For Calling Code

- **Idempotency is not automatic.** The same `act` call with the same
  arguments may return different results on retry. Code that needs
  idempotent LLM calls must use application-level deduplication
  (e.g., content hashing, external idempotency keys).
- **Memoisation requires explicit opt-in.** A generic cache keyed by
  input arguments alone is unsound. The application must decide whether
  variant results are acceptable, or use a semantic similarity check.
- **Composition does not amplify nondeterminism.** If an `act` calls
  another `act` that happens to be an `llm` delegate, the outer call
  is not *more* nondeterministic — but it inherits the inner call's
  nondeterminism. Replacing an inner `llm` with a `native` or `mock`
  delegate restores determinism at that level. This is the key advantage
  of the delegate model: nondeterminism is bounded by the `impl` block.

### For Testing

The delegate model enables three testing strategies with escalating
semantic guarantees:

1. **Mock substitution — deterministic correctness.** Replace `impl llm`
   with `impl mock` for unit tests. Tests validate type conformance,
   control flow, and error handling — they do not test LLM behaviour.
   This is the default and cheapest strategy.

2. **Replay testing — deterministic snapshots.** Record real LLM
   responses (inputs, prompt, output) in a snapshot file. Replay the
   snapshot deterministically in CI. Fails if the code changes the
   prompt or expected schema — guards against regressions without
   calling the LLM.

3. **Live testing — nondeterminism-aware assertions.** For integration
   tests that do call the LLM, traditional `assert result == expected`
   is insufficient because the result is inherently variable. The
   language should support testing patterns that acknowledge this:
   - **Range assertions:** `assert result.confidence > 0.8`
   - **Structural validation:** `assert result matches Sentiment`
   - **Quorum tests:** run N times, assert that >= K results satisfy
     a predicate (inspired by BAML's `test ... with quorum`).

   > **Note:** Quorum tests and nondeterminism-aware assertions are not
   > yet part of Orthon's language-level testing model (which does not
   > exist yet — see Gap #5 in `notes/baml-concepts-orthon-gap-analysis.md`).
   > This section establishes the *requirement*; the testing model
   > concept will define the concrete syntax.

### For Tracing and Observability

BAML's philosophy is *"trace nondeterminism"* — make the variability
visible in tooling. The `llm` delegate model should provide:

- **Variance tracking per call site.** The execution trace records not
  just latency and tokens, but also the *output distribution* across
  repeated calls: how often does the same input produce different
  outputs, and how divergent are those outputs?
- **Nondeterminism propagation markers.** When an `act` transitively
  calls an `llm` delegate, the trace marks the entire call chain as
  nondeterministic-influenced. This mirrors BAML's insight: an LLM call
  anywhere in the call graph makes everything above it consequential
  for nondeterminism.
- **Replay-aware tracing.** The trace format distinguishes between a
  live LLM call (nondeterministic, sampled) and a replayed call
  (deterministic, from snapshot). This prevents false conclusions when
  comparing traces across environments.

---

## Alternative Strategies

### LLM as a Native Language Function

```
fn summarize(text: String) -> Summary {
    """
    ...
    """
}
```

**Pros:** maximally concise syntax; deep type-system integration.

**Cons:** the language becomes dependent on a specific class of technology;
the computational model and external service are conflated; harder to
support alternative implementations.

### LLM as a Library

```
let summary = llm.call(prompt)
```

**Pros:** minimal language impact; simple to implement.

**Cons:** static contract information is lost; typing, tracing, and testing
become ad-hoc library concerns rather than language guarantees.

### LLM as a New Language Concept (`model` keyword, `prompt` type, etc.)

**Pros:** could provide richer LLM-specific semantics (streaming, tool
calling, multimodal).

**Cons:** violates orthogonality — introduces a concept that overlaps with
functions, delegates, and RPC; every LLM-specific feature must be supported
by every backend; the language core grows with technology-specific surface
area.

---

## Trade-offs

### Advantages

- **Core language remains technology-independent.** LLM is a delegate kind,
  not a language feature. The language does not need to evolve when LLM
  technology changes.

- **Uniform execution model.** Local, remote, and AI calls share the same
  contract-delegate pattern. Tooling (tracing, testing, mocking) works
  uniformly.

- **Simple testing.** Replace `impl llm` with `impl mock` or `impl native`.
  No special mocking framework required.

- **No provider lock-in at the language level.** Model names are symbolic.
  Provider resolution is deployment configuration.

- **Aligns with the `act`/`delegate` architecture.** LLM is a natural
  extension of the existing execution model, not a parallel universe.

### Disadvantages

- **The language cannot guarantee semantic correctness of the response** —
  only conformance to the type contract. An LLM may return a well-typed but
  factually wrong answer. This is inherent to LLMs, not fixable at the
  language level.

- **Non-determinism remains a property of the implementation.** Two calls
  with the same inputs may produce different outputs. This is structural
  nondeterminism (the LLM samples from a probability distribution), not
  merely implementation variance (e.g., hash order or timing). It:
  - **Breaks idempotency assumptions** — retrying after a failure may
    produce a semantically different (though well-typed) result.
  - **Invalidates memoisation by default** — caching by input arguments
    alone is incorrect unless the application explicitly accepts stale
    or variant results.
  - **Affects testing strategy** — `assert output == expected` is
    insufficient; quorum tests, range assertions, and statistical
    validation are required instead (see § Nondeterminism Semantic
    Consequences).

- **A separate runtime or framework is needed for production-quality
  support.** Caching, retries, model selection, cost tracking, rate
  limiting, and advanced observability are runtime concerns beyond the
  language specification. The language provides the *contract*; the runtime
  provides the *operational quality*.

- **Some capabilities may require delegate-kind extensions.** Streaming
  responses, tool calling, and multimodal inputs may need additional
  annotations on the `llm` delegate block — but these are extensions of the
  implementation, not the language core.

---

## Related Concepts and Alternatives

### Close Relatives

- **RPC as a remote delegate implementation.** Same pattern: a typed
  contract, a remote execution backend. LLM is "RPC to a model."

- **SQL as a specialized query executor.** A `sql` delegate kind would
  accept a query and return typed results. Same pattern, different backend.

- **Actor/Delegate as the execution model.** The `delegate` concept
  ([`DELEGATE.md`](../../essential/DELEGATE.md)) already provides the
  infrastructure: state ownership, serialized access, execution policy.
  LLM is one more policy.

- **Dependency Injection.** Choosing an implementation for a contract is
  DI. The `impl` block is a declarative DI binding at the module level.

### Concept Boundary

This concept does **not** cover:

- **LLM-native toolchain** — see
  [`LLM_NATIVE_TOOLCHAIN.md`](../../deferrable/LLM_NATIVE_TOOLCHAIN.md)
  for schema generation, LLM-oriented diagnostics, and the LLM Generability
  Gate. The toolchain is a separate concern from language semantics.

- **Prompt engineering or management.** Prompt templates are strings in
  `impl llm` blocks. Versioning, A/B testing, and prompt libraries are
  tooling/runtime concerns.

- **Streaming responses.** If an `act` returns a `Sequence`, an `llm`
  delegate could stream tokens. This is a delegate-kind extension, not a
  language change.

**Within scope:** Nondeterminism semantics — see
[§ Nondeterminism Semantic Consequences](#nondeterminism-semantic-consequences).
This section defines what nondeterminism means for calling code, testing,
and tracing. It does *not* propose changes to Orthon's Deterministic
Behavior principle (which requires a Tier 1 EDR); it documents the
carve-out that `llm` delegates create.

---

## Open Questions

1. **Should `impl llm` support structured output declaration separately from
   the return type?** The return type defines the schema, but some LLM use
   cases need a different output shape than the language-level return type
   (e.g., the LLM produces an intermediate representation that gets
   transformed).

2. **How should multi-step LLM workflows (chain-of-thought, agents) map to
   the delegate model?** A single `act` → single `impl llm` maps cleanly to
   one LLM call. Multi-step workflows may need composition of multiple
   `act` declarations — or a higher-level orchestration construct.

3. **Should `impl llm` support fallback models?** If the primary model is
   unavailable, should the delegate transparently fall back to another
   model? This is probably a runtime concern, not a language concern.

4. **How does error handling compose with LLM-specific failures?** Content
   filter rejections, token limit exceeded, invalid JSON — should these map
   to standard error types, or does `impl llm` need its own error taxonomy?

5. **Does `impl llm` need a `context` parameter for conversation history?**
   Multi-turn conversations require passing previous messages. Is this a
   parameter of the `act` (making the contract conversation-aware), or a
   property of the `llm` delegate (making it the delegate's responsibility
   to manage history)?

6. **Should the contract surface nondeterminism to the type system?** An
   `act` hides its delegate kind from callers — call sites cannot tell
   whether a call is deterministic (`native`, `mock`) or nondeterministic
   (`llm`). This is by design (§ Principles — uniform execution model),
   but it means the compiler cannot warn when a nondeterministic call is
   used in a context that assumes determinism (e.g., a pure function, a
   compile-time evaluation, a `const` initialiser). Options:
   - **No annotation** (current design). Callers assume nothing about
     determinism. The burden is on the application to test appropriately.
   - **Explicit effect annotation.** Add `@nondeterministic` (or similar)
     to the `impl llm` block. The compiler propagates it to all transitive
     callers, catching accidental use in deterministic contexts.
   - **Delegate kind leak.** Make the delegate kind part of the `act`'s
     type signature (e.g., `act ... impl llm`), sacrificing uniformity
     for visibility.

---

## Conclusion

LLM is not a fundamental language abstraction and should not enter Orthon's
Essential Core. The most natural model is to treat LLM as **one delegate
implementation kind for `act`**. This preserves language purity, provides a
uniform execution model, and enables AI integration without modifying
Orthon's base semantics.

The delegate model isolates nondeterminism behind the `act`/`delegate`
boundary, containing it without eliminating it. This is a deliberate
trade-off: Orthon preserves its deterministic core and avoids
LLM-specific semantics entering the language, at the cost of making
nondeterminism invisible at the type level. The carve-out to Principle 3
(§ Principles) and the tracing/testing strategies (§ Nondeterminism
Semantic Consequences) provide the visibility that the type system
deliberately withholds.

---

### Affected Documents

- [ ] `DELEGATE.md` — add `llm` to the list of delegate kinds; document
      determinism carve-out
- [ ] `IMPLEMENTATION_POLICIES.md` — add LLM Delegate Policy; add
      Nondeterminism Policy or cross-reference
- [ ] `IMPLEMENTATION_STRATEGIES.md` — add LLM delegate to execution
      strategies
- [ ] `GLOSSARY.md` — add `llm delegate` term; add `nondeterminism`
      (delegate-level) term
- [ ] `what/CORE_CONCEPTS.md` — no change (LLM is not a core concept)
- [ ] `DESIGN_PRINCIPLES.md` — **no change required** (document is locked;
      the determinism carve-out is documented here, not in the principles
      document itself)
- [ ] `notes/baml-concepts-orthon-gap-analysis.md` — update status of Gap #7
      (Nondeterminism) from "no action" to "addressed by LLM_AS_DELEGATE_IMPL.md
      § Nondeterminism Semantic Consequences"
