# Unpacking — Destructuring Assignment with Pattern Syntax

> **✅ ACCEPTED — [EDR-096](../../how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md).**
> EDR-096 supersedes the syntax decisions of
> [EDR-055](../../how/decision_records/architecture/EDR-055-unpacking.md).
>
> **Status:** Accepted 2026-07-27 under EDR-055; syntax redesigned 2026-10-17 under EDR-096.
>
> **See also:** [`PATTERN_MATCHING.md`](PATTERN_MATCHING.md),
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md),
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Identity,
> [`GLOSSARY.md`](../GLOSSARY.md) § Unpacking (Destructuring), Label, Rest Pattern, Tuple,
> [`NAMED_AND_OPTIONAL_PARAMETERS.md`](NAMED_AND_OPTIONAL_PARAMETERS.md)

---

## Issue (Why)

How do you concisely extract values from compound data structures without verbose indexing?

Explicit indexing (`list[1]`, `tuple[2]`) is:
- **Verbose** — repeated index access for multi-element extraction.
- **Brittle** — changing structure order breaks every access site silently.
- **Non-obvious** — `result[1]` reveals nothing about semantics; `ok, err = result` does.

The core problem: **binding names to structural positions** should be syntactic, not procedural.

Orthon's PRIMITIVE_BLOCKS (EDR-016) establishes `pack`/`unpack` as the fundamental value composition/decomposition primitive. Destructuring assignment is the syntactic expression of `unpack` — decomposing values into named bindings.

## Principles

1. **Symmetry with `pack`/`unpack`** — Construction and destruction use the same shape. `(x, y)` builds a tuple; `x, y = ...` takes one apart. The prefix star follows the same mirror rule in both roles (D-03): `first, *tail = nums` collects, `nums2 = (first, *tail)` spreads. This follows Representation Symmetry (DESIGN_PRINCIPLES.md).

2. **Pattern-based binding** — Destructuring follows the same pattern syntax as pattern matching (EDR-025).

3. **Consistency with PATTERN_MATCHING** — The same patterns (tuple, constructor, rest, ignore, nested) work in both `match` and destructuring contexts; the only difference is refutability (principle 6).

4. **Exhaustive by default** — Unused elements must be explicitly ignored: `_` for one position, `*_` for the remainder (D-01, D-02).

5. **One symbol, one meaning** — Syntax Principle 2 (`what/SYNTAX.md`). `..` stays Range, `?` stays null safety, `:` stays type annotation; no unpacking form reuses them (D-01, D-02, D-06).

6. **Irrefutable outside `match`** — A declaration, a `for` header and a parameter accept only patterns that cannot fail; refutable patterns are legal only in `match` (D-13).

7. **Explicit labels only** — A name attached to a tuple element or argument is always written `value as label`; nothing is inferred from bare identifiers (D-06).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Binding Policy | Declaration by assignment (EDR-074); whole-pattern `var` makes all names mutable |
| Ignore Policy | Governs syntax for unused elements (`_`, `*_`) |
| Function Argument Policy | Controls destructuring in function parameters and the call-site `value as name` spelling |
| Ownership Policy | Move versus borrow for destructured elements — undecided, tracked as open item DEF-b |
| Label Policy | Explicit labels only; position is the implicit 1-based label (EDR-082) |

## Model (What)

Destructuring assignment lets the programmer decompose tuples, finite collections and nominal types into individual bindings using pattern syntax. Patterns are written as a comma list without parentheses in declarations, `for` and `case`; parentheses are needed only for nested patterns and for the empty tuple `()` (D-05). Expression-side tuple literals keep their parentheses.

### Declaration Without a Keyword

Destructuring declares variables by assignment (EDR-074), with no keyword. `var` in front of the pattern makes ALL names in the pattern mutable (D-10).

```orthon
point = (10, 20)
x, y = point                   # declares x = 10, y = 20 (immutable)
var a, b = point               # declares a = 10, b = 20 (mutable)
ok, err = result               # tuple result split into two names
```

The complement rule follows EDR-074. A pattern whose names are all new declares them. A pattern whose names are all already declared reassigns them, and each of them must be mutable. A pattern that mixes new and already-declared names is a compile error.

```orthon
var p, q = (1, 2)
p, q = (q, p)                  # all names exist: reassignment (swap)
p, r = (5, 6)                  # compile error: p exists, r is new
```

### Ignore Pattern `_`

```orthon
x, _, z = triple               # middle value ignored
```

### Rest Pattern `*x`

The prefix star collects the remaining elements into `x`; `*_` discards them. A pattern contains exactly one star, at any position, and only in positional patterns (D-02, D-07).

