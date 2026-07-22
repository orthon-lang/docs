---
phase: quick-260722-rcc
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [how/concepts/research/UNION_INTERSECTION_TYPES.md, how/concepts/research/LITERAL_TYPES.md, how/concepts/research/TYPE_LEVEL_COMPUTATION.md]
autonomous: true
requirements: []
user_setup: []

must_haves:
  truths:
    - "how/concepts/research/UNION_INTERSECTION_TYPES.md exists as a DRAFT research file matching the DYNAMIC_TYPING.md/OPEN_CLASSES.md short-form template (Issue (Why), Examples, Implications for Orthon, Open Questions, Decision History, Cross-References), dated 2026-07-22"
    - "how/concepts/research/LITERAL_TYPES.md exists with the same short-form template, dated 2026-07-22"
    - "how/concepts/research/TYPE_LEVEL_COMPUTATION.md exists with the same short-form template, dated 2026-07-22, and bundles Conditional Types, Mapped Types, Template Literal Types, keyof, typeof, Indexed Access Types, infer, and Utility Types into ONE file rather than seven"
    - "UNION_INTERSECTION_TYPES.md explicitly disambiguates structural set-theoretic union/intersection combinators over arbitrary types from ALGEBRAIC_DATA_TYPES.md's tagged sum types requiring named variant declarations"
    - "LITERAL_TYPES.md explicitly disambiguates 'a specific value inhabits its own type' from ENUM_ALTERNATIVES.md's named-constant/sum-type/enum tradeoff discussion, and cross-references UNION_INTERSECTION_TYPES.md as the composition mechanism literal types feed into"
    - "TYPE_LEVEL_COMPUTATION.md explicitly disambiguates compile-time type-derivation-as-computation from GENERICS.md's parametric polymorphism and from GRADUAL_TYPING.md's typed/untyped boundary question, and explicitly names the Turing-completeness tension against Orthon's minimal-core and LLM-generability pillars rather than glossing over it"
    - "None of the three new files modifies any existing file in the repository (additive only), and none touches what/GLOSSARY.md"
  artifacts:
    - how/concepts/research/UNION_INTERSECTION_TYPES.md
    - how/concepts/research/LITERAL_TYPES.md
    - how/concepts/research/TYPE_LEVEL_COMPUTATION.md
  key_links:
    - "UNION_INTERSECTION_TYPES.md Issue (Why) names and links ALGEBRAIC_DATA_TYPES.md, stating the scope difference (structural, untagged, arbitrary-type union/intersection vs. nominal tagged sum types with named variants)"
    - "UNION_INTERSECTION_TYPES.md cross-references DYNAMIC_COLLECTIONS.md, which already uses `List[Int | String]`-style union syntax without ever defining its semantics"
    - "LITERAL_TYPES.md Issue (Why) names and links ENUM_ALTERNATIVES.md, stating the scope difference (single-value-as-type mechanism vs. named-constant/sum-type/enum tradeoffs), and links UNION_INTERSECTION_TYPES.md"
    - "TYPE_LEVEL_COMPUTATION.md Issue (Why) names and links GENERICS.md and GRADUAL_TYPING.md, stating both scope differences, and Implications for Orthon names the Turing-completeness tension against why/VISION.md's minimal-core and LLM Generability pillars explicitly"
---

<objective>
Add three new DRAFT concept research files to `how/concepts/research/` covering the
three language concepts from the user's Java-to-TypeScript comparison table that are
genuinely NOT represented by any existing research file, confirmed via grep across the
104-file corpus during orchestration: (1) Union & Intersection Types — TypeScript's
`T | U` and `T & U` structural, untagged, set-theoretic type combinators over arbitrary
types; (2) Literal Types — TypeScript's mechanism by which a specific value (`"GET"`,
`42`, `true`) inhabits its own singleton type, composing into closed unions as a
lightweight enum alternative; (3) Type-Level Computation — the bundled cluster of
Conditional Types, Mapped Types, Template Literal Types, `keyof`, `typeof` (type
position), Indexed Access Types, `infer`, and the Utility Types built from them, framed
as one underlying idea (a pure, compile-time functional mini-language over types) rather
than seven separate features. Every other row in the user's comparison table (classes,
interfaces/structural typing, method overloading, generics, reflection, decorators,
packages, enums, instanceof/narrowing, exceptions, discriminated unions, `any`/`unknown`,
optional/rest params, nullability, collections) is already covered by an existing file
under a different name and must NOT get a duplicate file in this plan.

