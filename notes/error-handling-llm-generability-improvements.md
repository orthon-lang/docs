# Error Handling — LLM Generability Improvements

> **Date:** 2026-07-29
> **Context:** BAML comparison (`notes/baml-concepts-orthon-gap-analysis.md` §6) — BAML's catch-by-type approach is simpler for LLMs than Orthon's `match` on `Result`/`!T`. Two non-language-change remedies proposed.
>
> **See also:** [`what/concepts/ERROR_HANDLING.md`](../what/concepts/ERROR_HANDLING.md),
> [`what/concepts/ERROR_UNION.md`](../what/concepts/ERROR_UNION.md),
> [`how/decision_records/architecture/EDR-020-error-handling.md`](../how/decision_records/architecture/EDR-020-error-handling.md),
> [`how/decision_records/architecture/EDR-023-error-union.md`](../how/decision_records/architecture/EDR-023-error-union.md),
> [`notes/llm-generability-gate.md`](llm-generability-gate.md)

---

## Problem

Orthon's error handling is more principled than BAML's `throws`/`catch`
(errors are values, composition via combinators, no privileged syntax).
However, the BAML comparison reveals a genuine LLM generability gap:

| Aspect | BAML `catch` | Orthon `match` on `Result` |
|--------|-------------|---------------------------|
| Arms map to error types | ✅ Direct: `Type => handler` | ⚠️ Nested: `match result { Ok(v) -> ..., Error(e) -> match e { ... } }` |
| Error set discovery | ✅ Inferred, tooltip-visible | ⚠️ LLM must read callee signature or Schema Provider |
| Boilerplate per call site | Minimal — one `catch` block | Nested `match` + two levels of arms |
| Composability | ❌ Terminal (catch ends the chain) | ✅ Combinators compose fluently |

The gap is not in semantics — both provide exhaustive handling and
compiler enforcement. The gap is in **LLM token-efficiency**: BAML's
model requires fewer tokens and less nesting for the common case.

Two improvements can close this gap **without** adding `catch` as a
language construct (which would violate Manifesto, Core Inclusion
Filter, and EDR-020):

---

## Improvement 1: Schema Provider — Inferred Error Set Exposure

### Current State

ERROR_UNION (`!T`) infers the error set at compile time, but the
inferred set is invisible in the source code. An LLM reading:

```orthon
fun process(path: String) -> !Output
```

...cannot determine which error tags `process` may produce without
tracing the call graph. EDR-023 § Compliance already mandates:

> *"The Schema Provider must expose inferred error sets for LLM querying."*

This is currently **unimplemented** — no schema format, no query
protocol, no tooling.

### Proposed Design

The Schema Provider exposes error sets via a queryable interface:

```text
$ orthon schema error-set process
{FileNotFound, AccessDenied, ParseError, Timeout}
```

