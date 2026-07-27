# Error Union

> **✅ ACCEPTED — [EDR-023](../how/decision_records/architecture/EDR-023-error-union.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`ERROR_HANDLING.md`](ERROR_HANDLING.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Error Union, Error Set, Error Tag,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

For the common case where errors are simple tags — file not found, access denied, parse failure — without payload data, forcing the programmer to declare, maintain, and convert between explicit error enums adds ceremony without corresponding semantic value.

The core problem: **error types should grow and shrink automatically as the implementation changes**, not require manual sync between the function body and the error type declaration.

Error Union solves this by making the error side of a fallible function an inferred, structurally-widening set of unit-like error tags. The compiler discovers every fallible call in the function body and computes the minimal error set automatically.

## Principles

1. **Tag-only errors represent the common case** — Most errors are simple identifiers (FileNotFound, Timeout, ParseError) without associated data. Payload-bearing errors are the exception, handled via explicit `Result<T, E>`.
2. **Inferred error sets stay correct by construction** — The error set is derived from the code, not authored. It grows and shrinks automatically as implementation changes.
3. **Explicit `Result<T, E>` for payload-bearing errors** — When an error needs associated data (line numbers, field names, validation details), use `Result<T, E>` explicitly.
4. **One propagation operator** — The `?` operator handles both Error Union and `Result<T, E>` propagation with identical short-circuit semantics. No `try`/`catch` keywords.
5. **Coexistence, not replacement** — Error Union does not replace `Result<T, E>`. Both representations coexist for different use cases.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Error Set Inference Policy | Determines how the compiler discovers and unions error tags across the call graph |
| Propagation Policy | Governs how `?` interacts with Error Union vs. `Result<T, E>` |
| Widening Policy | Controls implicit structural coercion between error sets (always allowed for subsets) |
| Schema Policy | Determines how inferred error sets are exposed via the Schema Provider for LLM querying |

## Model (What)

### Error Union Type `!T`

A fallible function returning `!T` indicates it can produce a tag-only error in addition to the success value of type `T`. The error set is inferred by the compiler.

```orthon
fun read_config(path: String) -> !Config
    let data = fs.read_file(path)?    # error set inferred from read_file
    return parse_config(data)          # error set merged from parse_config
```

### Error Tags

Error values are unit-like tags with no payload data:

```orthon
error.FileNotFound
error.AccessDenied
error.ParseError
error.Timeout
```

### Propagation with `?`

The `?` operator unwraps a `!T` value: if the value is an error, the error is returned immediately from the enclosing function; if successful, the `T` value is unwrapped.

```orthon
fun load_config() -> !Config
    let raw = fs.read_file("config.toml")?
    let parsed = parse_toml(raw)?
    return parsed
```

### Error Set Inference

The compiler infers the error set for each fallible function by:
1. Collecting every `?` operator whose result type includes error tags
2. Computing the union of all encountered error tags
3. The inferred set is the minimal set of error tags that can be produced by the function

```orthon
fun open_log() -> !File
    let f = fs.open("log.txt")?        # adds error.FileNotFound
    let locked = f.acquire_lock()?     # adds error.LockFailed
    return locked                        # error set = {FileNotFound, LockFailed}
```

### Structural Widening

A narrower error set coerces implicitly into any superset. No explicit `From` or `Into` conversion required.

```orthon
fun inner() -> !Result
    # error set = {NotFound, Timeout}

fun outer() -> !Result
    let r = inner()?                     # implicit widening: {NotFound,Timeout} ⊂ {NotFound,Timeout,AccessDenied}
    let verified = verify(r)?
    return verified
```

### `anyerror` Escape Hatch

A universal error supertype for boundaries where precise error set tracking is impractical:

```orthon
fun plugin_process(data: Bytes) -> anyerror!Processed
    # accepts any error set from plugin implementations
```

### Coexistence with `Result<T, E>`

When an error needs payload data, use `Result<T, E>` explicitly:

```orthon
fun parse_json(input: String) -> Result<Json, ParseError>
    where ParseError has line: Int, column: Int, message: String
```

The `?` operator handles both:

```orthon
fun process() -> !Output
    let json = parse_json(raw)?          # Result propagated via ? 
    let saved = db.store(json)?          # Error Union propagated via ?
    return saved
```

## Default Strategy

Error sets are always inferred — the programmer writes `!T` and the compiler computes the error set from the function body. Explicit error set declaration is available as an opt-in for documentation purposes. Structural widening is unconditional — any subset coerces into any superset. The Schema Provider exposes inferred error sets.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Explicit error set declaration | Programmer declares the error set explicitly (e.g., `!error{FileNotFound,AccessDenied}T`) for documentation and contract enforcement |
| Blocked widening | Widening requires an explicit combinator at the call site — preserves local error set visibility at the cost of boilerplate |
| No `anyerror` | Removes the universal escape hatch — all error sets must be precisely trackable. More restrictive but eliminates the "catch-all" anti-pattern |
| Error Union only (no `Result<T, E>`) | Single error model — all errors are tags, no payloads. Simpler but loses payload-bearing error capability |

## Open Questions

1. Should explicit error set declaration syntax be `fn foo() -> error{Foo, Bar}!T` or `fn foo() -> !T where errors: {Foo, Bar}`?
2. How does `anyerror` interact with the Schema Provider — should `anyerror` be flagged in LLM tooling?
3. Should error set inference cross module boundaries, or is each module's error set independently inferred?
4. What error representation does the Standard Library use — Error Union for simple cases, `Result<T, E>` for payload-bearing, or a hybrid?

## Decision History

- **Error Union adopted as primary error representation** over `Result<T, E>`-only model. Rationale: Inferred error sets eliminate error-enum boilerplate for the common case. Structural widening eliminates conversion boilerplate. Coexistence preserves payload-bearing error capability.
- **`?` as unified propagation operator** over `try`/`catch`. Rationale: One operator, one semantics. Avoids "one concept, one syntax" violation.
- **Tag-only errors** adopted over payload-bearing error tags. Rationale: Payload-bearing errors use `Result<T, E>` explicitly. Separation of concerns keeps each representation simple.
- **Accepted via EDR-023** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/concepts/ERROR_HANDLING.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
