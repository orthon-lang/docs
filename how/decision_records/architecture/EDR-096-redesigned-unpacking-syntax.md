# EDR-096: Redesigned Unpacking Syntax — Prefix-Star Rest Patterns, Explicit `as` Labels, Irrefutable-Only Patterns Outside `match`

**Status:** Accepted

**Date:** 2026-10-17

**Category:** Architecture

**Scope:** Subsystem

**Human Sign-off:** Reviewed-by: mniedre · Date: 2026-10-17 · Verdict: LOCKED

**Supersedes:** [EDR-055](./EDR-055-unpacking.md) (syntax decisions only; classification, desugaring to pack/unpack and the mirror-image principle remain in force)

---

### Context

EDR-055 (2026-07-27) adopted destructuring assignment as a Language construct that desugars to the `pack`/`unpack` primitives. Three of its commitments are not in question and stay in force: the classification as Language (syntactic sugar, no new runtime semantics), the desugaring to `pack`/`unpack`, and the principle that construction and destruction share a mirror-image syntax (`(x, y)` builds a tuple, and the same shape on the left of `=` takes one apart). What a 2026-10 design review found is that the concrete syntax EDR-055 chose collides with other accepted constructs, so the specification as written breaks its own consistency rules.

The collisions are specific. The dotted rest form (`..rest`, `{name, ..}`) reuses `..`, which is the Range operator ([EDR-083](./EDR-083-range.md), `what/syntax/RANGE_SYNTAX.md`); the same symbol would carry two meanings, against Syntax Principle 2 in `what/SYNTAX.md` (one symbol, one meaning). The obvious alternative for an ignore pattern, `?`, is already taken by safe navigation `?.`, the null-coalescing `??`, Result propagation `?` and the reserved `?T` ([NULL_SAFETY.md](../../../what/concepts/NULL_SAFETY.md), [TYPE_LEVEL_NULL_SAFETY.md](../../../what/concepts/TYPE_LEVEL_NULL_SAFETY.md)). The colon in the rename form `{name: n}` and in call-site `name: value` is reserved for type annotations (`what/syntax/GENERICS_SYNTAX.md` rule 4, `what/syntax/RANGE_SYNTAX.md` rule 3). The `let` keyword in every EDR-055 example contradicts declaration by assignment ([EDR-074](./EDR-074-declaration-by-assignment.md)). Brace record patterns destructure by field name and so contradict nominal types, and give a second spelling to a concept that the constructor pattern already expresses (Syntax Principle 1, one concept one syntax). Finally, `how/DESIGN_PRINCIPLES.md` POLA currently says that the star only packs and never unpacks, while EDR-055 principle 1 requires construct and destruct to look the same, which is Representation Symmetry applied to the star.

The author reviewed these collisions one question at a time and accepted thirteen decisions, recorded below, together with a list of items that were explicitly left open. This EDR records both. It does not touch the classification of the construct, and it does not decide any open item: where a question belongs to a wider mechanism (variadic functions, ownership of destructured values, Set spreading, Map literals, the star versus multiplication) it is recorded under "Deferred / Open" so that it cannot be mistaken for a settled point.

---

### Decision

1. **D-01 — The ignore pattern for one position stays `_`.** It is not replaced by `?`, because `?` already means safe navigation, null coalescing, Result propagation and the reserved `?T`; one symbol must keep one meaning (Syntax Principle 2). The remainder is discarded with `*_` (D-02).

    ```orthon
    x, _, z = triple          # middle value ignored
    first, *_ = xs            # remainder discarded
    ```

2. **D-02 — The rest pattern is the prefix star `*x`.** It collects the remaining elements into `x`; `*_` discards them. The dotted rest form, the brace rest form (`{name, ..}`) and the double-underscore idea (`__`, `__rest`) are withdrawn. `..` keeps its Range meaning only ([RANGE_SYNTAX.md](../../../what/syntax/RANGE_SYNTAX.md)).

    ```orthon
    first, *rest = (1, 2, 3, 4, 5)    # first = 1, rest = (2, 3, 4, 5)
    ```

