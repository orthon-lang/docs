# ADT and Union Types — Usage Patterns

> Exploratory note: a usage catalogue distilled from a Q&A discussion.
> Not a language feature. Cross-references the accepted concepts
> [`ALGEBRAIC_DATA_TYPES.md`](../what/concepts/ALGEBRAIC_DATA_TYPES.md)
> (EDR-039) and
> [`UNION_INTERSECTION_TYPES.md`](../what/concepts/UNION_INTERSECTION_TYPES.md)
> (EDR-045).

## Terminology anchor

The names that appear between `|` in an ADT declaration are **variants**
(GLOSSARY.md § Algebraic Data Type). Each variant has a same-named **data
constructor** that builds a value of the parent type. They are *not*
standalone types — `type Payment = Cash | CreditCard(...)` introduces the
type `Payment`, not types `Cash`/`CreditCard`. The word "representation" is
reserved in Orthon for the seven data representations (Value, Tuple,
Reference, Sequence, Set, Option, Result) and must not be used for ADT
variants.

Precise phrasing: `Cash`, `CreditCard`, `Crypto` are **variants of
`Payment`**, each with a **constructor of `Payment`**.

## ADT vs Union — when to reach for which

| Need | Mechanism |
|------|-----------|
| Closed, owned set of alternatives; compiler-enforced exhaustiveness | **ADT** (EDR-039) |
| Distinguish cases that share the same representation (a tag is needed) | **ADT** (variant tag) |
| Payload-free markers, recursion, domain modelling | **ADT** |
| Ad-hoc combination of **existing** named types at point of use | **Union** (EDR-045) |
| Combine types you do **not** own (e.g. `String \| IPAddress`) | **Union** |
| No declaration, "inline" | **Union** |

`|` means different things in each: in an ADT it separates *variant
constructors* (new names); in a union it combines *existing types*. Union is
untagged with no exhaustiveness; ADT is tagged with compiler-enforced
exhaustiveness.

## Pattern 1 — Union type (`A | B`)

Combines already-existing named or literal types; no declaration of new
names.

```orthon
type ID = String | Int
type Method = "GET" | "POST" | "PUT"    # literal-type union (EDR-043)

fun print_id(id: String | Int)
    match id:
        case s: String => print("ID: $s")
        case i: Int    => print("ID: $(i.to_string())")

let items: List[String | Int] = ["hello", 42, "world"]
```

Intersection types (`A & B`) are **not** accepted for v0.1 (EDR-045).

## Pattern 2 — Payload-free variants (enum use case)

A variant carrying no data — the fact of the case is the whole information.
ADT subsumes the dedicated enum construct (EDR-039).

```orthon
type Color = Red | Green | Blue
type Direction = North | South | East | West
```

Uses: enumerations, named states, sentinels. Canonical sentinel is `None` in
`type Option<T> = Some(value: T) | None`.

## Pattern 3 — Mixed ADT (some variants empty, some with fields)

The common real-world shape: some cases need a payload, others do not.

```orthon
type Payment = Cash | CreditCard(card_number: String) | Crypto(txid: String)

type Connection = Disconnected
                | Connecting
                | Connected(socket: Socket, since: Time)
                | Failed(error: String)

type Event = Play | Pause | Stop | Seek(position: Duration)
```

`match` must still cover the empty variants — the compiler enforces it.
Default layout is a tagged union; niche optimisation can store an empty
variant without extra space (e.g. `None` as a null pointer in
`Option<&T>`).

## Pattern 4 — Behaviour over a closed ADT (Path A: variants + match)

Behaviour is always **external** to the `type` (EDR-039 §4) — Orthon rejects
the "methods inside the class" model. Path A: one closed ADT, one `impl`,
dispatch via `match`.

```orthon
type Shape = Circle(radius: Float)
           | Rectangle(w: Float, h: Float)

trait Area
    fun area(self) -> Float

impl Area for Shape
    fun area(self) -> Float
        match self:
            Circle(r)        -> pi * r * r
            Rectangle(w, h)  -> w * h
```

Adding a variant (`Triangle`) makes every consuming `match` non-exhaustive —
a compile-time error. Path A **optimises adding operations** (a new function
is a new `match`, no existing code changes) at the cost of adding types.

## Pattern 5 — Independent types with external behaviour (Path B: types + traits)

When the alternatives should be **real types** with their own fields and
behaviour, declare each as a standalone product type and attach behaviour via
separate `impl` blocks sharing a trait.

```orthon
type Cash(amount: Money)
type CreditCard(card_number: String, holder: String, expiry: Date, cvv: String)
type Crypto(txid: String, wallet: String, chain: Chain)

trait Charge
    fun charge(self) -> Result

impl Charge for Cash
    fun charge(self) -> Result
        return ok()

impl Charge for CreditCard
    fun charge(self) -> Result
        return process_card(self.card_number, self.cvv)

impl Charge for Crypto
    fun charge(self) -> Result
        return broadcast(self.chain, self.txid)
```

Unify the family through the trait, not through a sum type:

```orthon
# static — monomorphised, default, zero overhead
fun pay<Charge as P>(p: P) -> Result
    return p.charge()

# dynamic — opt-in vtable
let payments: List[dyn Charge] = [Cash(...), CreditCard(...), Crypto(...)]
```

Path B **optimises adding types** (a new type is a new record + `impl`, no
existing code changes) at the cost of adding operations. This is the
data/behaviour split's answer to the Expression Problem (EDR-039 §4).

## Multiple constructors

Orthon product types have a single structural constructor (no OOP
constructor overloading). Alternative construction paths are **free
functions** ("smart constructors"), which is also where validation belongs:

```orthon
fun credit_card_from_token(token: String) -> Result<CreditCard, Error>
    ...
```

See [`parse-dont-validate-idiom.md`](parse-dont-validate-idiom.md).

## See also

- [`ALGEBRAIC_DATA_TYPES.md`](../what/concepts/ALGEBRAIC_DATA_TYPES.md) — EDR-039, ADT mechanism, Path A/B, Expression Problem
- [`UNION_INTERSECTION_TYPES.md`](../what/concepts/UNION_INTERSECTION_TYPES.md) — EDR-045, structural unions
- [`TRAITS.md`](../what/concepts/TRAITS.md) — external behaviour, static vs `dyn` dispatch
- [`PATTERN_MATCHING.md`](../what/concepts/PATTERN_MATCHING.md) — EDR-025, exhaustive matching
- [`GLOSSARY.md`](../what/GLOSSARY.md) — § Algebraic Data Type, Sum Type, Product Type, Union Type, Representation