Purpose: This repository's research corpus was cross-checked row-by-row against the
comparison table during orchestration. Three concepts have no dedicated file and no
adequate coverage under a different name — each is mechanically distinct from its
nearest existing neighbor (Union/Intersection Types vs. `ALGEBRAIC_DATA_TYPES.md`'s
tagged sum types; Literal Types vs. `ENUM_ALTERNATIVES.md`'s named-constant/enum
discussion; Type-Level Computation vs. `GENERICS.md`'s parametric polymorphism and
`GRADUAL_TYPING.md`'s typed/untyped boundary). All three surface consequential questions
for Orthon's minimal-core, orthogonality, and LLM Generability pillars
(`why/VISION.md`, `why/MANIFESTO.md`) that deserve their own DRAFT research artifact
ahead of Concept Design Review — none should be silently folded into an existing file's
scope, and none should be invented as a duplicate of work already done. Type-Level
Computation in particular names a real tension (Turing-complete type systems vs.
minimal-core/LLM-generability goals) rather than glossing over it.

Output: `how/concepts/research/UNION_INTERSECTION_TYPES.md`,
`how/concepts/research/LITERAL_TYPES.md`, and
`how/concepts/research/TYPE_LEVEL_COMPUTATION.md`, each a DRAFT research document
following the short-form structure established by the most recent "missing language
feature" batch (`DYNAMIC_TYPING.md`, `EXTENSION_FUNCTIONS.md`, `SMART_CAST.md`,
`OPEN_CLASSES.md`, `SINGLETON_CLASS.md`): Issue (Why), Examples (comparison table +
short code samples), Implications for Orthon (numbered), Open Questions, Decision
History, Cross-References (plain bullet links).
</objective>