3. **D-03 — `*x` has the same form in both roles.** In receiving positions (parameters, patterns) it collects; in sending positions (call arguments, literals) it spreads. The rationale is mirror symmetry: construct and destruct look the same, exactly like `(x, y)` (EDR-055 principle 1, Representation Symmetry). The consequence is an amendment of the POLA section of `how/DESIGN_PRINCIPLES.md`, which currently says that the star always packs and never unpacks: direction may be determined by position when the construct and destruct forms are mirror images of each other, while the rule that an operator's meaning must not depend on surrounding tokens is kept for every other case. The Semantic Purity table is not changed. The tension between the prefix star and numeric multiplication is recorded as open (DEF-f), because no arithmetic operators are formally specified yet.

    ```orthon
    first, *tail = nums           # receiving: collects
    nums2 = (first, *tail)        # sending: spreads
    ```

4. **D-04 — Spreading applies to tuples and finite collections** (List, Array). Lazy iterators and generators are never spread silently: they are collected explicitly first (`*gen.to_list()`), so that an unbounded or expensive sequence cannot be consumed by accident. Spreading a Set is left open (DEF-d), because a Set has no order.

    ```orthon
    all = (0, *items)                 # items: List or Array
    all2 = (0, *gen.to_list())        # explicit collection of a lazy source
    ```

5. **D-05 — Patterns in `for`, `case` and declarations are written without parentheses**, as a comma list. Parentheses are needed only for nested patterns and for the empty tuple `()`. Expression-side tuple literals keep their parentheses.

    ```orthon
    x, y = point                       # declaration
    for k, *rest in rows:              # loop
        ...
    case first, *rest =>               # match arm
    (a, b), c = nested                 # nested pattern needs parentheses
    point = (x, y)                     # expression side keeps parentheses
    ```

6. **D-06 — Labels are explicit only, written `value as label`.** The position of an element stays its implicit label (1-based, [EDR-082](./EDR-082-1-based-indexing.md)). The rules are:
    - No implicit or soft labels from bare identifiers. Rejected: a call such as `sub(b, a)` would silently swap arguments when the identifiers happen to match parameter names (the silent argument-swap trap), and it would break positional destructuring such as `x, y = (10, 20)`.
    - Labels do not participate in types or equality.
    - A label overrides position when binding call arguments.
    - Duplicate labels and double aliases (`3 as x as y`) are errors.
    - The direction of `as` (the thing on the left, the name on the right) matches its only existing use, generic bounds `<Iterator as T>` ([EDR-086](./EDR-086-generics-syntax-revision.md)).
    - This replaces `name: value` at call sites in [EDR-065](./EDR-065-named-and-optional-parameters.md). The colon stays reserved for type annotations, so parameter declarations keep `name: Type`.
    - The colon-equals form was considered and rejected: it reads as "declare or assign a variable" in Go and in the Python walrus operator, and it reverses the data-flow direction in patterns, which breaks mirror symmetry.

    ```orthon
    connect("example.com", true as use_ssl)    # call argument
    t = (1, 2, 3 as x)                         # tuple literal
    n as name, a as age = person               # pattern, by label
    ```

7. **D-07 — Exactly one `*` per pattern**, at any position, and only in positional patterns. `first, *middle, last` is valid; `*a, *b` is an error, because the split between the two collected remainders would be undefined.

8. **D-08 — The 1-tuple exists; its only spelling is the constructor `tuple(a)`.** `(a,)` is forbidden and `(a)` is always plain grouping. `tuple()` equals `()`, and `(a, b)` is sugar for `tuple(a, b)` — the same literal-equals-constructor pattern as `[1, 2, 3]` equals `List(1, 2, 3)` ([EDR-041](./EDR-041-collection-literal-syntax.md)) and `a..b` equals `range(a, b)` (EDR-083). The pattern form is `tuple(x)`. The rationale is that the rest of a 2-tuple is a 1-tuple: forbidding it would make the type of the rest depend on arity and break generic and recursive code (Haskell had to add `Solo` to repair exactly this).

    ```orthon
    one = tuple(7)             # the 1-tuple
    match xs:
        case tuple(x) => x     # exactly one element
    ```

9. **D-09 — Brace record patterns are removed from UNPACKING.** An anonymous record is a tuple whose elements all carry labels. Nominal types (`type Person(name: String, age: Int)`) are destructured only with the constructor pattern `Person(n as name, a as age)`, which mirrors construction `Person("Alice" as name, 30 as age)` and is consistent with `User(name, role: Admin)` in `what/concepts/PATTERN_MATCHING.md`. Anonymous patterns apply only to tuples. The reasons are the same explicitness that [EDR-044](./EDR-044-structural-typing.md) applies to trait satisfaction — cited here by analogy only, not as the rule that governs record patterns: a brace pattern would leak field visibility, match structurally on field names and give one concept two spellings.

    ```orthon
    Person(n as name, a as age) = p
    ```

