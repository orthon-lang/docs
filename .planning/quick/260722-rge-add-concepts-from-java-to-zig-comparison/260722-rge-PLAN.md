---
phase: quick-260722-rge
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [how/concepts/research/COMPILE_TIME_EXECUTION.md]
autonomous: true
requirements: []
user_setup: []

must_haves:
  truths:
    - "how/concepts/research/COMPILE_TIME_EXECUTION.md exists as a DRAFT research file matching the DYNAMIC_TYPING.md/EXTENSION_FUNCTIONS.md short-form template (Issue (Why), Examples, Implications for Orthon, Open Questions, Decision History, Cross-References), dated 2026-07-22"
    - "COMPILE_TIME_EXECUTION.md explicitly disambiguates Zig-style unified comptime from GENERICS.md's trait-bound + monomorphisation model, STRUCTURAL_TYPING.md's structural-trait-satisfaction model, METAOBJECTS.md's traits+annotations+macros model, and REFLECTION_ALTERNATIVES.md's reified-generics+derive-macros+sealed-types model — stating in its own sentence that all four solve generics/polymorphism/metaprogramming/reflection as separate mechanisms, whereas Zig's comptime solves all four with one homogeneous mechanism (ordinary code, ordinary control flow, `type` as a first-class value, executed at compile time)"
    - "COMPILE_TIME_EXECUTION.md covers duck-typed generic constraints (no declared trait/interface — constraint is whatever the function body calls) as part of the same concept, not split into a separate file"
    - "COMPILE_TIME_EXECUTION.md does not modify any existing file in the repository (additive only) and does not touch what/GLOSSARY.md"
  artifacts:
    - how/concepts/research/COMPILE_TIME_EXECUTION.md
  key_links:
    - "COMPILE_TIME_EXECUTION.md Issue (Why) names and links GENERICS.md, STRUCTURAL_TYPING.md, METAOBJECTS.md, and REFLECTION_ALTERNATIVES.md, stating the scope difference (four separate mechanisms vs. Zig's one unified mechanism)"
    - "COMPILE_TIME_EXECUTION.md Implications for Orthon links HOMOICONICITY.md (contrasting comptime's 'no separate macro language' with Lisp-style code-as-data and Orthon's Schema Provider) and why/MANIFESTO.md's Minimal Core / One concept-one syntax principles"
---

<objective>
Add one new DRAFT concept research file to `how/concepts/research/` covering the single
language concept from the user's Java-to-Zig comparison table that is genuinely NOT
represented by any existing research file: **compile-time execution as a unified
mechanism** — Zig's `comptime`, which collapses generics, duck-typed polymorphism,
reflection, and metaprogramming into one homogeneous feature (ordinary language code,
ordinary control flow, `type` treated as a first-class value, executed at compile time,
with no separate trait-declaration language, no macro syntax, and no annotation/derive
system).

Every other row in the user's 35-row Java-to-Zig comparison table is already covered by
an existing file under a different name and must NOT get a duplicate file in this plan.
Specifically, during planning the following rows were cross-checked and confirmed
already covered:
- OOP as primary unit / no classes, only struct+enum+union+functions (row 1) →
  `CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`
- Inheritance absent / composition over inheritance (row 2) →
  `COMPOSITION_OVER_INHERITANCE.md`
- Virtual methods / static dispatch by default (row 4) and generics without type
  erasure (row 12) → `GENERICS.md` (already states "Static dispatch by default" and
  "No type erasure" as Orthon principles)
- JVM vs native, class loading, runtime metadata / small binary (rows 5, 19, 20) →
  `NATIVE_COMPILATION.md` (already names Zig explicitly as an AOT-native example)
- Garbage collector / manual memory management / stack-first allocation (rows 6, 7) →
  `ALLOCATION.md` (Allocation Policy, explicit-allocation principle, `no_alloc` mode
  as an open question)
