# Error Representation Unification (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised on 2026-10-21 by the author while reviewing the contracts work: the
> error-union type `!T` stands out from `Option<T>` and `Result<T, E>`, and the
> symbol `!` has several meanings that must be resolved. Proposes one error
> representation, `Result<T, E>`, where `E` is any type (including a union of
> types) and may be inferred. Home: `how/concepts/research/essential/` —
> semantic hypothesis (essential tier). Its notation questions are registered by
> pointer in the `how/syntax/` decision queue (EDR-087).
>
> **Last updated:** 2026-10-21
>
> **Related:** [`ERROR_UNION.md`](../../../../what/concepts/ERROR_UNION.md) (EDR-023),
> [`ERROR_HANDLING.md`](../../../../what/concepts/ERROR_HANDLING.md) (EDR-020),
> [`UNION_INTERSECTION_TYPES.md`](../../../../what/concepts/UNION_INTERSECTION_TYPES.md) (EDR-045),
> [`NULL_SAFETY.md`](../../../../what/concepts/NULL_SAFETY.md),
> [`SYSTEM_ERROR.md`](../../../../what/concepts/SYSTEM_ERROR.md) (EDR-089),
> [`TRAITS.md`](../../../../what/concepts/TRAITS.md) and
> [`GENERICS.md`](../../../../what/concepts/GENERICS.md) (negative bounds, open),
> [`COPY_ON_WRITE.md`](../../../../what/concepts/COPY_ON_WRITE.md) (`!Send`, `!Sync`),
> [`../../../../what/SYNTAX.md`](../../../../what/SYNTAX.md) (Syntax Principles),
> [`../../../DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md),
> [`TRAIT_CONTRACT_UNIFICATION.md`](TRAIT_CONTRACT_UNIFICATION.md) (a consumer: its rule D2 does not depend on the notation)

---

## Problem

1. **Three representations side by side.** `Option<T>` (a value may be absent), `Result<T, E>` (an operation may fail, with data) and the error union `!T` (an operation may fail, with tag-only errors and an inferred set). The first two are ordinary generic types; `!T` is the only one written with a prefix symbol, so it stands out.
2. **The symbol `!` carries several meanings.**

   | Where | Meaning of `!` |
   |---|---|
   | `-> !Config` ([`ERROR_UNION.md`](../../../../what/concepts/ERROR_UNION.md)) | a type constructor: "`Config` or an error" |
   | `user!` ([`NULL_SAFETY.md`](../../../../what/concepts/NULL_SAFETY.md)) | postfix forced unwrap of an `Option`, panics on `None` |
   | `!Send`, `!Sync` ([`COPY_ON_WRITE.md`](../../../../what/concepts/COPY_ON_WRITE.md)); `!Hash`, `!FixedSize` (open questions of `TRAITS.md` and `GENERICS.md`) | a negative bound: "is not" |

   Boolean negation is the word `not`, so none of these is the usual meaning of `!`. This conflicts with Syntax Principle 2 ("one symbol → one meaning") and with Semantic Purity.
3. **Two mechanisms for one concern.** [`ERROR_HANDLING.md`](../../../../what/concepts/ERROR_HANDLING.md) § "Relationship with Error Union" already says that `Result<T, E>` supports error types that are union types, and shows `Result<Data, IOError | ParseError>` where the combined error type is the union of the sources. So composing several error sources is possible with `Result` today. What `Result` lacks is inference of `E` and a short form for tag-only errors; the error union provides exactly those. The two also define `?` differently: `ERROR_HANDLING.md` says `?` works on `Result` only, while `ERROR_UNION.md` makes it propagate both.

## Hypothesis

**One representation for fallible results: `Result<T, E>`.** `E` is any type, including a union of types; a tag-only error is a type with no fields. A function may omit `E`, and the compiler infers it as the union of the error types of the `?` operands in the body (the set inference of EDR-023). Widening, a narrower set accepted where a wider one is expected, is the assignability of union types (EDR-045). `anyerror` becomes a top error type. The symbol `!` is no longer a type constructor.

Consequences claimed (to be tested, not assumed):

1. Three representations become two (`Option`, `Result`), and `Option` stays a different concept (absence, not failure).
2. One of the three meanings of `!` disappears.
3. Tag-only errors keep their ergonomics: no error enum is declared, because `E` is inferred.
4. Errors with data and tags mix in one union, which addresses the complexity that EDR-023 gave as the reason to reject a unified `!T` with payloads.
5. `?` has one definition.

## Model (What)

> Syntax in this section is **provisional**. The notation of the inferred `E`, the declaration of tag types and the name of the top error type belong to the syntax pipeline.

| Today | Proposed |
|---|---|
| `-> !Config` | `-> Result<Config>`: `E` inferred |
| `-> error{NotFound, Io}!Value` (explicit set, syntax open in `ERROR_UNION.md`) | `-> Result<Value, NotFound \| Io>` |
| `-> anyerror!Processed` | `-> Result<Processed, Error>`: a top error type, name provisional |
| the tag `error.FileNotFound` | a type with no fields named `FileNotFound` (declaration form open) |
| `-> Result<Json, ParseError>` | unchanged |

**Inference** carries over from EDR-023: the compiler collects the error type of every `?` operand and takes the union; the inferred type is the minimal one.

**Widening.** `Result<V, A>` is accepted where `Result<V, A | B>` is expected. This follows from "`A` is assignable to `A | B`" ([`UNION_INTERSECTION_TYPES.md`](../../../../what/concepts/UNION_INTERSECTION_TYPES.md)) and requires `Result` to be covariant in `E`; the variance of `Result` is computed by position ([`GENERICS.md`](../../../../what/concepts/GENERICS.md)) and has to be confirmed.

**What does not change:** `Option<T>` with `?.`, `??`, `.or()`; the combinators of `Result`; the boundary between domain errors (representable as values) and system errors ([`SYSTEM_ERROR.md`](../../../../what/concepts/SYSTEM_ERROR.md)).

## Examples by usage context

Syntax is provisional.

```orthon
// 1. E inferred: a union of the tags that the body can produce
fun read_config(path: String) -> Result<Config>
    let data = fs.read_file(path)?          // may produce FileNotFound
    return parse_config(data)               // may produce ParseError

// 2. E explicit, with data
fun parse(input: String) -> Result<Json, ParseError>

// 3. E explicit, a union of tags
fun load(key: String) -> Result<Value, NotFound | Io>

// 4. Widening in a call chain: a subset is accepted where a superset is expected
fun inner() -> Result<Value, NotFound>
fun outer() -> Result<Value, NotFound | AccessDenied>
    let v = inner()?
    return verify(v)

// 5. Tag and data in one union
fun doc() -> Result<Doc, ParseError | NotFound>

// 6. A top error type where precise tracking is impractical (replaces anyerror)
fun plugin(d: Bytes) -> Result<Processed, Error>

// 7. Handling: narrowing over the members of the union
match read_config(path)
    Ok(cfg)          -> use(cfg)
    Err(FileNotFound) -> default_config()
    Err(e)           -> report(e)
```

Edge and incorrect cases:

```orthon
fun pure_length(s: String) -> Result<Int>   // WARNING: the inferred E is empty; return Int
fun a() -> Result<Int, NotFound>
    return b()?                              // ERROR if b can produce Io: Io is not in NotFound
```

## Evidence

### In the repository

- [`ERROR_HANDLING.md`](../../../../what/concepts/ERROR_HANDLING.md) (EDR-020) already allows a union as the error type of `Result`; the proposal extends it with inference.
- EDR-023 rejected "`Result<T, E>` only" because it forces an error enum for tag-only errors (ceremony). With inferred `E` and tags as types that reason no longer applies.
- EDR-023 rejected a "unified `!T` with payload-bearing errors" as complex when a set mixes tags and payloads. Union types give a defined semantics for such a mix.
- The alternatives table of EDR-023 does **not** list "`Result` with a union and inferred `E`". EDR-020, EDR-023 and EDR-045 were all accepted on 2026-07-27, so there is no argument from dates; the argument is an alternative that the record does not contain.
- `NULL_SAFETY.md`, `TRAITS.md`, `GENERICS.md`, `COPY_ON_WRITE.md` show the other meanings of `!` (table in Problem).

### Not verified (to be surveyed before an EDR)

- How other languages combine inferred error sets with ordinary result types (Zig error unions, Rust `Result` with conversions, Swift typed throws) is from general knowledge and has not been searched.
- Whether `Result` is covariant in `E` under the position-based variance rule of `GENERICS.md`.

## Alternatives

| ID | Alternative | Notes |
|----|-------------|-------|
| A0 | Keep EDR-023 as accepted: `!T` beside `Result<T, E>` | three representations; the multiple meanings of `!` remain |
| A1 | **This hypothesis:** `Result<T, E>` with union and inferred `E`; `!T` retired | one representation; needs an amendment of EDR-023 |
| A2 | Keep `!T` as sugar for `Result<T>` | two spellings for one concept, against Syntax Principle 1 unless `!T` is removed |
| A3 | Change only the symbol (a word instead of `!`) | resolves the symbol clash but keeps three representations |
| A4 | `Result<T, E>` with an explicit `E` everywhere | rejected by EDR-023: ceremony for tag-only errors |

## Impact and cascade

- Accepted records affected if A1 is chosen: EDR-023 (superseded in part), EDR-020 (gains inference of `E`, one definition of `?`), EDR-089 (the text "`Result<T,E>` / `!T`").
- Documents: 19 files under `what/` and `how/` mention the error union or `!T` in some form (counted on 2026-10-21, excluding the contract hypotheses, `INDEX.md` and `DECISION_LOG.md`); the notation itself appears in about ten lines of two files (`GLOSSARY.md`, `ERROR_UNION.md`). `error.X` tags appear in 21 places.
- This re-opens an accepted decision (Tier 2). Under the Transparency gate guard the record states what is new: an alternative absent from the EDR-023 table, and `ERROR_HANDLING.md` already admitting union error types.

## Open Questions

1. **Notation of the inferred `E`:** omit it (`Result<Config>`) or use a placeholder.
2. **Tags as types:** the declaration form, and the relation to the `error.X` syntax.
3. **The top error type:** name and meaning (replaces `anyerror`).
4. **Variance** of `Result` in `E` and the assignability `Result<V, A>` to `Result<V, A | B>`.
5. **Inference across module boundaries** (open in `ERROR_UNION.md`) and for recursive functions.
6. **An empty inferred set:** error or warning.
7. **One definition of `?`:** `ERROR_HANDLING.md` says `Result` only, `ERROR_UNION.md` says both.
8. **The remaining meanings of `!`:** postfix forced unwrap and prefix negative bound. A separate decision; a candidate for negative bounds is the word `not` (`not Send`), consistent with Named Before Symbolic and with boolean `not`.
9. **Standard Library error representation** (open in `ERROR_UNION.md`) becomes simpler if A1 is chosen.
10. **Contracts:** the rule D2 of the contract review ("the result of an implementation is assignable to the result of the contract member") does not depend on the notation.

## Next steps (route through the pipeline)

1. Decision Pipeline run (10 questions), then the Concept Design Review (8 steps). The contract review is paused by the author's decision of 2026-10-21; this hypothesis is independent of it.
2. The notation questions go through [`SYNTAX_PIPELINE.md`](../../../SYNTAX_PIPELINE.md); the coupling is registered in the [`how/syntax/`](../../../syntax/README.md) decision queue.
3. If accepted: an EDR (Tier 2) superseding EDR-023 in part and amending EDR-020; the cascade becomes a tracked task.

## Human Sign-off

> **Reviewed-by:** — · **Date:** — · **Verdict:** pending
> Required before any EDR may be filed (Concept Design Review Step 7). Agents and GSD flows must not self-certify this field.

---

### Affected Documents

- [ ] `what/concepts/ERROR_UNION.md` (EDR-023)
- [ ] `what/concepts/ERROR_HANDLING.md` (EDR-020)
- [ ] `what/concepts/SYSTEM_ERROR.md` (EDR-089)
- [ ] `what/concepts/TYPE_INFERENCE.md`
- [ ] `what/CORE_CONCEPTS.md`, `what/GLOSSARY.md`, `what/PRIMITIVE_BLOCKS.md`, `what/LIBRARY_BOUNDARY.md`, `what/THESES.md`
- [ ] `what/concepts/NULL_SAFETY.md`, `TRAITS.md`, `GENERICS.md`, `COPY_ON_WRITE.md` (other meanings of `!`)
- [ ] `how/decision_records/INDEX.md`

## See also

- [`ERROR_UNION.md`](../../../../what/concepts/ERROR_UNION.md), [`ERROR_HANDLING.md`](../../../../what/concepts/ERROR_HANDLING.md), [`UNION_INTERSECTION_TYPES.md`](../../../../what/concepts/UNION_INTERSECTION_TYPES.md)
- [`CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md), [`SYNTAX_PIPELINE.md`](../../../SYNTAX_PIPELINE.md)