10. **D-10 — Destructuring declares variables without a keyword** (declaration by assignment, [EDR-074](./EDR-074-declaration-by-assignment.md)): `x, y = point`. The prefix `var` makes ALL names in the pattern mutable: `var x, y = point`. The `let` keyword in destructuring examples is withdrawn; its general fate belongs to the dormant SEED-003, which is not touched. Following EDR-074, a pattern whose names are all new declares them, a pattern whose names are all already declared reassigns them (each must be mutable), and a pattern mixing new and already-declared names is a compile error. The same rule applies in `for var k, v in rows` and `case var first, *rest =>`.

11. **D-11 — Per-element mutability is NOT supported**; only whole-pattern `var`. The workaround is a separate `var z = x`. Per-element mutability can be added later without breaking existing code.

12. **D-12 — No default values inside patterns.** A default belongs to the whole parameter per [EDR-065](./EDR-065-named-and-optional-parameters.md) (`fun distance(Point(x, y) = Point(0, 0))`); the absence of a value is `Option`, handled with `??`. For a statically known tuple or type the field always exists, and a form such as `case (x = 0, y)` is visually ambiguous with the literal pattern `(0, y)`.

13. **D-13 — Outside `match`, only irrefutable patterns are allowed** (declarations, `for`, parameters). Refutable patterns (literals, `Some`, `Ok`, variants) are errors there and legal only in `match`. Applying a tuple pattern to an `Option<T>` value is a type error; use `match`, `??` or flow-sensitive narrowing ([EDR-028](./EDR-028-type-level-null-safety.md)). There is no implicit Option lifting.

Illustrative combined example (Orthon syntax):

```orthon
point = (10, 20)
x, y = point                         # declares x, y (immutable)
var a, b = point                     # declares a, b (mutable)
first, *rest = xs                    # rest pattern
n as name, a2 as age = person        # by label (anonymous labeled tuple)
Person(n as name, a as age) = p      # nominal type: constructor pattern
for k, *others in rows: ...
match xs:
    case () => 0
    case tuple(one) => one           # exactly one element
    case first, *rest => first + sum(rest)
connect("example.com", true as use_ssl)
```

#### Deferred / Open (recorded, not decided)

The following items were explicitly left open by the author. They are recorded here so that the redesign cannot be read as having decided them.

- **DEF-a.** Whether the whole star mechanism — variadic functions `fun f(*args)`, spreading in calls, and a kwargs-like labeled remainder such as `fun wrapper(host, *opts): connect(host, *opts)` — is included in v0.1 is decided later. Working hypothesis only: tuple elements may carry labels (metadata) and the remainder keeps its labels, so that one star replaces both Python argument forms (`*args` and `**kwargs`), in line with the DRY principle in `how/DESIGN_PRINCIPLES.md`. Variadic functions are already an open question in [FUNCTIONS.md](../../concepts/research/essential/FUNCTIONS.md) and in Open Questions item 3 of [NAMED_AND_OPTIONAL_PARAMETERS.md](../../../what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md). D-03 fixes the form of the star in both roles; it does not decide that variadic functions or call-site spreading exist in v0.1.
- **DEF-b.** Move versus borrow on destructuring is to be decided jointly for all constructs, not in UNPACKING. The tracked discrepancy: EDR-055 Compliance item 7 and the ownership wording in `PATTERN_MATCHING.md` ("matching consumes the scrutinee by default; borrowing via `&`") conflict with the COPY_ON_WRITE value semantics ("assignment creates a logical copy"; [COPY_ON_WRITE.md](../../../what/concepts/COPY_ON_WRITE.md)). The PATTERN_MATCHING wording is not rewritten by this EDR; a short pointer to this item is added there.
- **DEF-c.** Let-else and the irrefutable single-arm match (Open Question 4 of `PATTERN_MATCHING.md`).
- **DEF-d.** Spreading a Set (no order).
- **DEF-e.** Map and brace-literal syntax (`COLLECTION_LITERAL_SYNTAX.md` is a candidate only).
- **DEF-f.** The prefix star versus numeric multiplication; no arithmetic operators are formally specified yet.

