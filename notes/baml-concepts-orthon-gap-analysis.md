# BAML Concepts — Orthon Gap Analysis

> Analysis of BAML language concepts missing from Orthon that are worth
> adopting for an LLM-native language. Based on review of
> [boundaryml.com/explore](https://boundaryml.com/explore) and BAML BEPs.
>
> **Date:** 2026-07-28

---

## Tier 1 — Language-Level Gaps (most impactful)

### 1. LLM Calls as Native Functions (P0 — #1 Gap)

This is BAML's defining feature. An LLM call is just a function: the
prompt is the body, the return type *is* the structured output schema,
and the compiler validates both.

```baml
class Resume {
  name: string,
  email: string?,
}

function extract_resume(text: string) -> Resume {
  client: "openai/gpt-4o-mini"
  prompt: `
    Extract the resume.
    ${ctx.output_format}
    ${text}
  `
}
```

It composes with ordinary code — LLM calls participate in type-checking,
tracing, testing, and optimisation like any other function.

**Orthon status:** Orthon has the LLM Generability Gate (validating that
LLMs can *produce* Orthon code) but no concept for Orthon code *calling*
LLMs. VISION.md says "LLMs as first-class code generators" but not "LLMs
as first-class runtime services." This is a missing semantic dimension.

**Recommendation:** Introduce an LLM-calling primitive (or `@llm`
attribute on functions) that treats prompts as typed functions returning
structured data. Essential-tier concept.

---

### 2. Literal Types — "Make Undesired States Unrepresentable" (P0)

BAML elevates this to a core design philosophy:

```baml
class Ticket {
  status: "open" | "closed",   // "opne" won't compile
}
```

An agent sampling tokens will eventually produce an invalid state. If
that state cannot compile, it cannot ship.

**Orthon status:** `LITERAL_TYPES` is accepted (EDR-043), but the
"undesirable states unrepresentable" philosophy was not previously named
as an explicit cross-cutting concern. Research file created at
`how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`
(2026-07-29) to document the pattern and its relationship to EDR-039
(ADTs), EDR-043 (Literal Types), EDR-018/028 (Null Safety), and
EDR-045 (Union Types).

**Recommendation (updated):** Keep the pattern as a documented
cross-cutting concern in `essential/`. Elevating it to a design principle
in `DESIGN_PRINCIPLES.md` would require a Tier 1 EDR, since that document
is locked. The current approach — a dedicated research file linking the
relevant EDRs — is the correct intermediate step.

---

### 3. Function-Coloring-Free Concurrency (P1)

BAML follows Go's model — any function can be spawned, no `async` modifier:

```baml
function main() -> int {
  let a = spawn { work(1) };   // no 'async function' needed
  let b = spawn { work(2) };
  (await a) + (await b)
}

function work(i: int) -> int { i * i }   // plain function
```

**Orthon status:** `ASYNC_AWAIT` is accepted (EDR-046) with `async` as an
explicit modifier. CONCURRENCY_MODEL uses delegate/actor model with
message passing.

**Why this matters for LLMs:** Function coloring creates a bifurcated
world — LLMs must track whether each function is async or not. Go-style
eliminates that tracking cost.

**Recommendation:** Reconsider whether ASYNC_AWAIT should require the
`async` modifier, or follow BAML/Go's coloring-free approach.

---

### 4. Filesystem as Namespace — No Imports (P1)

BAML uses `ns_` directory prefixes. The directory tree IS the module
system. No `import` statements.

```text
baml_src/
├── ns_catalog/
│   └── product.baml
└── ns_orders/
    └── order.baml
```

Definitions within a namespace are available; cross-namespace access uses
fully qualified names (`root.catalog.Product`).

**Orthon status:** `CONTEXT_LIMITED_MODULES` is accepted (EDR-062) but is
a conventional module system with explicit imports.

**Why this matters for LLMs:** Import management is a major source of LLM
context pollution. Agents grep for imports, add wrong ones, miss needed
ones, and go on "side quests across the codebase" (BAML's phrase).
Eliminating imports eliminates that entire class of error.

**Recommendation:** Consider a namespace-via-filesystem model as an
alternative to explicit imports, or at minimum ensure `baml describe`-style
tooling exists in the implementation repo.

---

### 5. Tests and Testsets as First-Class Language Constructs (P1)

BAML treats tests as code living in any `.baml` file:

```baml
test "greet returns greeting" {
  assert.equal(greet("world"), Greeting { message: "hi, world" })
}

testset "from a csv" {
  let rows = load_csv("data.csv");
  for (let row in rows) {
    test ("classify: " + row.text) {
      assert.equal(classify(row.text), row.expect)
    }
  }
}

test "tolerates flaky runs" with quorum {   // custom runner
  assert.is_true(check_inventory())
}
```

Data-driven tests, quorum tests for nondeterministic functions, custom
runners — all part of the language.

**Orthon status:** No testing model exists. Not in any tier.

**Why this matters for LLMs:** When LLMs generate code that calls
nondeterministic services (LLMs!), `assert output == expected` is broken.
BAML's quorum tests and custom runners address this directly.

**Recommendation:** Add a testing model as an Essential or Important-tier
concept.

---

### 6. Typed Error Handling via `throws` with Inferred Error Sets (P2)

BAML infers the error set from `throws` declarations and checks
exhaustiveness of catch arms:

```baml
function show(ok: bool) -> string {
  fetch_page(ok) catch (error) {
    NetError => "recovered: " + error.detail,
    // Compiler warns: ParseError arm is unreachable
    ParseError => "unreachable",
  }
}
```

**Orthon status:** `Result<T,E>` and `ERROR_UNION` (`!T`) are accepted.
Orthon takes the Rust/Zig path (return-based); BAML takes the
checked-exceptions path with type-based catch arms.

**Why this matters for LLMs:** BAML's approach surfaces error types in
catch arms directly — LLMs don't need to pattern-match on `Result`
variants. "Catch by type" may be more natural for LLMs.

**Recommendation:** Design comparison EDR. Orthon's `Result` approach is
more principled (errors are values), but BAML's approach is arguably
simpler for LLMs to generate correctly.

---

### 7. Nondeterminism as a First-Class Semantic Concern (P1)

BAML's philosophy: "Trace nondeterminism." An LLM call anywhere in the
call graph makes everything above it nondeterministic. BAML surfaces this
in tooling (profiler, workflow view, trace replay).

**Orthon status:** Orthon's semantic model assumes determinism
(Deterministic Behavior principle in DESIGN_PRINCIPLES.md). The
EXECUTION_MODEL doesn't address nondeterminism.

**Why this matters for LLMs:** An LLM-native language will have LLM
calls, and LLM calls are nondeterministic. The semantic model should
account for this.

**Recommendation:** Add a nondeterminism dimension to the semantic model
or execution model, or at minimum document how the deterministic model
composes with nondeterministic LLM calls.

---

## Tier 2 — Tooling/Ecosystem Gaps (LLM Agent UX)

These aren't language concepts per se, but they're what makes BAML
*usable by agents*. They belong in the implementation repo architecture,
not in the language spec.

### 8. `baml describe` — AST-Aware Discovery

Agents query a symbol → get definition, dependencies, and all call sites
without reading files into context. More efficient than LSP for agents.

### 9. `baml run <function>` — Every Function as CLI

Agents invoke any function directly. No test harness, no `main()`
wrapper. Transformative for LLM write-run-observe loops.

### 10. `baml run -e` — Inline Execution

```sh
$ baml run -e '"a,b,c".split(",")'
["a", "b", "c"]
```

Agents don't need to create files for experiments.

### 11. `baml pack` — Function as Standalone Binary

Ships selected functions as a 12 MB CLI binary. LLM-generated utilities
become deployable artifacts with one command.

### 12. Typed `eval` (planned in BAML)

Compiles LLM-generated source against an expected type signature, returns
typed compiler errors that feed back to the agent:

```baml
let callback = package.build().get<() -> string>("hello");
```

### 13. Scoped Function Mocking (planned in BAML)

Replace dangerous functions within a lexical scope for sandboxing
LLM-generated code.

---

## Summary: What to Prioritize

| Priority | BAML Concept | Gap in Orthon | Action |
|----------|-------------|---------------|--------|
| **P0** | LLM calls as native functions | No LLM-calling primitive | New Essential-tier concept |
| **P0** | Literal types → "undesired states unrepresentable" | LITERAL_TYPES is Important-tier | Elevate to Essential, make it a design principle |
| **P1** | Function-coloring-free concurrency | ASYNC_AWAIT requires `async` modifier | Revisit ASYNC_AWAIT design |
| **P1** | Filesystem-as-namespace (no imports) | CONTEXT_LIMITED_MODULES with imports | Consider import-free namespace model |
| **P1** | Tests as first-class code | No testing model | New concept |
| **P1** | Nondeterminism as semantic concern | Deterministic Behavior principle | Add nondeterminism dimension |
| **P2** | Typed `throws` + catch-by-type | `Result<T,E>` + `!T` (different approach) | Design comparison EDR |
| **P2** | `describe` / `run` / `pack` / `run -e` tools | No agent tooling | Architecture note (future repo) |
| **P3** | Typed `eval` + scoped mocking | AST_MACROS only (compile-time) | Defer to implementation repo |

---

## Key Finding

**Orthon has excellent LLM *generability* (LLMs can write Orthon) but
zero LLM *integration* (Orthon can't call LLMs).** BAML closes that loop.
For Orthon to be a true LLM-native language, it needs at minimum:

1. An LLM-calling primitive (concept #1 above)
2. "Undesired states unrepresentable" elevated from Important to
   Essential-tier design philosophy (concept #2 above)

---

## Reference

- BAML Explore: <https://boundaryml.com/explore>
- BAML BEPs: <https://beps.boundaryml.com>
- BAML Docs: <https://docs.boundaryml.com>