```orthon
nums = (1, 2, 3, 4, 5)
first, *rest = nums            # first = 1, rest = (2, 3, 4, 5)
first, *_ = nums               # remainder discarded
*init, last = nums             # init = (1, 2, 3, 4), last = 5
first, *middle, last = nums    # middle = (2, 3, 4)
*a, *b = nums                  # compile error: more than one star
```

### Spreading in Sending Positions

The same star form spreads in sending positions (call arguments and literals), the mirror image of collecting (D-03). Spreading applies to tuples and finite collections (List, Array). A lazy iterator or generator is never spread silently; it is collected explicitly first (D-04).

```orthon
first, *tail = nums            # receiving: collects
nums2 = (first, *tail)         # sending: spreads
with_zero = (0, *items)        # items: List or Array
from_gen = (0, *gen.to_list()) # explicit collection of a lazy source
```

The FORM of the star in both roles is decided. Whether variadic functions, spreading in calls and a labeled remainder are part of v0.1 is open (DEF-a), as is spreading a Set (DEF-d).

### Labels

A label is written `value as label`. The position of an element is its implicit 1-based label (EDR-082). No label is ever inferred from a bare identifier (D-06).

```orthon
t = (1, 2, 3 as x)                          # tuple literal with a labeled element
connect("example.com", true as use_ssl)     # call argument with a label
n as name, a as age = person                # pattern: bind by label
```

The rules:
- A label overrides position when binding call arguments.
- Duplicate labels are errors; so is a double alias such as `3 as x as y`.
- Labels do not participate in types or equality.
- No implicit labels from bare identifiers: `sub(b, a)` must never silently match parameters named `a` and `b`, and `x, y = (10, 20)` must stay a purely positional binding.
- The direction of `as` (the thing on the left, the name on the right) matches its only other use, generic bounds `<Iterator as T>` (EDR-086).
- The colon stays reserved for type annotations, so a declaration is `name: Type` while a call site is `value as name`.

An anonymous record is simply a tuple whose elements all carry labels.

### Nominal Types: Constructor Patterns

A nominal type is destructured only with its constructor pattern, which mirrors construction (D-09). Brace record patterns do not exist.

```orthon
type Person(name: String, age: Int)

p = Person("Alice" as name, 30 as age)     # construction
Person(n as name, a as age) = p            # constructor pattern: n = "Alice", a = 30
```

### The 1-Tuple and the Empty Tuple

The 1-tuple exists and has exactly one spelling, the constructor `tuple(a)`. The form `(a,)` is forbidden, and `(a)` is always plain grouping. `tuple()` equals `()`, and `(a, b)` is sugar for `tuple(a, b)` (D-08).

```orthon
one = tuple(7)                 # the 1-tuple
group = (7)                    # not a tuple: grouping, just 7
nothing = ()                   # the empty tuple, same as tuple()

tuple(v) = one                 # pattern form: exactly one element
() = nothing                   # the empty-tuple pattern
```

The rest of a 2-tuple is a 1-tuple; forbidding it would make the type of a rest pattern depend on arity and break generic and recursive code.

### Nested Patterns

```orthon
(a, b), c = nested             # parentheses mark the nested pattern
Person(n as name, a as age), score = entry
```

### Patterns in `for`

```orthon
for k, *others in rows:
    print("{k}: {others}")

for index, value in items.enumerate():
    print("Index {index}: {value}")

for var k, v in rows:          # all names in the pattern are mutable
    k = k + 1

for Person(n as name, a as age) in people:
    print("{n} is {a} years old")
```

### Patterns in `case`

```orthon
match xs:
    case () => 0
    case tuple(one) => one
    case first, *rest => first + sum(rest)

match xs:
    case var first, *rest => consume(first, rest)    # var: all names mutable
```

### Patterns in Parameters

A default belongs to the whole parameter, never to a position inside a pattern (D-12; EDR-065). In a parameter list the comma already separates parameters, so a tuple pattern keeps its parentheses.

```orthon
fun distance(Point(x, y) = Point(0, 0)) -> Float
    return sqrt(x * x + y * y)

fun first_of((x, _): Pair<Int, Int>) -> Int
    return x

fun handle_users(Person(n as name, a as age))
    print("{n} is {a} years old")
```

### Refutable Patterns Are Errors Outside `match`

Outside `match`, only irrefutable patterns are allowed: declarations, `for` and parameters (D-13). Literal patterns and variant patterns (`Some`, `Ok`, user variants) are legal only in `match`. A tuple pattern applied to an `Option<T>` value is a type error; there is no implicit Option lifting.

```orthon
Some(v) = maybe_value          # error: refutable (variant) pattern in a declaration
(0, y) = point                 # error: refutable (literal) pattern in a declaration
x, y = maybe_point             # type error: maybe_point is Option<Point>
```