Two further points were identified while drafting this EDR. They are labelled as drafting questions, phrased as questions, and are not decisions:

- **(g)** Are redundant outer parentheses around a top-level pattern rejected or tolerated? All examples in the documents use the canonical unparenthesized form (D-05).
- **(h)** What is the representation of the collected remainder when the scrutinee is a List or an Array rather than a tuple?

---

### Consequences

**Positive:**
- Every symbol in the unpacking syntax keeps a single meaning: `..` is only Range, `?` is only null-safety and propagation, `:` is only a type annotation, `as` is only naming (Syntax Principle 2).
- Construct and destruct are mirror images for tuples, for the star in both roles, and for nominal constructor patterns (Representation Symmetry).
- Removes record patterns, the `let` keyword in destructuring, the dotted rest family and the colon call-site spelling: four constructs fewer for the same expressive power.
- Declaration by assignment (EDR-074) applies uniformly to destructuring; there is no separate destructuring keyword.
- Outside `match`, a refutable pattern is a compile-time error, so a failed destructuring cannot occur at run time in a declaration, a `for` header or a parameter.
- The 1-tuple makes the type of a rest pattern uniform across arities.

**Negative:**
- Call-site named arguments change from `name: value` to `value as name`, a spelling that is novel relative to common keyword-argument syntax. Follow-up (flagged, not fixed by this EDR): the old spelling still appears in other accepted documents, observed at planning time in DATACLASSES, PROPERTIES, OBJECT_INITIALIZATION, SORTING, DECLARATIVE_MULTI_KEY_SORT, DECLARATIVE_CONSTRUCTS, IMMUTABLE_DATE_TIME, ASYNC_AWAIT and EMIT_AS_INTERMEDIATE_RESULT under `what/concepts/`, in four examples in `what/GLOSSARY.md`, and in EDR-042, EDR-067 and EDR-095. A follow-up sweep is needed.
- Research drafts under `how/concepts/research/` and files under `notes/` keep the historical spellings (the dotted rest form, let-prefixed destructuring, walrus-style declarations) and are intentionally not rewritten.
- The prefix star is overloaded between collecting, spreading and (potentially) multiplication; the interplay with multiplication is open (DEF-f).
- The ownership discrepancy (DEF-b) is tracked, not resolved.
- EDR-055's body is retained unchanged for history, so a reader who opens it sees the withdrawn syntax; its Status line and a header note point here.

---

### Compliance

