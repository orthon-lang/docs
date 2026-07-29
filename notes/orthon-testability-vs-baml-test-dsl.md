# Orthon Testability vs BAML Test DSL

> **Date:** 2026-07-29
> **Context:** Comparison of Orthon's built-in testability (via language design) vs BAML's `test`/`testset`/`quorum` syntax approach. Derived from the BAML gap analysis discussion.

---

## Core Insight

Orthon already achieves most of what BAML's test DSL provides, but through **language architecture**, not special syntax. The key mechanisms:

### 1. `fun`/`proc`/`new` — Purity by Declaration

| Kind | Effect | Return | Testability Consequence |
|------|--------|--------|------------------------|
| `fun` | Read-only, never mutates | Always returns a value | **Pure by declaration** — test without setup, assert without surprises |
| `proc` | Mutates `self`, identity preserved | May return value or nothing | **Mutation explicitly marked** — mock only where needed |
| `new` | Creates new value, identity changes | Always returns new value | **Transformation explicit** — no hidden identity mutation |

A `fun` call:
- Has no hidden side effects
- Depends only on its inputs
- Can be tested with `assert result == expected` and nothing else

### 2. Context Parameters (`using`) — Dependencies Extracted from `()`

**EDR-037** (Accepted, SEMANTIC_MODEL correction) splits parameters:

```
fun process_order(order_id: Int) using db: Database, logger: Logger -> Receipt
#              ↕ data                     ↕ dependencies
#       "what is computed?"       "in what environment?"
```

- `()` contains **only data** — the subject of computation
- `using` contains **dependencies** — the environment
- Compiler resolves `using` automatically via `given` instances
- No DI framework, no reflection, no manual threading

**Testing consequence:** A test for `process_order`:
```orthon
fun test_process_order():
    let mock_db = MockDatabase()
    let mock_log = MockLogger()
    let result = process_order(42) using mock_db, mock_log
    assert result.status == "confirmed"
```
Dependencies are explicit, injectable, and compiler-verified.

### 3. `==` Has One Meaning

Structural equality only — no overloaded `==`. Tests always compare what they say.

---

## Comparison: BAML vs Orthon

| Aspect | BAML (test-as-syntax) | Orthon (testability-through-design) |
|--------|----------------------|--------------------------------------|
| **Dependency isolation** | Needs external DI/mock framework | `using` — built-in DI at language level |
| **Function purity** | No language guarantee | `fun` — compiler-enforced purity |
| **Test syntax** | `test`/`testset`/`quorum` blocks | Ordinary `fun` declarations |
| **Data-driven tests** | `testset` + `load_csv` syntax | `for` + sequence — already composable |
| **Custom runners** | `with quorum { ... }` | Potentially via Execution Program |
| **Nondeterminism handling** | `quorum` runner | **Not addressed** (Orthon assumes determinism) |
| **LLM cognitive load** | Must learn test DSL | One model: "tests are just `fun`" |

---

## What Orthon Covers (~70%)

Orthon's `fun` + `using` + `proc`/`new` already eliminates most reasons for a test DSL:

- **Unit tests** → `fun` calling `fun`, asserting results
- **Mocking** → `using` with mock implementations
- **Data-driven tests** → `for` over a sequence of inputs, calling `fun` per item
- **Pure/effect separation** → `fun` vs `proc` tells you what needs mock setup

## What Remains Open (~30%)

1. **Nondeterminism** — If Orthon gets native LLM calls (#1 recommendation from BAML gap analysis), `fun`'s determinism guarantee breaks. Need quorum-style tests for LLM-powered functions.
2. **Test organization** — BAML lets tests live in any file beside production code. Orthon's module system (CONTEXT_LIMITED_MODULES) is more traditional; test placement is unresolved.
3. **`baml run <function>`** — Every function as CLI. Not a language concept but transformative for agent code-test cycles.

---

## Recommendation

Do **not** add `test`/`testset` as language syntax. The composability of `fun` + `using` + sequences already covers the pattern. Instead:

1. **Document the testing pattern** — show how `fun` + `using` composes into a testing idiom without extra syntax.
2. **Add quorum-testing as a concept** only if/when LLM-calling primitives are added (nondeterminism needs it).
3. **Defer test DSL** — the language doesn't need it; a potential `orthon run` tool in the implementation repo can provide the `baml run`-style ergonomics.

---

## References

- BAML gap analysis: `notes/baml-concepts-orthon-gap-analysis.md` §5
- Context Parameters: `how/concepts/research/essential/CONTEXT_PARAMETERS.md`
- EDR-037: `how/decision_records/architecture/EDR-037-context-parameters.md`
- SEMANTIC_MODEL.md: `what/SEMANTIC_MODEL.md` § Mutation