There are three legal routes for an optional value:

```orthon
match maybe_point:                         # 1. match
    case Some(pt) => use(pt)
    case None => fallback()

pt = maybe_point ?? Point(0, 0)            # 2. null coalescing, then destructure
x, y = pt

if maybe_point != None:                    # 3. flow-sensitive narrowing (EDR-028)
    x, y = maybe_point
```

### Mutability Is Per Pattern, Not Per Element

Only whole-pattern `var` exists (D-11). To make one name mutable while keeping the others immutable, bind it separately.

```orthon
x, y = point
var z = x                      # z is a mutable copy; x stays immutable
```

### No Defaults Inside Patterns

A pattern never carries a default value (D-12). For a statically known tuple or nominal type the field always exists; absence of a value is `Option`, handled with `??`. A form such as a default written inside a tuple pattern would also be visually ambiguous with the literal pattern `(0, y)`.

### Desugaring

All destructuring forms desugar to `pack`/`unpack` primitives and element extraction, with no new runtime semantics:

```orthon
# Tuple desugaring:
x, y = point
# → tmp = point; x, y = unpack(tmp)

# Rest desugaring: the remainder is the pack of the elements that the other
# positions did not take
first, *rest = nums
# → tmp = nums; first = tmp[1]; rest = pack(all elements of tmp after the first)

# Constructor-pattern desugaring: extraction by label
Person(n as name, a as age) = p
# → n = p.name; a = p.age
```

Rest-based extraction takes the elements by position around the single star; label-based extraction takes each element by its label. Both are expressible with the existing primitives.

## Default Strategy

All compound data types that can be destructured support the forms above. Patterns mirror PATTERN_MATCHING. Unused bindings produce a compile-time warning unless prefixed with `_`.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Positional only | Only tuples support destructuring; nominal types use `.field` access. Simpler but asymmetric. |
| Named only | Only labeled elements and nominal types support destructuring; tuples use indexed access. |
| Explicit unpack function | User-defined `unpack` method on types (Python `__iter__`). More flexible but less predictable. |
| Rejected syntax variants | The rejected spellings for ignore, rest, spreading, labels, 1-tuples, mutability, defaults and refutable patterns are listed in the Alternatives Considered table of [EDR-096](../../how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md). |

## Open Questions

The four original open questions are resolved: mutable destructuring by D-10 and D-11, default values by D-12, composition with optional values by D-13, and ownership moved to the joint decision DEF-b. The items still open (recorded in EDR-096, not decided):

1. **DEF-a** — Is the whole star mechanism — variadic functions, spreading in calls, a kwargs-like labeled remainder — included in v0.1? Working hypothesis: tuple elements may carry labels and the remainder keeps them, so one star replaces both Python argument forms.
2. **DEF-b** — Move versus borrow on destructuring, decided jointly for all constructs. Tracked discrepancy between the consume-by-default wording in PATTERN_MATCHING and the copy-on-write value semantics ([COPY_ON_WRITE.md](COPY_ON_WRITE.md)).
3. **DEF-c** — Let-else and the irrefutable single-arm match (PATTERN_MATCHING Open Question 4).
4. **DEF-d** — Spreading a Set (no order).
5. **DEF-e** — Map and brace-literal syntax.
6. **DEF-f** — The prefix star versus numeric multiplication; no arithmetic operators are formally specified yet.
7. **Drafting question (g)** — Are redundant outer parentheses around a top-level pattern rejected or tolerated?
8. **Drafting question (h)** — What is the representation of the collected remainder when the scrutinee is a List or Array rather than a tuple?

## Decision History

- **2026-07-27** — Accepted via EDR-055. Classification: Language. Destructuring syntax matching pack/unpack symmetry (PRIMITIVE_BLOCKS). All forms desugar to `pack`/`unpack` primitives — no new runtime semantics.
- **2026-10-17** — Syntax redesigned via EDR-096 (supersedes the syntax decisions of EDR-055; classification, desugaring and the mirror-image principle remain in force). Withdrawn: the dotted rest form and its brace variant, brace record patterns, the let-prefixed declaration form, parenthesized patterns in `for`, `case` and declarations, per-element mutability markers, and call-site `name: value`. Adopted: prefix-star rest and spread, explicit `as` labels, constructor patterns for nominal types, the `tuple(a)` 1-tuple, keyword-free declaration with whole-pattern `var`, irrefutable-only patterns outside `match`. Resolved from the old Open Questions: mutability (D-10, D-11), defaults (D-12), optional values (D-13); ownership moved to DEF-b.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/PATTERN_MATCHING.md`
- [x] `what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md`
- [x] `how/DESIGN_PRINCIPLES.md`
- [x] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/SEMANTIC_MODEL.md`