<execution_context>
@$HOME/.claude/gsd-core/workflows/execute-plan.md
@$HOME/.claude/gsd-core/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@how/concepts/research/DYNAMIC_TYPING.md
@how/concepts/research/OPEN_CLASSES.md
@how/concepts/research/ALGEBRAIC_DATA_TYPES.md
@how/concepts/research/STRUCTURAL_TYPING.md
@how/concepts/research/ENUM_ALTERNATIVES.md
@how/concepts/research/GENERICS.md
@how/concepts/research/GRADUAL_TYPING.md
@how/concepts/research/DYNAMIC_COLLECTIONS.md
@how/concepts/research/SMART_CAST.md
@how/concepts/research/METAOBJECTS.md
@why/VISION.md
@why/MANIFESTO.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Write UNION_INTERSECTION_TYPES.md (structural union &amp; intersection type combinators)</name>
  <files>how/concepts/research/UNION_INTERSECTION_TYPES.md</files>
  <action>
    Create the file with title `# Union &amp; Intersection Types`. Directly below the
    title, copy the DRAFT status banner block verbatim in structure and wording from
    `DYNAMIC_TYPING.md`'s header (same warning-emoji glyphs, same "exploratory research
    material for the Concept Design Review process (Milestone 2)" wording, same
    "registered only after acceptance via EDR (Architecture category)" line, same syntax
    note about abstract syntax and Phase 5), but set the "Last updated:" line to
    2026-07-22.

    Then write these sections in exactly this order, matching `DYNAMIC_TYPING.md`'s and
    `OPEN_CLASSES.md`'s heading set (`## Issue (Why)`, `## Examples`,
    `## Implications for Orthon`, `## Open Questions`, `## Decision History`, then a
    horizontal rule and `### Cross-References`) — do not use the longer `_concept.md`
    template (no Principles/Policy Footprint/Model/Default Strategy/Alternative
    Strategies sections); this concept is scoped as a short-form research note.

    **`## Issue (Why)`** — Open by describing TypeScript's `A | B` (union) and `A & B`
    (intersection) as structural, set-theoretic type combinators that operate over
    arbitrary types — including anonymous object shapes — with no tag, discriminant, or
    prior variant declaration required; any two types can be combined at the point of
    use. Then explicitly name and link `ALGEBRAIC_DATA_TYPES.md`, stating the scope
    difference in its own sentence: that document's sum types require declaring a named
    type up front with named variants, and support exhaustive `match` against an
    explicit tag baked into the type's own memory layout; this concept requires no
    named-type declaration, no tag, and applies to any two existing types, including
    types the union's author does not own. Note that intersection types (`A & B`) merge
    the structural shape of two types into a new anonymous type with all members of
    both — a type-level-only operation with no runtime object created. Then note that
    Orthon's own research corpus already assumes a union combinator exists without
    defining it: `DYNAMIC_COLLECTIONS.md` describes heterogeneous collections "via
    explicit union types (`List[Int | String]`)" and uses `Int | String` directly in a
    code sample, but never specifies whether this is a structural combinator over
    arbitrary types or shorthand for an already-declared sum type — state that this gap
    is exactly what this file exists to close. Close with the core problem statement:
    should Orthon have a structural union/intersection type combinator distinct from
    `ALGEBRAIC_DATA_TYPES.md`'s nominal tagged sum types, or should tagged sum types
    remain the sole "one of several" mechanism, with untagged structural unions rejected
    as a duplicate concept per the Manifesto's "One concept — one syntax"?

    **`## Examples`** — A comparison table with columns Language | Union mechanism |
    Intersection mechanism | Tag/discriminant required | Narrowing mechanism, covering:
    TypeScript (`A | B` structural union, `A &amp; B` structural intersection/merge; no
    tag required, though the "discriminated union" pattern optionally adds one manually;
    narrowing via `typeof`/`instanceof`/`in` checks and control-flow analysis); Scala 3
    (native `A | B` and `A &amp; B`, both structural, mirroring TypeScript directly);
    Python (`A | B` via PEP 604 / `typing.Union[A, B]`, structural union; no intersection
    type support — `Protocol` multiple inheritance is the closest workaround; narrowing
    via `isinstance` checks); Rust (no native union types; sum types (`enum`) require
    named variants instead; no intersection type — trait bounds `T: A + B` are the
    closest analogue for "must satisfy A and B," but that is a generic constraint on a
    type parameter, not a value-level type combinator; narrowing via exhaustive `match`
    on the enum's declared variants); Kotlin (no union types — sealed classes are the
    idiomatic alternative; intersection types exist only implicitly via generic bounds
    `&lt;T&gt; where T : A, T : B`); Java/C# (neither union nor intersection types exist
    as language features). Follow the table with one short TypeScript code sample
    showing a union type (`type ID = string | number;` with a function narrowing on it
    via `typeof`) and one short TypeScript code sample showing an intersection type
    (`type Named = { name: string }; type Aged = { age: number }; type Person = Named &amp;
    Aged;`), noting in prose immediately after that no runtime object or class is
    created by either declaration — both are purely compile-time type combinators.

    **`## Implications for Orthon`** — Number these implications: (1) Orthon must decide
    whether `Int | String` in `DYNAMIC_COLLECTIONS.md`'s heterogeneous-collection example
    invokes a real structural union combinator or is informal shorthand for an
    already-declared sum type — this file recommends the underlying semantics be made
    explicit during Concept Design Review before `DYNAMIC_COLLECTIONS.md`'s example is
    treated as settled syntax; (2) structural unions of arbitrary types (including
    anonymous object shapes) risk violating the Manifesto's "One concept — one syntax" —
    Orthon already has `ALGEBRAIC_DATA_TYPES.md`'s tagged sum types as the canonical "this
    OR that" mechanism, and introducing a second, structurally-typed union combinator
    alongside it creates two ways to express "one of several types," which the Language
    Design Gate's Orthogonality check should scrutinize directly; (3) if Orthon adopts
    structural unions, exhaustiveness checking — a core promise of
    `ALGEBRAIC_DATA_TYPES.md` ("the compiler enforces this") — becomes harder to
    guarantee: an untagged `Int | String` union relies on runtime narrowing
    (`SMART_CAST.md`-style `is` checks) rather than pattern-matching a declared tag, and
    the compiler cannot warn "new variant added, match not updated" the way it can for
    nominal ADTs; (4) intersection types conflict with Orthon's existing composition
    story (`COMPOSITION_OVER_INHERITANCE.md`) in kind, not degree — those describe
    runtime/instance-level composition of behaviour, while `A &amp; B` is a purely
    type-level combinator merging structural shapes with no runtime object; if Orthon's
    data model is Data First / value-semantics (`DATA_MODEL.md`), a structural
    intersection combinator may be redundant with simply declaring a new product
    type/record carrying all fields of both; (5) per the LLM Generability pillar
    (`why/VISION.md` "Small surface, consistent rules"), an open-ended structural
    union/intersection algebra over arbitrary types multiplies the number of shapes an
    LLM must reason about at a call site — every function accepting `A | B` requires the
    LLM to correctly branch/narrow both cases, and the compiler cannot exhaustiveness-
    check unless the union is closed and tagged; recommend either omitting general
    structural unions from the core, or restricting them to a narrow set of built-in
    cases (e.g. `T | None` for optionality, already covered by `NULL_SAFETY.md`'s
    `Option[T]`), leaving the general "one of several named forms" need to
    `ALGEBRAIC_DATA_TYPES.md` exclusively.

    **`## Open Questions`** — Include at least: (1) is `Int | String` in
    `DYNAMIC_COLLECTIONS.md`'s heterogeneous-collection example meant to invoke a real
    structural union type, or a stand-in for "some ADT," and should that document be
    updated once this is decided?; (2) should Orthon support structural union types at
    all, or should `ALGEBRAIC_DATA_TYPES.md`'s nominal tagged sum types remain the sole
    "sum" mechanism, with untagged unions rejected as redundant?; (3) if structural
    unions are adopted, should they be restricted to closed sets of concrete named types
    (improving exhaustiveness), or fully open to any structural shape (TypeScript's
    model)?; (4) does Orthon need intersection types at all, or does declaring a new
    named type composing existing types' fields (already supported via records/product
    types) fully cover the same need without a separate `&amp;` combinator?; (5) if
    narrowing over unions is supported, how does it interact with `SMART_CAST.md`'s
    existing narrowing rules — does an untagged union require the same
    "effectively-immutable" precondition?

    **`## Decision History`** — Write exactly: "Initial research — no decisions recorded
    yet."

    Then a horizontal rule (`---`) followed by `### Cross-References` with plain bullet
    markdown links to: `ALGEBRAIC_DATA_TYPES.md`, `STRUCTURAL_TYPING.md`,
    `SMART_CAST.md`, `DYNAMIC_COLLECTIONS.md`, `NULL_SAFETY.md`, and
    `COMPOSITION_OVER_INHERITANCE.md`.

    Do not modify `what/GLOSSARY.md` or any other existing file.
  </action>
  <verify>
    <automated>cd /Users/mniedre/git/orthon-lang/docs && bash -c 'set -e