1. `what/concepts/UNPACKING.md` is the canonical specification of the redesigned syntax.
2. `what/concepts/PATTERN_MATCHING.md` carries the rest pattern, the `tuple(x)` pattern, the one-star rule and the irrefutability rule outside `match`, and only points at the tracked ownership discrepancy (DEF-b).
3. No accepted document uses the withdrawn spellings (the dotted rest form, brace record destructuring, the let-prefixed destructuring form, walrus-style declarations) outside historical records (the EDR-055 body, `DECISION_LOG.md` history, research drafts and notes).
4. `what/GLOSSARY.md` defines Label, Rest Pattern and Tuple and carries the updated Unpacking (Destructuring) entry.
5. EDR-055 shows `Superseded by EDR-096`; `INDEX.md` and `what/CORE_CONCEPTS.md` cite EDR-096.
6. The call-site syntax `value as name` of D-06 appears in `NAMED_AND_OPTIONAL_PARAMETERS.md`, with amendment notes in EDR-065 and EDR-053.
7. The POLA section of `how/DESIGN_PRINCIPLES.md` states the mirror-image exception of D-03 and the Semantic Purity section is unchanged.
8. Desugaring remains to `pack`/`unpack`; no new runtime semantics are introduced.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| `?` as the ignore pattern | Rejected per D-01: `?` already means `?.`, `??`, Result propagation and the reserved `?T`; a fifth meaning breaks Syntax Principle 2. |
| Keep the dotted rest form (`..rest`, `{name, ..}`) | Rejected per D-02: `..` is the Range operator (EDR-083); one symbol would carry two meanings. |
| Double-underscore rest (`__`, `__name`) | Rejected per D-01/D-02: a second ignore spelling beside `_`, and a name-shape convention instead of a symbol with a single role. |
| Postfix `x*` for spreading with prefix `*x` for collecting | Rejected per D-03: breaks mirror symmetry and reverses the role of the same position by form, a POLA-style role reversal. |
| Double parentheses `f((nums))` for spreading | Rejected per D-03/D-05: `(a)` is plain grouping (D-08), so this would overload grouping with a spread meaning. |
| `unpack(x)` as a function at call sites | Rejected per D-03/D-04: a pseudo-function that yields several values has no type as an expression; the prefix star is the mirror of the pattern. |
| `name: value` labels (`x: 3`) | Rejected per D-06: the colon is reserved for type annotations (GENERICS_SYNTAX rule 4, RANGE_SYNTAX rule 3). |
| `name := value` labels (`x:=3`) | Rejected per D-06: reads as "declare or assign a variable" in Go and the Python walrus operator, and reverses the data-flow direction in patterns, breaking mirror symmetry. |
| Implicit labels from bare identifiers (`x` equals `x as x`) | Rejected per D-06: the silent argument-swap trap `sub(b, a)` and breakage of positional destructuring `x, y = (10, 20)`. |
| `KV`/`Named<V>` carrier elements instead of tuple labels | Rejected per D-06/D-09: a wrapper type for every labeled element changes element types and equality; labels are metadata outside types. |
| `{name, age}` anonymous patterns over nominal types | Rejected per D-09: visibility leaks, structural matching on field names, and two spellings of one concept; nominal types use the constructor pattern. |
| Forbidding 1-tuples | Rejected per D-08: the rest of a 2-tuple is a 1-tuple, so the type of the rest would depend on arity and break generic and recursive code. |
| The `(a,)` spelling for the 1-tuple | Rejected per D-08: the trailing comma is a single invisible character that decides the type; `(a)` stays grouping and `tuple(a)` is explicit. |
| Per-element `var` inside a pattern | Rejected per D-11: more syntax for a rare need; whole-pattern `var` plus a separate `var z = x` suffices, and the per-element form can be added later without breaking code. |
| Default values inside patterns | Rejected per D-12: the default belongs to the whole parameter (EDR-065), absence is Option with `??`, and `case (x = 0, y)` is ambiguous with the literal pattern `(0, y)`. |
| Let-else | Not adopted per D-13; left open as DEF-c together with the irrefutable single-arm match. |
| Implicit Option lifting of tuple patterns | Rejected per D-13: it would hide a possible absence inside a declaration; use `match`, `??` or flow-sensitive narrowing. |
| Keeping `let` for destructuring | Rejected per D-10: contradicts declaration by assignment (EDR-074) and gives destructuring a keyword that plain declarations do not need. |

---

### Gate Validation

Gate selection per `DECISION_VALIDATION.md` § Gate Selection: all seven gates are applied, because the decision changes and removes language rules (it replaces syntax of an accepted construct, amends a design principle and withdraws several forms).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The programmer-facing problem is unchanged ("extract values from compound data without verbose indexing"); the redesign removes symbol collisions a programmer would otherwise meet, and keeps one mirror-image rule: what builds a value also takes it apart. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass (Flag) | Each withdrawn spelling is replaced by one that keeps single meanings (`..` Range, `?` null-safety, `:` type annotation). Flag: the prefix star versus numeric multiplication is open (DEF-f). |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | The model shrinks: record patterns, the `let` keyword in destructuring, the dotted rest family and the colon call-site spelling are removed, and nothing new is added except the explicit label `as`. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass (Flag) | All forms still desugar to `pack`/`unpack`; no new primitive and no layer violation. Flag: the ownership discrepancy with COPY_ON_WRITE is tracked, not resolved (DEF-b). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | The decision is a syntactic transformation and a set of static rules (irrefutability, one star, label uniqueness) that are independent of allocation, evaluation or execution strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "the pattern on the left is the mirror of the expression on the right". Per-element `var` can be added later without breaking code, and the 1-tuple avoids a Solo-style retrofit. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass (Flag) | Destructuring, star rest and constructor patterns are familiar from Python, Rust and Swift. Flag: `value as label` at call sites is novel versus keyword-argument spellings; mitigated by compile-time errors for duplicate labels and refutable patterns, which give a model direct self-correction feedback. |

**Gates not applied:** None.

**Detailed reasoning:** See the `DECISION_LOG.md` entry "UNPACKING Syntax Redesign (EDR-096)" for the per-gate reasoning trail.