- Exceptions / Error Union / checked exceptions absent (rows 8, 9) →
  `ERROR_HANDLING.md` (already models `Result<T, E>` + `?` operator + "no unchecked
  exceptions — all fallibility is declared")
- Boxing/unboxing absent, unified type system (rows 13, 14) → `DATA_MODEL.md`,
  `VALUE_SEMANTICS.md`
- Optional/null (rows 15, 16) → `NULL_SAFETY.md`
- Access modifiers minimized (row 17) → `VISIBILITY_AND_ENCAPSULATION.md`
- No constructors / struct-literal initialization (rows 21, 22) →
  `OBJECT_INITIALIZATION.md` (already rejects builder boilerplate in favour of
  named-parameter struct construction)
- Destructors / `defer` / `finally` (rows 23, 24) → `DISPOSABLE_PATTERN.md` (already
  lists Go's `defer` in its comparison table)
- No built-in concurrency model, minimal threading abstraction (rows 25, 26) →
  `CONCURRENCY.md`, `COMPILE_TIME_CONCURRENCY_SAFETY.md`
- Automatic reference coercion mostly absent (row 27) → the "Explicit over implicit"
  principle recurring across `ALLOCATION.md`, `GENERICS.md`, `EXPLICIT_COMPOSITION.md`
- Method overloading absent (row 28) → `FUNCTION_OVERLOADING.md` (already rejects
  overloading as the Default Strategy)
- Tagged unions / exhaustive switch (rows 31, 32) → `ALGEBRAIC_DATA_TYPES.md`,
  `ENUM_ALTERNATIVES.md`
- Class/Object as language foundation absent (row 30) → `ROOT_OBJECT.md` (no universal
  base type)
- Minimalistic standard library (row 35) → out of scope for concept research (a
  strategy/scope decision, not a language concept)

The remaining rows (3, 10, 11, 33 — interfaces via duck-typed `comptime`, reflection
replaced by `comptime`, annotations replaced by language-level metaprogramming,
metaprogramming via `comptime`) all point at the same underlying, currently-uncovered
concept: comptime as ONE unified mechanism, rather than Java's or Kotlin's or Rust's
several separate mechanisms for the same needs.

Purpose: This repository's research corpus (100+ files in `how/concepts/research/`) was
cross-checked row-by-row against the comparison table during orchestration, following
the same audit discipline used for the prior Java-to-Ruby comparison. The single
genuine gap — Zig's `comptime` unification — surfaces a consequential question for
Orthon's Minimal Core, One-Concept-One-Syntax, and LLM Generability pillars
(`why/VISION.md`, `why/MANIFESTO.md`) that deserves its own DRAFT research artifact
ahead of Concept Design Review, rather than being silently folded into `GENERICS.md`'s
trait-bound model or `REFLECTION_ALTERNATIVES.md`'s reified-generics model (both of
which retain separate mechanisms for separate needs — the opposite of what makes
Zig's approach distinctive).

Output: `how/concepts/research/COMPILE_TIME_EXECUTION.md`, a DRAFT research document
following the short-form structure established by the most recent "missing language
feature" batch (`DYNAMIC_TYPING.md`, `EXTENSION_FUNCTIONS.md`, `OPEN_CLASSES.md`,
`SINGLETON_CLASS.md`): Issue (Why), Examples (comparison table + short code samples),
Implications for Orthon (numbered), Open Questions, Decision History,
Cross-References.
</objective>

<execution_context>
@$HOME/.claude/gsd-core/workflows/execute-plan.md
@$HOME/.claude/gsd-core/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@how/concepts/research/DYNAMIC_TYPING.md
@how/concepts/research/GENERICS.md
@how/concepts/research/STRUCTURAL_TYPING.md
@how/concepts/research/METAOBJECTS.md
@how/concepts/research/REFLECTION_ALTERNATIVES.md
@how/concepts/research/HOMOICONICITY.md
@why/VISION.md
@why/MANIFESTO.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Write COMPILE_TIME_EXECUTION.md (comptime as a unified mechanism)</name>
  <files>how/concepts/research/COMPILE_TIME_EXECUTION.md</files>
  <action>
    Create the file with title `# Compile-Time Execution (Unified Comptime)`. Directly
    below the title, copy the DRAFT status banner block verbatim in structure and
    wording from `DYNAMIC_TYPING.md`'s header (same warning-emoji glyphs, same
    "exploratory research material for the Concept Design Review process (Milestone 2)"
    wording, same "registered only after acceptance via EDR (Architecture category)"
    line, same syntax note about abstract syntax and Phase 5), but set the "Last
    updated:" line to 2026-07-22.

    Then write these sections in exactly this order, matching `DYNAMIC_TYPING.md`'s
    heading set (`## Issue (Why)`, `## Examples`, `## Implications for Orthon`,
    `## Open Questions`, `## Decision History`, then a horizontal rule and
    `### Cross-References`) — do not use the longer `_concept.md` template (no
    Principles/Policy Footprint/Model/Default Strategy/Alternative Strategies
    sections); this concept is scoped as a short-form research note like
    `DYNAMIC_TYPING.md` and `EXTENSION_FUNCTIONS.md`.

    **`## Issue (Why)`** — Open with the framing question: should generics,
    duck-typed polymorphism, reflection, and metaprogramming each get their own
    dedicated language mechanism, or should a single compile-time execution model
    serve all four needs at once? Describe Zig's `comptime`: any function parameter
    can be declared `comptime`, including parameters whose declared type is `type`
    itself — so `fn max(comptime T: type, a: T, b: T) T` is an ordinary function
    definition, not special generic syntax; there is no `<T>` bracket syntax, no
    separate generic-parameter list, and no declared trait/interface bounding what
    `T` must support. Constraint checking is deferred entirely to instantiation: the
    compiler substitutes the concrete type and simply compiles the function body as
    written; if the body calls `.eq()` on a `T` value and the substituted type has no
    `eq` method, the resulting compile error points at the actual call site inside
    the generic function body, not at a mismatch against a separately declared
    contract. The same `comptime` keyword also drives Zig's reflection story
    (`@typeInfo(T)`, `@field(value, name)`, `@hasDecl(T, name)` are ordinary function
    calls evaluated at compile time, not a separate reflection API) and its
    metaprogramming story (code that generates code is just ordinary `if`/`for`/`while`
    control flow running at compile time — no macro language, no AST manipulation, no
    annotation processor, no `@derive`-style declarative code-generation system).
    Then explicitly name and link `GENERICS.md`, `STRUCTURAL_TYPING.md`,
    `METAOBJECTS.md`, and `REFLECTION_ALTERNATIVES.md`, stating the scope difference
    in its own sentence: each of those four documents already proposes a solution for
    one slice of this problem space (trait-bound monomorphised generics in
    `GENERICS.md`; structural trait satisfaction, still with a declared `trait`
    contract, in `STRUCTURAL_TYPING.md`; traits/annotations/macros as three distinct
    mechanisms in `METAOBJECTS.md`; reified generics in inline functions plus
    `@derive` macros plus sealed-types-and-pattern-matching as three more distinct
    mechanisms in `REFLECTION_ALTERNATIVES.md`) — but none of them evaluates the
    alternative of collapsing all of generics, duck-typed constraint checking,
    reflection, and metaprogramming into the single mechanism Zig demonstrates: one
    execution phase (compile time), one syntax (ordinary function/control-flow
    syntax), one data kind that is new relative to runtime (`type` as a first-class
    value), and zero additional declarative sublanguages (no trait-bound syntax
    needed, no macro syntax needed, no annotation syntax needed). Close with the core
    problem statement: should Orthon adopt Zig's single unified compile-time
    execution mechanism instead of (or alongside) the four separate mechanisms already
    proposed elsewhere in this research corpus, and if the unified approach is
    preferred, does it strictly subsume the other four documents' proposals or leave
    gaps (coherence guarantees, structural-satisfaction ergonomics, IDE/LLM
    discoverability of "what a generic function actually requires") that the four
    separate mechanisms currently handle better?

    **`## Examples`** — A comparison table with columns Language | Generic
    constraint mechanism | Constraint checked against | Reflection mechanism |
    Metaprogramming mechanism | Number of distinct sublanguages, covering: Zig
    (`comptime T: type` parameter, no declared constraint — checked against the
    function body's actual usage at instantiation time; `@typeInfo`/`@field`/
    `@hasDecl` as ordinary compile-time function calls; ordinary `if`/`for`/`while`
    executed at compile time; one — comptime is the only sublanguage, identical
    syntax to runtime code); Rust (trait bounds `where T: Ord`, checked against a
    declared trait; `std::any::TypeId` — limited, mostly opt-in; procedural macros
    with their own token-stream/AST API (`syn`, `quote`); three — trait syntax,
    a separate macro-definition API, and ordinary code); Java (generics via type
    erasure with `extends`/`super` bounds, checked against a declared interface;
    `java.lang.reflect` runtime API; annotation processors with their own API; four
    — generics syntax, reflection API, annotation syntax, and ordinary code); Kotlin
    (generic constraints via `where`, checked against a declared interface; `KClass`
    reflection plus `reified` type parameters in `inline` functions as a special
    case; compiler plugins (`kotlinx.serialization`) with their own API; four —
    generic syntax, reflection API, `reified`/`inline` special case, and compiler
    plugin API); C++ (templates duck-typed structurally like Zig pre-C++20, but
    C++20 `concepts` reintroduce a declared-constraint sublanguage; no first-class
    reflection until the C++26 reflection proposal, itself a new sublanguage;
    template metaprogramming has historically used the type system itself as an
    ad-hoc Turing-complete sublanguage, distinct from ordinary runtime code; three
    or four depending on `concepts`/reflection adoption). Follow the table with a
    short Zig code sample showing `fn max(comptime T: type, a: T, b: T) T` calling
    `a > b` inside the body with no declared bound, and a short contrasting Rust
    sample showing the same function requiring `where T: PartialOrd` — note in
    prose immediately after the Rust sample that `GENERICS.md`'s Default Strategy
    already chose the Rust-style trait-bound approach for Orthon, so this file's
    purpose is to make the road not taken (and its trade-offs) explicit for the
    eventual Concept Design Review, not to silently override that existing choice.

    **`## Implications for Orthon`** — Number these implications: (1) a single
    unified compile-time execution mechanism directly serves the Manifesto's
    "One concept — one syntax" and Minimal Core principles better, in the abstract,
    than four separate mechanisms for four related needs — but `GENERICS.md`,
    `STRUCTURAL_TYPING.md`, `METAOBJECTS.md`, and `REFLECTION_ALTERNATIVES.md` were
    each drafted independently and have already converged on a *different* answer
    (declared trait bounds, monomorphisation, and a small number of well-scoped
    sublanguages) — so adopting the unified `comptime` model now would mean revising
    four existing DRAFT documents' Default Strategy sections at Concept Design
    Review time, not merely adding a fifth; (2) the core trade-off Zig accepts and
    the four existing documents implicitly reject is **discoverability**: with
    declared trait bounds (`GENERICS.md`'s model), a reader (human or LLM) can see a
    generic function's full contract without reading its body; with Zig-style
    duck-typed `comptime`, the contract is implicit in the function body and only
    surfaces as a compile error at a specific call site — this cuts directly against
    the LLM Generability pillar's "Small surface, consistent rules" framing in
    `why/VISION.md`, since an LLM generating a call to a `comptime`-generic function
    cannot determine its requirements without reading (and correctly interpreting)
    the function body, rather than a declared signature; (3) the reflection-collapse
    half of `comptime` (`@typeInfo`, `@field`, `@hasDecl` as ordinary compile-time
    function calls rather than a separate reflection API) is a genuinely
    orthogonal-and-simpler answer to the same problem `REFLECTION_ALTERNATIVES.md`
    already frames — "should Orthon have any runtime reflection, or should all
    metaprogramming needs be served by compile-time mechanisms" — and is worth
    weighing against that document's `reified`-generics-in-inline-functions special
    case, since a single `comptime` phase applicable to any function eliminates the
    "only inside inline functions" restriction; (4) the macro-collapse half of
    `comptime` (ordinary `if`/`for`/`while` at compile time, no macro language, no
    AST manipulation) is a different, arguably simpler answer than
    `HOMOICONICITY.md`'s two modelled options (full Lisp-style code-as-data macros,
    or Orthon's Schema-Provider-for-LLM-queries approach) — `comptime` needs neither
    code-as-data nor a schema query API for code generation, because it just runs
    the same language twice (once at compile time, once at runtime), so this file's
    Open Questions should be cross-checked against `HOMOICONICITY.md`'s Decision
    History once that document reaches Concept Design Review; (5) if Orthon's
    Concept Design Review ultimately re-affirms the existing four-mechanism approach
    (declared trait bounds for generics, a scoped reflection subset, traits for
    behaviour, `@derive` for common derivations), this file's role is to ensure that
    decision is made *with* the unified alternative on the table and explicitly
    rejected for stated reasons (most likely: discoverability and IDE/LLM tooling
    support for "what does this generic function require," which declared contracts
    provide and duck-typed `comptime` does not) — not simply never considered.

    **`## Open Questions`** — Include at least: (1) should Orthon's generics use
    declared trait bounds (as `GENERICS.md`'s Default Strategy currently specifies)
    or duck-typed `comptime`-style constraint checking at instantiation time, or a
    hybrid where trait bounds are optional annotations that narrow — but do not
    replace — instantiation-time duck-typed checking?; (2) if a unified compile-time
    execution mechanism is adopted, does it strictly subsume
    `REFLECTION_ALTERNATIVES.md`'s `reified`-generics-in-inline-functions proposal,
    or does that proposal solve a narrower, still-useful problem (reflection inside
    non-generic inline functions) that a general `comptime` parameter does not
    address?; (3) how would LLM-facing tooling (the Schema Provider,
    `LLM_NATIVE_TOOLCHAIN.md`) present a `comptime`-generic function's actual
    requirements to an LLM if those requirements are implicit in the function body
    rather than declared in the signature — would this require the compiler to
    synthesize and expose an inferred constraint set, effectively re-deriving a
    declared-trait-like contract for tooling purposes even if the source syntax
    doesn't require one?; (4) does allowing `type` as an ordinary first-class
    parameter value (rather than a special generic-parameter syntax) interact safely
    with Orthon's Data Model and Data Modifier abstractions in `DATA_MODEL.md`, or
    does it require treating `type` as a distinct kind of value with its own rules;
    (5) should compile-time-only code (functions or branches that only ever execute
    with `comptime` arguments) be visibly marked in Orthon's syntax, the way Zig
    requires the `comptime` keyword at each call site, to preserve the Explicit
    Semantics pillar's requirement that a reader can tell from local syntax whether
    code runs at compile time or runtime?

    **`## Decision History`** — Write exactly: "Initial research — no decisions
    recorded yet."

    Then a horizontal rule (`---`) followed by `### Cross-References` with plain
    bullet markdown links (matching `DYNAMIC_TYPING.md`'s bullet-link style, not a
    checkbox list) to: `GENERICS.md`, `STRUCTURAL_TYPING.md`, `METAOBJECTS.md`,
    `REFLECTION_ALTERNATIVES.md`, and `HOMOICONICITY.md`.

    Do not modify `what/GLOSSARY.md` or any other existing file.
  </action>
  <verify>
    <automated>cd /Users/mniedre/git/orthon-lang/docs && bash -c 'set -e
F="how/concepts/research/COMPILE_TIME_EXECUTION.md"
test -f "$F"
grep -q "DRAFT" "$F"
grep -q "Last updated:\*\* 2026-07-22" "$F"
ISSUE=$(grep -n "^## Issue (Why)" "$F" | head -1 | cut -d: -f1)
EX=$(grep -n "^## Examples" "$F" | head -1 | cut -d: -f1)
IMPL=$(grep -n "^## Implications for Orthon" "$F" | head -1 | cut -d: -f1)
OPEN=$(grep -n "^## Open Questions" "$F" | head -1 | cut -d: -f1)
HIST=$(grep -n "^## Decision History" "$F" | head -1 | cut -d: -f1)
XREF=$(grep -n "^### Cross-References" "$F" | head -1 | cut -d: -f1)
test -n "$ISSUE" && test -n "$EX" && test -n "$IMPL" && test -n "$OPEN" && test -n "$HIST" && test -n "$XREF"
test "$ISSUE" -lt "$EX"
test "$EX" -lt "$IMPL"
test "$IMPL" -lt "$OPEN"
test "$OPEN" -lt "$HIST"
test "$HIST" -lt "$XREF"
grep -q "GENERICS.md" "$F"
grep -q "STRUCTURAL_TYPING.md" "$F"
grep -q "METAOBJECTS.md" "$F"
grep -q "REFLECTION_ALTERNATIVES.md" "$F"
grep -q "HOMOICONICITY.md" "$F"
grep -qi "comptime" "$F"
grep -qi "duck" "$F"
! git diff --name-only -- what/GLOSSARY.md | grep -q GLOSSARY
echo OK'</automated>
  </verify>
  <done>
    `how/concepts/research/COMPILE_TIME_EXECUTION.md` exists with the DRAFT banner
    dated 2026-07-22, sections in order Issue (Why), Examples, Implications for
    Orthon, Open Questions, Decision History, Cross-References. The Issue (Why)
    section names and links `GENERICS.md`, `STRUCTURAL_TYPING.md`, `METAOBJECTS.md`,
    and `REFLECTION_ALTERNATIVES.md`, stating the scope difference (four separate
    mechanisms vs. one unified `comptime` mechanism). The Examples section contains
    the five-language comparison table and Zig/Rust code samples. Implications for
    Orthon links `HOMOICONICITY.md` and discusses the discoverability trade-off
    against the LLM Generability pillar. No existing file, including
    `what/GLOSSARY.md`, is modified.
  </done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

None. This plan creates one new static Markdown research document in a
documentation-only repository — no code, no user input, no external service, no
execution surface. The Zig/Rust/Java/Kotlin/C++ snippets shown in the Examples
section are illustrative prose content, not executed by any tool in this repository.

## STRIDE Threat Register

| Threat ID | Category | Component | Severity | Disposition | Mitigation Plan |
|-----------|----------|-----------|----------|-------------|-----------------|
| T-quick260722rge-01 | Tampering | how/concepts/research/COMPILE_TIME_EXECUTION.md | low | accept | Plain-text Markdown creation reviewed via automated grep/heading-order verification in the task's verify step; the file is committed to git for auditability. |
| T-quick260722rge-02 | Repudiation | Cross-references to GENERICS.md, STRUCTURAL_TYPING.md, METAOBJECTS.md, REFLECTION_ALTERNATIVES.md, HOMOICONICITY.md | low | accept | All cross-referenced files already exist in the repository (confirmed during planning); the task's automated verify greps for the relevant filenames' presence, preventing dangling or invented references. |
| T-quick260722rge-03 | Tampering | what/GLOSSARY.md and other existing files (unintended modification) | low | accept | The task is scoped to `<files>` containing only the new file; the verify step confirms `what/GLOSSARY.md` shows no diff, matching the precedent set by the prior Java-to-Ruby comparison plan (OPEN_CLASSES.md, SINGLETON_CLASS.md), which did not touch the glossary for DRAFT research files. |

</threat_model>

<verification>
1. `how/concepts/research/COMPILE_TIME_EXECUTION.md` exists with the DRAFT banner
   (dated 2026-07-22) and all six sections in order: Issue (Why), Examples,
   Implications for Orthon, Open Questions, Decision History, Cross-References.
2. The Issue (Why) section explicitly disambiguates the concept from `GENERICS.md`,
   `STRUCTURAL_TYPING.md`, `METAOBJECTS.md`, and `REFLECTION_ALTERNATIVES.md`,
   stating that each of those documents solves one slice of the problem with a
   separate mechanism, while this concept is the single unified alternative.
3. Implications for Orthon cross-references `HOMOICONICITY.md` and grounds its
   reasoning in `why/VISION.md`'s LLM Generability pillar and `why/MANIFESTO.md`'s
   Minimal Core / One-concept-one-syntax principles.
4. The concept is not a duplicate of any existing research file's scope (verified
   during planning via `grep -ril "comptime"` across `how/concepts/research/` —
   zero hits prior to this plan — plus a full read of `GENERICS.md`,
   `STRUCTURAL_TYPING.md`, `METAOBJECTS.md`, `REFLECTION_ALTERNATIVES.md`, and
   `HOMOICONICITY.md` confirming each proposes a distinct, non-unified mechanism).
5. `what/GLOSSARY.md` and all other existing files show no diff (`git status` /
   `git diff --name-only` limited to the one new file).
6. All 35 rows of the user's Java-to-Zig comparison table were cross-checked against
   the existing 100+ file research corpus during planning; every row besides the
   comptime-unification rows (3, 10, 11, 33) was confirmed already covered by an
   existing file under a different name (see the `<objective>` row-by-row mapping).
</verification>

<success_criteria>
- `how/concepts/research/COMPILE_TIME_EXECUTION.md` exists, is a DRAFT research
  document following the `DYNAMIC_TYPING.md`/`EXTENSION_FUNCTIONS.md` short-form
  structure, and is written entirely in English.
- The file clearly disambiguates its scope from the four existing files covering
  adjacent, narrower mechanisms (`GENERICS.md`, `STRUCTURAL_TYPING.md`,
  `METAOBJECTS.md`, `REFLECTION_ALTERNATIVES.md`) rather than duplicating any of
  them.
- The file's Implications for Orthon section grounds its reasoning in Orthon's
  stated pillars (`why/VISION.md`, `why/MANIFESTO.md`) and in genuinely related
  existing research (`HOMOICONICITY.md`), not generic restatement.
- No other file in the repository — including `what/GLOSSARY.md` — is modified by
  this plan.
- No duplicate file is created for any of the other 34 rows in the comparison
  table, all of which are already covered by existing research under different
  names.
</success_criteria>

<output>
Create `.planning/quick/260722-rge-add-concepts-from-java-to-zig-comparison/260722-rge-SUMMARY.md` when done
</output>