F="how/concepts/research/UNION_INTERSECTION_TYPES.md"
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
grep -q "ALGEBRAIC_DATA_TYPES.md" "$F"
grep -q "STRUCTURAL_TYPING.md" "$F"
grep -q "DYNAMIC_COLLECTIONS.md" "$F"
grep -q "SMART_CAST.md" "$F"
grep -q "COMPOSITION_OVER_INHERITANCE.md" "$F"
grep -qi "intersection type" "$F"
grep -qi "tagged sum type" "$F"
! git diff --name-only -- what/GLOSSARY.md | grep -q GLOSSARY
echo OK'</automated>
  </verify>
  <done>
    `how/concepts/research/UNION_INTERSECTION_TYPES.md` exists with the DRAFT banner
    dated 2026-07-22, sections in order Issue (Why), Examples, Implications for Orthon,
    Open Questions, Decision History, Cross-References. The Issue (Why) section names
    and links `ALGEBRAIC_DATA_TYPES.md` and states the scope difference (structural
    untagged combinator vs. nominal tagged sum types), and names `DYNAMIC_COLLECTIONS.md`'s
    existing undefined use of union syntax. The Examples section contains the
    six-language comparison table and TypeScript union/intersection code samples.
    Implications for Orthon reasons about exhaustiveness, composition, and LLM
    Generability. No existing file, including `what/GLOSSARY.md`, is modified.
  </done>
</task>