For agent tooling (equivalent to BAML's `baml describe`):

```text
$ orthon describe process
process(path: String) -> !Output
  error set: {FileNotFound, AccessDenied, ParseError, Timeout}
  dependencies: fs.read_file, parse_config, verify_access
```

And in editor/IDE hover:

```orthon
fun process(path: String) -> !Output
  # ^ error set: {FileNotFound, AccessDenied, ParseError, Timeout}
```

### Integration with LLM Toolchain

The Schema Provider makes error sets **machine-readable** — an LLM
agent can query a function's error set before generating a `match` or
combinator chain, reducing hallucination:

```text
LLM (generating code for process call site):
  → queries schema error-set process
  → receives {FileNotFound, AccessDenied, ParseError, Timeout}
  → generates exhaustive match or appropriate combinator chain
```

### What It Changes

| Artifact | Change |
|----------|--------|
| `what/concepts/ERROR_UNION.md` | Add Schema Provider exposure to Default Strategy |
| `how/architecture/ARCHITECTURE.md` § LLM Toolchain | Define Schema Provider query protocol for error sets |
| Implementation repo | Build `orthon schema error-set <function>` CLI command |

---

## Improvement 2: Convenience Combinators — Catch-by-Type as Library

### Current State

The common case LLMs need to generate — "call a fallible function and
handle each error type differently" — requires:

```orthon
match process(path):
    Ok(output) -> handle(output)
    Error(e) -> match e:
        FileNotFound  -> create_default()
        AccessDenied  -> escalate(e)
        ParseError    -> retry()?
        Timeout       -> retry()?
```

This is 8 lines of nesting for a pattern that could be 4.

### Proposed Combinators

Add convenience combinators to the Standard Library that collapse the
two-level `match` into a single pattern dispatch:

```orthon
# handle() combinator — full case analysis (both success and error arms)
process(path).handle(
    on_ok:    fn(Output) -> Result<Processed, Error>,
    on_error: fn(ErrorTag) -> Result<Processed, Error>
) -> Result<Processed, Error>

# Usage:
process(path).handle(
    FileNotFound  -> create_default(),
    AccessDenied  -> escalate(e),
    ParseError    -> retry()?,
    Timeout       -> retry()?,
)
```

```orthon
# recover() combinator — error-side only, returns the Ok value mapped
process(path).recover(
    FileNotFound  -> create_default(),
    AccessDenied  -> escalate(e),
).map(|output| handle(output))
```

```orthon
# catch() combinator — named after the concept, library not keyword
process(path).catch(
    FileNotFound  -> create_default(),
    AccessDenied  -> escalate(e),
).unwrap()
```

### Core Inclusion Filter

These combinators pass at **Level 3 (Standard Library)**:

| Level | Test | Result |
|-------|------|--------|
| L3 Library | Can be expressed as StdLib fn using existing primitives? | ✅ Yes — `match` + pattern matching |
| L2 Pattern | Is it a composition of existing primitives? | ✅ Yes — documented as idiomatic pattern |
| L1 Primitive | Is it an atomic operation? | ❌ No — decomposable into `match` |
| L0 Data Model | Changes semantic foundation? | ❌ No |

No language change required. No new keyword. No Manifesto violation.

### Requirements

1. **Must be expressible using existing Core primitives** — `match`, pattern matching, sum types. If the combinator requires compiler magic (e.g., special-case pattern dispatch on error types), it fails the Library test.
2. **Must support exhaustive checking** — the combinator should work with the compiler's existing exhaustiveness checker on the internal `match`.
3. **Named before symbolic** — each combinator has a named function equivalent.

### LLM Generability Impact

```orthon
# Before (8 lines, 2 levels of nesting)
match process(path):
    Ok(output) -> ...
    Error(e) -> match e:
        FileNotFound -> ...
        AccessDenied -> ...

# After (4 lines, 1 level)
process(path).handle(
    FileNotFound -> ...,
    AccessDenied -> ...,
)
```

Token-equivalent to BAML's `catch (error) { Type => ... }` without
adding a language keyword.

---

## Relationship to LLM_GENERABILITY_GATE

Both improvements directly address LLM_GENERABILITY_GATE criteria
(see [`notes/llm-generability-gate.md`](llm-generability-gate.md)):

| Criterion | Current Score | With Schema Provider | With Combinators |
|-----------|:---:|:---:|:---:|
| Schema-serializable | ⚠️ Error sets inferred but not exposed | ✅ Queryable via schema CLI | — |
| Predictable generation | ⚠️ Nested match is error-prone | — | ✅ Flat dispatch |
| No hallucination surface | ⚠️ LLM may forget Error variant | ✅ Schema query prevents | ✅ Combinator structure guides |
| Self-correctable | ⚠️ Compiler catches missing arms | — | ✅ Same compiler coverage |

---

## Decision

No formal decision required — both improvements are **implementation
concerns**, not language specification changes.

- **Schema Provider** is already mandated by EDR-023 (Compliance item 2).
  This note documents the specific LLM generability requirement.

- **Convenience combinators** are Standard Library additions (Level 3),
  implementable without EDR. The `what/concepts/ERROR_HANDLING.md` §
  Result Combinators table can be extended when the implementation repo
  exists.

The key constraint: **do not add `catch` as a language keyword**. Both
improvements achieve the LLM ergonomics goal without violating Orthon's
design principles.
