# Code Block as Higher-Order Function

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during exploratory work as analysis of a language
> pattern. It will be formally reviewed through the Concept Design
> Review process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Problem

How does a language allow **blocks of code to be passed as arguments** to
functions, stored in variables, or returned from other functions — using
a lightweight, syntactically distinct notation that makes higher-order
calls read like control structures?

Most languages support first-class functions (lambdas, closures), but they
typically require a distinct syntactic construct: the `fn` or `lambda`
keyword, arrow syntax (`=>`, `->`), or a special anonymous-function
literal. While this works, it introduces ceremony for the common case of
passing a single block of code to a function — especially when the block
is short and used inline.

Several languages (Ruby, Smalltalk, Groovy, Kotlin, Swift) have converged
on a **block-passing convention** where a code block can be placed after
the function call, outside the argument parentheses, making the call read
like a built-in control structure. This is both a syntactic sugar and a
semantic pattern: the block is still a closure (it can capture variables),
but its syntactic placement signals "this argument is the body of
computation."

The core question: **should Orthon support a dedicated block-passing
convention** as syntactic sugar over anonymous-function arguments, or
should it rely entirely on general-purpose lambda syntax (as described in
[`FUNCTIONS.md`](FUNCTIONS.md))?

## Examples

### Ruby — The Definitive Block Language

Ruby makes code blocks a first-class syntactic citizen. Every method can
receive an implicit block that is not part of the formal parameter list.

#### Implicit block with `yield`

```ruby
def repeat(times)
  i = 0
  while i < times
    yield(i)    # calls the implicit block with argument i
    i += 1
  end
end

# Usage — block as trailing argument
repeat(3) do |i|
  puts "Hello #{i}"
end

# Alternative brace syntax for single-line blocks
repeat(3) { |i| puts "Hello #{i}" }
```

Two syntaxes coexist:

- `do ... end` — multi-line blocks, conventional for statements
- `{ |params| ... }` — single-expression blocks, conventional for
  pipelines and short closures

Both are semantically equivalent: each is a **closure** — it captures
variables from the enclosing scope by reference.

#### Block as explicit `Proc` parameter (`&`)

When a block needs to be stored, passed to another method, or returned,
it must be captured as a `Proc` using the `&` prefix:

```ruby
def create_multiplier(factor)
  # Capture the implicit block as a Proc and return it
  ->(x) { yield(x * factor) }
end

# Alternatively, capture via & parameter
def make_counter
  count = 0
  # Return a Proc that increments and yields
  Proc.new { count += 1; yield(count) }
end
```

The `&` operator converts between blocks and `Proc` objects:

```ruby
def method_with_explicit_block(&block)
  # block is now a Proc
  block.call(42)
end

method_with_explicit_block { |x| x * 2 }
# => 84

# Passing a Proc as a block to another method
def each_item(&block)
  [1, 2, 3].each(&block)
end
```

#### Block returning a block

```ruby
def make_adder(n)
  # Returns a Proc that accepts another block
  ->(x) do
    yield(n + x) if block_given?
    n + x
  end
end

adder = make_adder(5) do |result|
  puts "Result: #{result}"
end

adder.call(3)  # prints "Result: 8", returns 8
```

#### Key observation: `yield` vs `block.call`

Ruby's `yield` is faster than calling `block.call` because the compiler
optimises the implicit-block path — it avoids allocating a `Proc` object
when the block is only invoked, never stored or passed along. This
optimisation means that the block-passing pattern is not just syntactic
sugar but also a **performance hint**: blocks that do not escape their
scope can be inlined or stack-allocated.

### Smalltalk — Blocks from the Ground Up

Smalltalk was the original inspiration for Ruby's block syntax. Every
control structure is a method that takes a block:

```smalltalk
" ifTrue: is a method on Boolean "
value > 0
    ifTrue: [ :v | Transcript show: 'Positive: '; show: v ]
    ifFalse: [ Transcript show: 'Zero or negative' ].

" whileTrue: is a method on BlockClosure "
[ queue isEmpty ] whileFalse: [ queue processNext ].
```

In Smalltalk, blocks (`[...]`) are first-class objects (`BlockClosure`)
with no special syntax — even `ifTrue:`, `whileTrue:`, and `timesRepeat:`
are library methods, not language keywords. This is the purest expression
of the "code block as argument" pattern.

### Kotlin — Trailing Lambda

Kotlin allows a lambda to be moved outside the parentheses when it is the
last argument:

```kotlin
// Normal call
repeat(3, { println(it) })

// Trailing lambda — block outside parens
repeat(3) { println(it) }
```

Kotlin does not distinguish between blocks and lambdas; the block syntax
is purely positional sugar. Unlike Ruby, there is no `yield` keyword —
the lambda is always an explicit parameter.

### Swift — Trailing Closure

Swift follows the same pattern as Kotlin:

```swift
// Without trailing closure
UIView.animate(withDuration: 0.3, animations: {
    view.alpha = 0
})

// With trailing closure
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
}
```

Swift allows **multiple trailing closures** (since 5.3) for functions that
accept several closures:

```swift
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
} completion: { finished in
    // done
}
```

### Groovy — Closure as Last Parameter

Groovy's syntax is similar to Ruby but uses method-level `Closure`
parameters:

```groovy
def repeat(times, Closure closure) {
    times.times { closure(it) }
}

repeat(3) { println "Hello $it" }
```

## Implications for Orthon

### 1. Block-passing aligns with Orthon's design philosophy

The block-passing convention supports several Orthon design principles:

- **Explicitness** — A block syntactically visible at the call site signals
  "this argument is the primary computation." Unlike Ruby's implicit yield
  (which is invisible in the parameter list), Orthon could require the
  block parameter to be declared explicitly, making the contract clear.

- **Learnability** — `each(items) { |x| ... }` reads more naturally than
  `each(items, fn (x) ...)` for the common case of passing one computation.
  The block syntax distinguishes "data arguments" from "code arguments."

- **LLM Generability** — A simple, consistent block syntax reduces the
  syntactic surface the LLM must learn. Ruby-style `do...end` pairs are
  easier for an LLM to close correctly than deeply nested parentheses.

### 2. Relationship to existing FUNCTIONS.md

`FUNCTIONS.md` already establishes:

- Functions are first-class values (Principle 1)
- Uniform call syntax (Principle 2)
- Explicit closure capture (Principle 3)
- Named before anonymous (Principle 4)

A block-passing convention would be a **syntactic sugar** over anonymous
functions — the block `do |x| x + 1 end` would desugar to
`fn (x) -> x + 1`. It would **not** add new semantics; it would add a
new syntactic form for the common case of passing a single function
argument.

### 3. Key design questions for Orthon

If Orthon adopts a block-passing convention, several decisions must be
made:

| Question | Ruby's answer | Kotlin's answer | Implication for Orthon |
|---|---|---|---|
| Is the block implicit or explicit? | Implicit (`yield`) | Explicit (lambda parameter) | Orthon prefers explicitness — explicit block parameter |
| Is `yield` a keyword? | Yes | No (use `invoke`) | Orthon could provide `yield` as sugar over calling the block param |
| Are there two syntaxes? | `do...end` + `{ }` | Single (braces only) | Orthon's minimal-core principle favours one syntax |
| Can the block be stored/returned? | Via `&` → `Proc` | It's a value already | Orthon's first-class functions make this natural |
| `break`/`next` semantics? | `break` exits method, `next` skips iteration | `return` returns from outer scope | Must define block → enclosing scope semantics carefully |

### 4. Minimal core vs. syntactic sugar

A block-passing convention sits at the boundary between "core semantics"
and "syntactic sugar" (Decision Pipeline Q5–Q7). If Orthon's first-class
functions already support anonymous closures, then blocks are sugar. If
blocks introduce new semantics (like `yield` with special control flow),
they become a core concept.

**Recommendation for Orthon:** Adopt blocks as **syntactic sugar** over
anonymous functions, with a single syntax (e.g., `do |params| ... end`)
and an explicit block parameter in the function signature. This gives the
readability benefit without fragmenting the semantic model.

### 5. Policy interaction

If blocks are syntactic sugar over functions, they inherit the same
Policy Footprint (see `FUNCTIONS.md`):

| Policy Type | Role (inherited from functions) |
|---|---|
| Evaluation Policy | When is the block evaluated? (eager by default) |
| Lifetime Policy | How long do captured variables live? |
| Allocation Policy | Is the closure stack- or heap-allocated? |
| Mutability Policy | Can the block mutate captured state? |

The block-passing convention adds one new concern:

- **Inlining Policy** — Should non-escaping blocks be inlined at the call
  site (like Ruby's `yield` optimisation) or always allocated as closures?
  Default: inline when the block does not escape its scope.

## Open Questions

1. **Single syntax or dual?** — Should Orthon have one block syntax
   (e.g., `do ... end`) or two (a multi-line form and a single-expression
   form)? Ruby's two-form approach adds flexibility but violates the
   "one concept per syntax" principle.

2. **`yield` keyword?** — Should Orthon have a `yield` keyword that calls
   the block parameter, or should the block be called as a normal function
   value (`block(arg)`)? Ruby's `yield` is optimised but invisible; an
   explicit call is more transparent.

3. **Block → closure conversion** — If blocks are syntactic sugar, how
   does the programmer capture a block as a first-class value to pass or
   return? Via a prefix operator like Ruby's `&`? Via explicit assignment
   to a function-typed variable?

4. **Control flow semantics** — What does `return` mean inside a block?
   Does it return from the enclosing function (like Ruby's `proc`) or only
   from the block (like Ruby's `lambda`)? This is the most common
   confusion point in real-world Ruby code.

5. **Multiple block parameters** — Should Orthon allow multiple block
   arguments (like Swift's multiple trailing closures)? This complicates
   the syntax but enables callbacks and configuration patterns.

6. **Interaction with Execution Program model** — If Orthon's Execution
   Program model separates semantics from execution strategy, can a
   block be serialized or passed across execution boundaries (remote
   calls, WASM boundaries)?

## Cross-References

- [`FUNCTIONS.md`](FUNCTIONS.md) — First-class functions, closures, and
  uniform call syntax (the semantic foundation that block-passing would
  build upon)
- [`SIGNIFICANT_WHITESPACE.md`](SIGNIFICANT_WHITESPACE.md) — Block
  delimiters and whitespace sensitivity (block syntax affects delimiter
  design)
- [`GENERATORS.md`](GENERATORS.md) — Generator/yield research (related
  but distinct: generators produce sequences, blocks pass computation)
- [`ITERATION_LOOP.md`](ITERATION_LOOP.md) — Loop and iteration
  constructs (blocks are a natural fit for `each`-style iteration)
- [`../../DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) — Orthon's
  design principles (explicitness, minimal core, composition over
  exceptions)
- [`../../../why/MANIFESTO.md`](../../../why/MANIFESTO.md) — Manifesto
  principles relevant to syntactic sugar decisions