<task type="auto">
  <name>Task 2: Write LITERAL_TYPES.md (value-as-type mechanism)</name>
  <files>how/concepts/research/LITERAL_TYPES.md</files>
  <action>
    Create the file with title `# Literal Types`. Directly below the title, copy the
    DRAFT status banner block verbatim in structure and wording from `DYNAMIC_TYPING.md`'s
    header (same glyphs, same wording, same syntax note), with the "Last updated:" line
    set to 2026-07-22.

    Then write the same six sections in the same order as Task 1's file (`## Issue
    (Why)`, `## Examples`, `## Implications for Orthon`, `## Open Questions`,
    `## Decision History`, then a horizontal rule and `### Cross-References`) — the
    short-form structure, not the full `_concept.md` template.

    **`## Issue (Why)`** — Open by describing that TypeScript lets a specific concrete
    value — a string like `"GET"`, a number like `42`, a boolean like `true` — be a type
    unto itself, inhabited only by that exact value; these literal types compose with
    union types into lightweight closed sets, e.g. `type Method = "GET" | "POST" |
    "PUT"`, as an alternative to a dedicated enum construct. Then explicitly name and
    link `ENUM_ALTERNATIVES.md`, stating the scope difference in its own sentence: that
    document compares three named strategies for modelling "a finite set of named
    values" (full enum types, named constants with auto-increment, sum types/algebraic
    data types) but never itself describes literal types as a fourth option, nor
    explains the more fundamental "value as type" mechanism a `"GET" | "POST" | "PUT"`
    union relies on — this file exists to name that mechanism directly. Then explicitly
    name and link `UNION_INTERSECTION_TYPES.md`, stating the relationship: that file's
    Issue concerns combining *existing* types (`Int | String`); this file's Issue
    concerns a single, specific *value* becoming its own type, which is then composed
    via that file's union mechanism into a closed set. Close with the core problem
    statement: should Orthon let literal values act as their own types, and if so, are
    literal types widened automatically (inferred as `String`/`Int`/`Bool`) or preserved
    verbatim depending on binding context, as in TypeScript's `let` (widens) vs.
    `const`/`as const` (preserves) distinction?

    **`## Examples`** — A comparison table with columns Language | Literal type support
    | Widening behaviour | Compose into closed union | Typical use, covering:
    TypeScript (string/number/boolean/enum-member literal types; `let` widens to the
    base type unless `as const` or an explicit annotation preserves the literal; compose
    via `|` into closed unions; typical use — HTTP methods, discriminated-union tags,
    config option strings); Python (`typing.Literal["GET", "POST"]` wraps specific
    values as a type explicitly; no automatic widening since Python's type system is
    optional/gradual; compose via `Union[Literal["GET"], Literal["POST"]]` or the
    `Literal["GET", "POST"]` shorthand; typical use — the same string-enum stand-in for
    JSON-facing APIs); Rust (no literal types as first-class citizens usable in a
    signature; the closest analogues are `const` generics — compile-time integer/bool
    values as type parameters — and literal patterns in `match`; typical use — const
    generics for array sizes, not general closed-set modelling); Kotlin/Java/C# (no
    literal types; the nearest equivalent is a real `enum`; typical use — none, always
    falls back to enum or sealed class); Swift (no general literal types; enums with raw
    values are the idiomatic mechanism instead). Follow the table with a TypeScript code
    sample showing `type Method = "GET" | "POST" | "PUT"; function request(method:
    Method) { ... }`, then a widening-contrast sample: `let x = "GET"; // type string
    (widened)` next to `const y = "GET"; // type "GET" (literal preserved)`.

    **`## Implications for Orthon`** — Number these implications: (1) ground this in
    `ENUM_ALTERNATIVES.md`'s "Option A: named constants" and its stated core problem
    ("does the language need a dedicated enum construct, or can named constants / sum
    types cover the same ground") — literal types are a fourth genuinely distinct answer
    that `ENUM_ALTERNATIVES.md`'s Model section does not enumerate; recommend
    `ENUM_ALTERNATIVES.md` be revisited during Concept Design Review to either fold
    literal-type unions in as an explicit additional option or reject them outright; (2)
    per the Manifesto's "One concept — one syntax," Orthon should be wary of shipping
    three overlapping closed-set mechanisms simultaneously (`ENUM_ALTERNATIVES.md`'s
    named constants, `ALGEBRAIC_DATA_TYPES.md`'s tagged sum types, and literal-type
    unions) — each solves "a fixed set of distinct values" with different ergonomics and
    different guarantees, and the minimal-core principle (`why/VISION.md` "Small core —
    all features decompose to a minimal primitive set") argues for picking one
    mechanism, not three; (3) if Orthon adopts literal types, the widening-vs-preserve
    question must have one explicit, always-applicable rule rather than TypeScript's
    context-dependent inference (mutable `let` widens, `const`/`as const` does not) — a
    context-dependent widening rule is exactly the "hidden conversion" and
    context-dependent-rule class of problem `why/VISION.md`'s LLM Generability section
    warns against ("no hidden conversions to forget"); (4) literal types compose with
    `UNION_INTERSECTION_TYPES.md`'s union combinator to form closed sets (`"GET" |
    "POST"`) — if Orthon rejects general structural unions (per that file's Implication
    2), literal types alone provide little value, since their primary use case is
    exactly this composition; the two decisions should be made together, not
    independently; (5) a literal-type approach to "closed set of distinct values"
    provides weaker exhaustiveness guarantees than `ALGEBRAIC_DATA_TYPES.md`'s tagged sum
    types — a string literal union has no notion of "all variants" beyond the union's
    syntactic membership, and adding a new arm silently changes the type rather than
    requiring an explicit new variant declaration; Orthon should weigh this against the
    ergonomic win (no declaration ceremony) before allowing literal-type unions into the
    core language.

    **`## Open Questions`** — Include at least: (1) should Orthon give literal values
    (string/int/bool) their own singleton type at all, or should every literal always
    widen immediately to its base type, closing off the TypeScript-style closed-string-
    union idiom entirely?; (2) if literal types are adopted, should widening be governed
    by the binding form — Orthon already distinguishes `=` mutable vs. `:=` immutable
    bindings per `GRADUAL_TYPING.md` — rather than TypeScript's opt-in `as const`?; (3)
    does adopting literal types make `ENUM_ALTERNATIVES.md`'s Option A (named constants)
    redundant, or do the two serve genuinely different needs (external string/JSON-facing
    values vs. internal enumerations)?; (4) should literal types be restricted to
    primitive scalars (string/int/bool) only, matching TypeScript, or could Orthon extend
    the concept to other literal forms?

    **`## Decision History`** — Write exactly: "Initial research — no decisions recorded
    yet."

    Then a horizontal rule (`---`) followed by `### Cross-References` with plain bullet
    markdown links to: `ENUM_ALTERNATIVES.md`, `UNION_INTERSECTION_TYPES.md`,
    `ALGEBRAIC_DATA_TYPES.md`, and `GRADUAL_TYPING.md`.

    Do not modify `what/GLOSSARY.md` or any other existing file.
  </action>
  <verify>
    <automated>cd /Users/mniedre/git/orthon-lang/docs && bash -c 'set -e
F="how/concepts/research/LITERAL_TYPES.md"
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
grep -q "ENUM_ALTERNATIVES.md" "$F"
grep -q "UNION_INTERSECTION_TYPES.md" "$F"
grep -q "ALGEBRAIC_DATA_TYPES.md" "$F"
grep -q "GRADUAL_TYPING.md" "$F"
grep -qi "widen" "$F"
grep -qi "literal type" "$F"
! git diff --name-only -- what/GLOSSARY.md | grep -q GLOSSARY
echo OK'</automated>
  </verify>
  <done>
    `how/concepts/research/LITERAL_TYPES.md` exists with the DRAFT banner dated
    2026-07-22, sections in order Issue (Why), Examples, Implications for Orthon, Open
    Questions, Decision History, Cross-References. The Issue (Why) section names and
    links `ENUM_ALTERNATIVES.md` (stating literal types are a fourth, undescribed option)
    and `UNION_INTERSECTION_TYPES.md` (stating the composition relationship). Implications
    for Orthon reasons about minimal core and LLM Generability's "no hidden conversions"
    principle. No existing file, including `what/GLOSSARY.md`, is modified.
  </done>
</task>

<task type="auto">
  <name>Task 3: Write TYPE_LEVEL_COMPUTATION.md (compile-time type-computation cluster)</name>
  <files>how/concepts/research/TYPE_LEVEL_COMPUTATION.md</files>
  <action>
    Create the file with title `# Type-Level Computation`. Directly below the title,
    copy the DRAFT status banner block verbatim in structure and wording from
    `DYNAMIC_TYPING.md`'s header (same glyphs, same wording, same syntax note), with the
    "Last updated:" line set to 2026-07-22.

    Then write the same six sections in the same order as Tasks 1 and 2 (`## Issue
    (Why)`, `## Examples`, `## Implications for Orthon`, `## Open Questions`,
    `## Decision History`, then a horizontal rule and `### Cross-References`) — the
    short-form structure, not the full `_concept.md` template. Bundle all sub-features
    into this ONE file rather than splitting into seven, mirroring how the prior quick
    task folded `method_missing` into `SINGLETON_CLASS.md` rather than splitting it out
    — list each sub-feature with its own short example inside the Examples section, but
    frame the file around the single underlying concept.

    **`## Issue (Why)`** — Open by describing TypeScript's Conditional Types (`T extends
    U ? X : Y`), Mapped Types (`{ [K in keyof T]: ... }`), Template Literal Types,
    `keyof`, `typeof` (used in type position), Indexed Access Types (`T[K]`), and
    `infer`, plus the Utility Types built entirely from them (`Partial`, `Pick`, `Omit`,
    `Record`). State that individually these look like seven unrelated features, but
    together they form one underlying idea: a small, pure, functional programming
    language that operates on types instead of values, evaluated entirely at compile
    time — with its own conditionals (`extends ? :`), its own mapping/iteration (mapped
    types), its own pattern matching and binding (`infer`), and its own string
    manipulation (template literal types). Then explicitly name and link `GENERICS.md`,
    stating the scope difference in its own sentence: that document is about parametric
    polymorphism — writing one function/type that works across multiple types via
    trait-bounded type parameters (`func lookup&lt;K: Hash, V&gt;`); this concept is about
    *computing a new type as a function of an existing type's shape* — not "does this
    type satisfy a bound" but "derive a new type from another type." Then explicitly
    name and link `GRADUAL_TYPING.md`, stating that document concerns whether/when type
    annotations are required and how typed/untyped boundaries interact, while this
    concept assumes full static typing is already in play and concerns the expressive
    power of the type language itself once inside it — a separate axis entirely. Note
    that TypeScript's type-level language is widely documented as Turing-complete —
    community type-level programs have implemented string calculators, JSON parsers, and
    even a Game of Life simulation purely inside the type checker — and state plainly
    that this is a direct, sharp tension against Orthon's stated minimal-core and
    LLM-generability goals, one that must be named explicitly here rather than glossed
    over. Close with the core problem statement: does Orthon want a Turing-complete (or
    at least highly expressive) compile-time type-computation language, a fixed closed
    set of built-in type-level utilities (`Partial`/`Pick`/`Omit`-equivalents as compiler
    intrinsics with no user-defined type-level functions), or no type-level computation
    mechanism at all beyond ordinary generics?

    **`## Examples`** — A comparison table with columns Language | Type-level
    computation support | Turing-complete? | Typical use, covering: TypeScript (full set
    — conditional types, mapped types, template literal types, keyof/typeof/indexed
    access, infer — plus Utility Types built from them; yes, Turing-complete, informally
    demonstrated by community type-level programs; typical use — deriving DTO/form types
    from a source type such as `Partial&lt;User&gt;`, string-keyed lookups, validating string
    literal shapes); Haskell (type families, both open and closed, directly parallel
    conditional/mapped types at the type level; GHC extensions make this Turing-complete;
    typical use — advanced library-internal type-level programming such as `servant` and
    `singletons`); Scala 3 (match types, e.g. `type Elem[X] = X match { case Array[t] =&gt;
    t }`, approximate conditional types; typical use — library-internal type
    transformations); Rust (const generics, associated types, and trait-based type-level
    computation such as the `typenum` crate approximate some of this, but require
    explicit trait machinery per computation with no first-class mapped/conditional type
    syntax; not Turing-complete in the same lightweight sense, though const generics plus
    traits can encode significant computation; typical use — compile-time array-size
    arithmetic); Kotlin/Java/C#/Go (no type-level computation; typical use — none, always
    falls back to runtime reflection or code-generation tooling such as annotation
    processors or source generators for the same DTO-derivation use case TypeScript
    solves at the type level). Follow the table with three short TypeScript code
    samples: a conditional type (`type IsString&lt;T&gt; = T extends string ? true :
    false;`), a mapped type (`type Partial&lt;T&gt; = { [K in keyof T]?: T[K] };`), and a
    template literal type (`` type Route = `/users/${string}`; ``). For the remaining
    sub-features (`keyof`, `typeof` in type position, Indexed Access Types, `infer`, and
    the Utility Types), give one short prose sentence each rather than a full code
    sample, consistent with bundling them into this single file.

    **`## Implications for Orthon`** — Number these implications, making sure the
    minimal-core/LLM-generability tension is named directly rather than softened: (1)
    direct tension with Minimal Core (`why/VISION.md`'s "Small core — all features
    decompose to a minimal primitive set" and the Manifesto's "Minimal core, maximum
    expressiveness") — a Turing-complete type-level language is, definitionally, an
    entire second programming language nested inside the type system, with its own
    semantics, its own recursion, and its own failure modes (type-level infinite loops
    causing compiler hangs or stack overflows are a documented TypeScript pain point) —
    this is about as far from "minimal core" as a feature can get, and must be named as
    a genuine, serious conflict rather than smoothed over; (2) direct tension with LLM
    Generability (`why/VISION.md`'s "Small surface, consistent rules" and "Explicit
    semantics reduce hallucination") — recursive conditional/mapped type definitions
    require simulating compile-time execution to know what a type resolves to; neither
    an LLM nor a human can determine a derived type's shape by local inspection alone,
    which is exactly the "hidden conversion" and "context-dependent rule" class of
    problem the LLM-readiness pillar exists to eliminate; (3) per the Manifesto's
    "Extensibility over built-in magic" and "Composition over exceptions" — if Orthon
    wants the ergonomic wins (deriving a `Partial&lt;T&gt;`/`Pick&lt;T, K&gt;`-equivalent from an
    existing record type without hand-writing a parallel type), the closed-set-of-built-
    in-utilities approach is more consistent with Orthon's philosophy than a general,
    user-extensible type-level language: ship `Partial`, `Pick`, `Omit`, `Record` (or
    equivalents) as fixed compiler intrinsics or stdlib types with fixed, documented
    semantics, rather than exposing `keyof`/mapped-type/conditional-type primitives that
    let users compose their own arbitrary type-level programs; (4) `GENERICS.md`'s
    existing trait-bounded parametric polymorphism (`func lookup&lt;K: Hash, V&gt;`) already
    covers "write code that works across multiple types" — type-level computation
    instead answers "derive a brand-new type from an existing type's shape," a
    materially different and additive need; adopting it should be evaluated as an
    explicit, separate extension to `GENERICS.md`'s model, not implied by it; (5) if
    Orthon adopts only the closed-set intrinsic approach from Implication 3, it should
    still study whether that need is better served entirely outside the type system —
    e.g. compile-time macros/derive mechanisms already anticipated by
    `ALGEBRAIC_DATA_TYPES.md`'s `@derive` and `STRUCTURAL_TYPING.md`'s `@derive(Show, Eq,
    Clone)` and `METAOBJECTS.md`'s macro/annotation research — since a derive-macro model
    produces the same DTO-shaping ergonomics without introducing a second, compile-time-
    only computational language layered on top of the value-level one.

    **`## Open Questions`** — Include at least: (1) does Orthon want any type-level
    computation beyond ordinary generics, or does `GENERICS.md`'s trait-bounded model
    plus `METAOBJECTS.md`'s `@derive` mechanism fully cover the legitimate "derive a type
    from another type" use cases TypeScript's Utility Types solve?; (2) if some
    type-level computation is wanted, should it be restricted to a small, fixed,
    non-recursive set of compiler intrinsics (no user-defined type-level functions, no
    `infer`, no recursive conditional types), eliminating the Turing-completeness and its
    associated compiler-hang failure mode?; (3) how would Orthon's compiler bound
    type-level computation to guarantee termination if any user-extensible mechanism is
    allowed — a recursion-depth limit, as some TypeScript tooling imposes informally?;
    (4) should the "derive new type from existing type's shape" need be solved entirely
    through macros/derive annotations (`METAOBJECTS.md`) instead of a parallel type-level
    expression language, keeping exactly one mechanism for "generate code/types from
    other code/types" per the Manifesto's "one concept, one syntax"?

    **`## Decision History`** — Write exactly: "Initial research — no decisions recorded
    yet."

    Then a horizontal rule (`---`) followed by `### Cross-References` with plain bullet
    markdown links to: `GENERICS.md`, `GRADUAL_TYPING.md`, `ALGEBRAIC_DATA_TYPES.md`,
    `STRUCTURAL_TYPING.md`, and `METAOBJECTS.md`.

    Do not modify `what/GLOSSARY.md` or any other existing file.
  </action>
  <verify>
    <automated>cd /Users/mniedre/git/orthon-lang/docs && bash -c 'set -e
F="how/concepts/research/TYPE_LEVEL_COMPUTATION.md"
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
grep -q "GRADUAL_TYPING.md" "$F"
grep -q "METAOBJECTS.md" "$F"
grep -q "ALGEBRAIC_DATA_TYPES.md" "$F"
grep -qi "turing-complete" "$F"
grep -qi "conditional type" "$F"
grep -qi "mapped type" "$F"
grep -qi "infer" "$F"
grep -qi "keyof" "$F"
! git diff --name-only -- what/GLOSSARY.md | grep -q GLOSSARY
echo OK'</automated>
  </verify>
  <done>
    `how/concepts/research/TYPE_LEVEL_COMPUTATION.md` exists with the DRAFT banner
    dated 2026-07-22, sections in order Issue (Why), Examples, Implications for Orthon,
    Open Questions, Decision History, Cross-References. The Issue (Why) section names
    and links `GENERICS.md` and `GRADUAL_TYPING.md`, states both scope differences, and
    explicitly names the Turing-completeness tension. The Examples section bundles
    Conditional Types, Mapped Types, Template Literal Types, keyof, typeof, Indexed
    Access Types, infer, and Utility Types into this single file with three code samples
    and five prose-only sub-feature mentions. Implications for Orthon names the
    minimal-core and LLM-generability conflict directly rather than softening it. No
    existing file, including `what/GLOSSARY.md`, is modified.
  </done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

None. This plan creates three new static Markdown research documents in a
documentation-only repository — no code, no user input, no external service, no
execution surface. The TypeScript/Rust/Python/Haskell/Scala/Kotlin/Java/C# snippets
shown in each file's Examples section are illustrative prose content, not executed by
any tool in this repository.

## STRIDE Threat Register

| Threat ID | Category | Component | Severity | Disposition | Mitigation Plan |
|-----------|----------|-----------|----------|-------------|-----------------|
| T-quick260722rcc-01 | Tampering | how/concepts/research/UNION_INTERSECTION_TYPES.md, how/concepts/research/LITERAL_TYPES.md, how/concepts/research/TYPE_LEVEL_COMPUTATION.md | low | accept | Plain-text Markdown creation reviewed via automated grep/heading-order verification in each task's verify step; all three files are committed to git for auditability. |
| T-quick260722rcc-02 | Repudiation | Cross-references to ALGEBRAIC_DATA_TYPES.md, STRUCTURAL_TYPING.md, ENUM_ALTERNATIVES.md, GENERICS.md, GRADUAL_TYPING.md, DYNAMIC_COLLECTIONS.md, SMART_CAST.md, NULL_SAFETY.md, COMPOSITION_OVER_INHERITANCE.md, METAOBJECTS.md | low | accept | All cross-referenced files already exist in the repository (confirmed during planning via `ls`/`grep`); each task's automated verify greps for the relevant filenames' presence, preventing dangling or invented references. |
| T-quick260722rcc-03 | Tampering | what/GLOSSARY.md and other existing files (unintended modification) | low | accept | Each task is scoped to a single `<files>` entry containing only the new file; each verify step confirms `what/GLOSSARY.md` shows no diff, matching the precedent set by the prior 260722-r2m commit, which did not touch the glossary for DRAFT research files. |

</threat_model>

<verification>
1. `how/concepts/research/UNION_INTERSECTION_TYPES.md` exists with the DRAFT banner
   (dated 2026-07-22) and all six sections in order: Issue (Why), Examples,
   Implications for Orthon, Open Questions, Decision History, Cross-References.
2. `how/concepts/research/LITERAL_TYPES.md` exists with the same structure and date.
3. `how/concepts/research/TYPE_LEVEL_COMPUTATION.md` exists with the same structure and
   date, bundling all eight TypeScript type-level sub-features into one file.
4. UNION_INTERSECTION_TYPES.md's Issue (Why) explicitly disambiguates itself from
   ALGEBRAIC_DATA_TYPES.md and names DYNAMIC_COLLECTIONS.md's existing undefined union
   syntax usage.
5. LITERAL_TYPES.md's Issue (Why) explicitly disambiguates itself from
   ENUM_ALTERNATIVES.md and cross-references UNION_INTERSECTION_TYPES.md as the
   composition mechanism.
6. TYPE_LEVEL_COMPUTATION.md's Issue (Why) explicitly disambiguates itself from
   GENERICS.md and GRADUAL_TYPING.md, and its Implications for Orthon names the
   Turing-completeness/minimal-core/LLM-generability tension directly, without glossing
   over it.
7. None of the three files duplicates any existing research file's scope (verified
   during planning via `grep -il` across `how/concepts/research/` for "union type",
   "intersection type", "keyof", "mapped type", "conditional type", "template literal",
   and "literal type" — no dedicated file found prior to this plan).
8. `what/GLOSSARY.md` and all other existing files show no diff (`git status` /
   `git diff --name-only` limited to the three new files).
</verification>

<success_criteria>
- `how/concepts/research/UNION_INTERSECTION_TYPES.md`,
  `how/concepts/research/LITERAL_TYPES.md`, and
  `how/concepts/research/TYPE_LEVEL_COMPUTATION.md` all exist, are DRAFT research
  documents following the DYNAMIC_TYPING.md/OPEN_CLASSES.md short-form structure, and
  are written entirely in English.
- Each file clearly disambiguates its scope from the existing file covering an adjacent
  concept (ALGEBRAIC_DATA_TYPES.md for union/intersection types; ENUM_ALTERNATIVES.md
  for literal types; GENERICS.md and GRADUAL_TYPING.md for type-level computation)
  rather than duplicating it.
- TYPE_LEVEL_COMPUTATION.md bundles all eight sub-features (Conditional Types, Mapped
  Types, Template Literal Types, keyof, typeof, Indexed Access Types, infer, Utility
  Types) into one file, framed around the single underlying "type-level computation"
  concept, and explicitly names the Turing-completeness tension against Orthon's
  minimal-core and LLM-generability goals rather than omitting or softening it.
- Each file's Implications for Orthon section grounds its reasoning in Orthon's stated
  pillars (why/VISION.md, why/MANIFESTO.md) and in genuinely related existing research,
  not generic restatement.
- No other file in the repository — including what/GLOSSARY.md — is modified by this
  plan.
</success_criteria>

<output>
Create `.planning/quick/260722-rcc-add-concepts-from-java-to-typescript-com/260722-rcc-SUMMARY.md` when done
</output>
