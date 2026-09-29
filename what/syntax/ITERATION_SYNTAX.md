# Iteration Syntax

> **Accepted — EDR-053 (Iteration Loop); amended by EDR-094 (for-expression
> yields Optional<T>, iteration ownership).** Explicit syntax record HOMING the
> already-accepted iteration syntax in `what/syntax/`, following the
> [RANGE_SYNTAX.md](RANGE_SYNTAX.md) precedent — a record giving accepted syntax
> a home, not a new syntax-pipeline decision. The canonical semantic
> specification lives in [what/concepts/ITERATION_LOOP.md](../concepts/ITERATION_LOOP.md);
> this file records the surface form.

## Canonical forms

```orthon
# Basic iteration
for item in items:
    process(item)

# Index-based via range (inclusive-inclusive 1..N)
for i in 1..len(array):
    process(array[i])

# Destructuring in loop variable
for (index, value) in items.enumerate():
    print("Index {index}: {value}")

# while — condition-based looping
while queue.not_empty():
    process(queue.dequeue())

while i < limit:
    process(i)
    i = i + 1

# loop — infinite loop with optional break-value
let result = loop:
    let item = queue.receive()
    if is_valid(item) then break item

loop:                       # bare form, no break-value
    process.next_event()

# break — exit the loop; continue — skip to next iteration
for item in items:
    if is_target(item):
        handle(item)
        break

for item in items:
    if skip(item):
        continue
    process(item)

# for as an EXPRESSION — Optional search (value position)
let found = for row in table:
    let parsed = parse(row)
    if parsed.is_valid() and parsed.key == target:
        break parsed          # -> Some(parsed)
# run off the end -> None ; found : Optional<Row>

# Iteration ownership — markers ride on the SOURCE
for x in coll                # shared read borrow; element type &T
for x in &mut coll           # write-borrow; element type &mut T; requires mut coll
for x in $coll               # move / consume; element type T; rebind after
```

## Rules

1. **One iteration construct.** `for ... in` is the only sequence-consuming loop.
   Index iteration uses range syntax (`for i in 1..len(seq)`), not a separate
   index loop (EDR-053).
2. **`while` is the condition construct; `loop` is the infinite loop.** `while`
   repeats until a condition is false; `loop` runs indefinitely with an optional
   `break value` for expression-oriented use (EDR-053).
3. **No C-style three-clause loop.** There is no `for (;;)` form — for-each plus
   range iteration cover its uses (EDR-053).
4. **No significant whitespace.** Block structure is explicit; indentation is
   cosmetic (Syntax Principle 3).
5. **`for` is an expression in value position.** It yields `Optional<T>`:
   `break v` / `return v` → `Some(v)`; running off the end → `None`; a bare
   `break` → `None` early. In statement position `for` yields no value. Scope is
   Optional search only — a simple predicate stays `coll.find(...)`, and
   accumulation stays `fold` / `reduce` (EDR-094).
6. **`for`-`else` is not part of the language.** It is rejected by EDR-094; the
   pattern is covered by the Optional `for`-expression plus `match`.
7. **Iteration is borrow-by-default.** The ownership markers ride on the SOURCE
   and are inherited from Reference / OWNERSHIP / MUTABILITY — this record does
   not define them (see EDR-094 boundary flags B1–B4). Structural mutation
   (`coll.push` / `coll.remove`) is not a `for`-mode; it happens outside the loop
   and requires `mut coll`.
8. **`$` and `&mut` spellings are provisional.** The `$` glyph (move) and the
   `&mut` spelling (write-borrow) are provisional pending Ownership / MUTABILITY
   (EDR-094 B2/B3) — the semantic model is fixed, the concrete spelling inherited.

## Cross-References

- [EDR-053](../../how/decision_records/architecture/EDR-053-iteration-loop.md) — deciding record.
- [EDR-094](../../how/decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md) — amendment (`for`-expression → `Optional<T>`, iteration ownership).
- [what/concepts/ITERATION_LOOP.md](../concepts/ITERATION_LOOP.md) — full semantic specification.
- [what/concepts/ITERATOR_PROTOCOL.md](../concepts/ITERATOR_PROTOCOL.md) — `for` desugaring.
- [RANGE_SYNTAX.md](RANGE_SYNTAX.md) — index iteration via ranges.
- [what/SYNTAX.md](../SYNTAX.md) — hub.
- [README.md](README.md) — index.
