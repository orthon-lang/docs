# Orthon Glossary

> A unified reference of all project terminology.
> Each entry includes a definition, cross-references to source documents, and links to related terms.

---

## A

### Algebraic Data Type (ADT)

A type defined as a choice between named variants, where each variant
carries its own fields. ADTs combine **sum types** ("this OR that") and
**product types** ("this AND that") into a single declaration.

```orthon
type Shape = Circle(radius: Float)
           | Rectangle(width: Float, height: Float)
```

The `|` separates variants. Variant fields are named by default (positional
shorthand available for single-field variants). ADTs subsume dedicated enum
constructs — payload-free variants (`type Color = Red | Green | Blue`)
serve the enum use case with compiler-enforced exhaustiveness.

ADTs compile to sealed trait hierarchies (EDR-019) with automatic
discriminant generation. Pattern matching (EDR-025) on ADTs must cover
all variants — the compiler enforces exhaustiveness.

- **Source:** `../what/concepts/ALGEBRAIC_DATA_TYPES.md`, [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md)
- **See also:** [Sum Type](#sum-type), [Product Type](#product-type), [Tagged Union](#tagged-union), [Exhaustiveness](#exhaustiveness), [Pattern Matching](#pattern-matching)

### Async

A modifier on `proc`/`fun`/`new` indicating that the function may suspend
execution at `await` points and resume later. `async` is an **execution
modifier**, not a semantic category — it composes orthogonally with the
three declaration kinds (`async proc`, `async fun`, `async new`). Async
says nothing about parallelism or concurrent access; those are expressed
via `spawn` (parallel execution) and `exclusive` (access serialisation).

```orthon
async fun fetch(url: Url) -> String
    return await httpClient.get(url)
```

An `async` function returns `Future<T>`. Calling without `await` produces
a `Future` without suspension (colourless model).

- **Source:** `../what/concepts/ASYNC_AWAIT.md` (EDR-047)
- **See also:** [Await](#await), [Future](#future), [Spawn](#spawn), [Scope](#scope-structured-concurrency), [Exclusive Access](#exclusive-access)

### Await

The syntactic marker of a suspension point in an `async` function. `await`
suspends execution until the awaited `Future<T>` resolves to a value of
type `T`. `await` is the only yield point (cooperative scheduling).

```orthon
let result = await future   # suspend until future completes
```

`await` is required only when the current code needs the result value.
Without `await`, an async function call produces a `Future<T>` without
suspension.

- **Source:** `../what/concepts/ASYNC_AWAIT.md` (EDR-047)
- **See also:** [Async](#async), [Future](#future)

### Allocation Policy

An Implementation Policy that controls how memory is acquired and
deallocated. Part of the Strategy system ([EDR-034](../how/decision_records/architecture/EDR-034-allocation.md)).
Defines five mutually exclusive values: `Heap`, `Arena`, `Linear`,
`GC`, `Static`. The Allocation Policy is classified as Policy, not
Language — allocation is an implementation choice about HOW primitives
are materialised in memory.

```text
DEFAULT_STRATEGY: Allocation = Arena
EMBEDDED_STRATEGY: Allocation = Static
```

- **Source:** `../how/IMPLEMENTATION_POLICIES.md` § Allocation Policy, [EDR-034](../how/decision_records/architecture/EDR-034-allocation.md)
- **See also:** [Implementation Strategy](#implementation-strategy), [Policy](#policy), [Region-Based Memory](#region-based-memory)

### Algebraic Data Type (ADT)

A type defined as a choice between named variants, where each variant carries its own fields. ADTs combine **sum types** ("this OR that") and **product types** ("this AND that") into a single declaration.

```orthon
type Shape = Circle(radius: Float)
           | Rectangle(width: Float, height: Float)
```

The `|` separates variants. Variant fields are named by default (positional shorthand available for single-field variants). ADTs subsume dedicated enum constructs — payload-free variants (`type Color = Red | Green | Blue`) serve the enum use case with compiler-enforced exhaustiveness.

ADTs are declared with the `type` keyword and compile to sealed trait hierarchies (EDR-019) with automatic discriminant generation. Pattern matching (EDR-025) on ADTs must cover all variants — the compiler enforces exhaustiveness.

- **Source:** `../what/concepts/ALGEBRAIC_DATA_TYPES.md`, [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md)
- **See also:** [Sum Type](#sum-type), [Product Type](#product-type), [Tagged Union](#tagged-union), [Exhaustiveness](#exhaustiveness), [Pattern Matching](#pattern-matching)

### Architecture

The layered structure of Orthon: Core Language → Syntax → Standard Library → Implementation Strategy.

The central architectural invariant: the **Core Language** and **Standard
Library** define interfaces and contracts; the **Implementation Strategy**
fulfills them. User code depends only on the interfaces, never on the
strategy.

- **Source:** `../how/architecture/ARCHITECTURE.md`
- **See also:** [Core Language](#core-language), [Implementation Strategy](#implementation-strategy), [Standard Library](#standard-library)

---

## B

### Binding Identity

Whether two *names* currently refer to the same storage location — a
compiler/runtime aliasing concern (borrow tracking, aliasing analysis),
not a first-class equality operator exposed on ordinary values. Distinct
from Value Identity; the Semantic Model's Identity dimension exists
partly to keep the two from being conflated.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity
- **See also:** [Value Identity](#value-identity), [Semantic Dimension](#semantic-dimension), [Orthogonality](#orthogonality)

### Boolean Blindness

An anti-pattern where a function returns `Bool` as a *validation
verdict*, erasing the fact of validity as soon as the check ends. At the
call site, the program *knows* the value is valid but the type system
cannot use that knowledge — the witness of validity was discarded with
the boolean. The remedy is the **Parse, Don't Validate** idiom: parse the
value into a more precise type (`Option<T>` / `Result<T,E>` / a
constrained type) whose construction guarantees validity.

```orthon
# Boolean Blindness — witness erased
fun isValidEmail(s: String) -> Bool
    return s.contains("@") and s.contains(".")

# Parse, don't validate — witness is the type
fun parseEmail(s: String) -> Option<Email>
    ...
```

- **Source:** `../notes/parse-dont-validate-idiom.md`, [`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md)
- **See also:** [Parse, Don't Validate](#parse-dont-validate), [Option Type](#option-type), [Constrained Type](#constrained-type)

---

## C

### Canonical Form

One of the equivalent syntactic ways to express a language construct. All canonical forms of a feature must be documented together (see *Show All Canonical Forms* principle).

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Documentation Principle
- **See also:** [Operator Equivalence](#operator-equivalence)

### Channel

A typed, bounded or unbounded message-passing conduit between delegates.
`Channel<T>` wraps delegate mailboxes to provide ergonomic `send`/`receive`
operations with static type safety. Channels are StdLib types — no new
language semantics — and form the backbone of CSP-style concurrency patterns
(select, fan-out/fan-in, pipeline).

```orthon
let ch = Channel<String>(buffer: 10)
delegate producer:
    ch.send("hello")
delegate consumer:
    let msg = ch.receive()
```

- **Source:** `../what/concepts/CONCURRENCY.md` (EDR-049)
- **See also:** [Delegate](#delegate), [Sequence](#sequence)

### Collection Literal

A compact, readable syntax for constructing collections (lists, maps, sets)
without create-then-mutate boilerplate. Collection literals are syntactic
sugar over standard library constructor calls — they are classified as

### Constrained Type

A nominal type that wraps a primitive base type with a validation predicate
declared at the type level. The constraint is checked at every boundary where
a raw value enters the type (construction, assignment, parameter passing with
implicit conversion). Constraint lives **only on the type** — consuming
functions carry no duplicate `requires`. Construction uses the `Callable`
trait (consistent with uniform call syntax), not a dedicated `new`/`make`
keyword.

Example: `type Age = Int requires v >= 0 && v <= 150` creates a type `Age`
distinct from `Int`, where values are validated at every entry boundary.

- **Source:** `what/concepts/CONSTRAINED_TYPES.md`
- **Accepted:** EDR-080
- **See also:** [Contract](../how/concepts/research/important/CONTRACTS.md), [Struct](../how/concepts/research/important/STRUCT_AS_NOMINAL_PRODUCT_TYPE.md)
StdLib, not Language ([EDR-041](../how/decision_records/architecture/EDR-041-collection-literal-syntax.md)).

Collection literals are **immutable by default**. Mutable variants require
an explicit `mut` qualifier. Concrete syntax is deferred to Phase 5;
candidates include `[]` for lists, `{}` for maps, and `{}`/`[]` for sets.

- **Source:** `../what/concepts/COLLECTION_LITERAL_SYNTAX.md`, [EDR-041](../how/decision_records/architecture/EDR-041-collection-literal-syntax.md)
- **See also:** [Combinator](#combinator), [Iterator Protocol](#iterator-protocol)

### Combinator

A higher-order function on `Iterator[T]` that transforms, filters, or aggregates a sequence without materialising intermediate collections. Combinators are lazy by default — they return new `Iterator` values that apply the transformation on demand. Examples: `map`, `filter`, `fold`, `take`, `skip`, `chain`, `zip`, `enumerate`.

Combinators are classified as **StdLib** (EDR-032) — they are method implementations on the `Iterator[T]` trait, not language constructs. Loop fusion (combining multiple combinator passes) is an Implementation Strategy concern.

```orthon
let result = numbers@filter(fn(x) -> x > 0)@map(fn(x) -> x * 2)@collect()
```

- **Source:** `../what/concepts/COMPOSABLE_COLLECTION_OPS.md`
- **See also:** [Iterator Protocol](#iterator-protocol), [Generator](#generator), [Lazy Sequence](#lazy-sequence)

### Core Language

The minimal, stable set of language semantics and semantic contracts,
independent of any particular implementation. The Core defines *what
programs mean*, not *how they execute*. It specifies the interfaces
that every Implementation Strategy must fulfill.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Core Language, `../how/DESIGN_PRINCIPLES.md` § Minimal Core
- **See also:** [Architecture](#architecture), [Implementation Strategy](#implementation-strategy), [Standard Library](#standard-library)

### Composition (of primitives)

The process by which multiple primitive operations combine to produce a
derived language feature. A feature is well-formed when its decomposition
into primitives is unique and does not require additional semantics beyond
what the primitives provide.

- **Source:** `../what/PRIMITIVE_BLOCKS.md` § Composition Rules
- **See also:** [Primitive Block](#primitive-block), [Semantic Layer](#semantic-layer), [Dependency Flow](#dependency-flow)

### Comptime

Shorthand for "compile time." In Orthon, a unified execution phase where code with the `comptime` keyword is evaluated during compilation using the same language semantics as runtime code. Comptime serves three roles: **generics** (`comptime T: type` parameters replace `<T>` syntax), **reflection** (`@typeInfo`, `@field`, `@hasDecl` as comptime function calls), and **metaprogramming** (ordinary control flow executed at comptime to generate code).

Public comptime parameters MUST include explicit trait bounds for LLM discoverability. Private comptime parameters MAY omit bounds (duck-typed). Comptime evaluation is deterministic and sandboxed (no IO, filesystem, or network access).

```orthon
fun max(comptime T: type + Comparable, a: T, b: T) -> T
    return if a > b then a else b
```

- **Source:** `../what/concepts/COMPILE_TIME_EXECUTION.md`, EDR-031
- **See also:** [Combinator](#combinator), [Core Language](#core-language), [Macro](#macro)

### Context Parameter

A function parameter that is resolved automatically from the enclosing
scope rather than passed explicitly at the call site. In Orthon, context
parameters use the `require`/`using` dual-keyword model: `require`
declares a context dependency in a function or class signature, and
`using` provides the value at the call or construction site. This
replaces the single-keyword `using`/`using` model from EDR-037 with
an unambiguous keyword split (EDR-081).

```orthon
fun process(order_id: Int) require Database db, Logger log -> Receipt
process(42) using prod_db, prod_log
```

- **Source:** `../what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md` (EDR-081), EDR-037
- **See also:** [Dependency Slot](#dependency-slot), [Implicit Context Flow](#implicit-context-flow)

### Correctness by Construction (CbC)

A cross-cutting pattern in which the language makes invalid states
*structurally impossible to express*, so a program that compiles is
already correct with respect to the invariants the type system can
capture. Orthon realizes CbC through the composition of orthogonal
mechanisms: ownership + move semantics (invariants become local and
provable), value semantics by default, immutable-by-default bindings,
ADTs with exhaustive `match`, literal types, `Option`/`Result`, and
contracts. CbC is a *documented pattern*, not a design principle
(`DESIGN_PRINCIPLES.md` is locked).

- **Source:** `../how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`, [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) § Ownership (Formal foundation)
- **See also:** [Invariant Classification](#invariant-classification), [Constrained Type](#constrained-type)

---

## D

### Data

The primary abstraction in Orthon. Values viewed without imposed semantic meaning — the raw material that modifiers transform.

```
(1, 2, 3)    → data, without semantic label
tuple(1, 2, 3) → the same data, now explicitly a Tuple
```

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Data, `how/concepts/research/DATA_MODEL.md`
- **See also:** [Data Modifier](#data-modifier), [Representation](#representation)

### Data Modifier

A construct that transforms data from one representation to another. Modifiers express programmer intent; the compiler determines the most efficient implementation.

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Data Modifiers
- **See also:** [Data](#data), [Representation](#representation)

### Data Operations Primitive

A primitive block whose primary responsibility is transforming, accessing,
or controlling data rather than constructing it. The six Data Operations
primitives are `assignment`, `function`, `call`, `attribute access`,
`scope`, and `reference`. Distinguished from Data Primitives which
produce or structure data.

- **Source:** `../what/PRIMITIVE_BLOCKS.md`
- **See also:** [Data Primitive](#data-primitive), [Primitive Block](#primitive-block), [Data Modifier](#data-modifier)

### Default Value

A value assigned to a function or constructor parameter when no argument
is provided at the call site. Default values are ordinary expressions
evaluated at call time. Combined with named parameters, they eliminate
telescoping-constructor anti-patterns and reduce overload explosion.
In Orthon, default values are part of the general function call model
(not a constructor-specific mechanism).

```orthon
fn connect(host: String, port: Int = 80, useSsl: Bool = false)
connect(host: "example.com")    # port defaults to 80, useSsl to false
```

- **Source:** `../what/concepts/OBJECT_INITIALIZATION.md` (EDR-054), `../what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md` (EDR-065)
- **See also:** [Named and Optional Parameters](#named-and-optional-parameters), [Object Initialization](#object-initialization)

### Delegate

A concurrent execution context in Orthon's concurrency model. Created with the `delegate` keyword (or the `act` modifier on a type declaration), a delegate owns isolated state and communicates via message passing. Internally, each delegate is implemented as an actor with a mailbox and single-threaded message processing, but the programmer never writes `actor` or manages mailboxes directly.

```orthon
let counter = delegate(Counter(0))
counter <- increment()    # asynchronous message send
```

- **Source:** `../what/concepts/CONCURRENCY_MODEL.md`, EDR-033
- **See also:** [Exclusive Access](#exclusive-access), [Foreign Function Interface (FFI)](#foreign-function-interface-ffi)

### Deferred Invocation

The suspendable computation returned by submitting an invocation to a
`defer` execution context (`ctx <- fn(...)` with `ctx = defer(obj)`).
It is the unit of work scheduled into the coroutine context — semantically
a coroutine-like suspendable computation, distinct from the context itself.
Awaiting it (`await(ctx)`) yields until the computation is ready. The exact
type name is deferred to Phase 5; the Core Language commits to the concept,
not the name.

- **Source:** `../how/concepts/research/essential/EXECUTION_CONTEXT_INVOCATION.md` (OQ2 Resolution)
- **See also:** [Delegate](#delegate), [Await](#await), [Future](#future)

### Dependency Slot

A class-level `require` declaration that groups shared context
dependencies for all methods in a class, filled per-instance at
construction time. Dependency slots carry a compile-time initialization
guarantee (no `null`, no uninitialized state) and enable prod/test
differentiation at instance granularity.

```orthon
class UserService require Database db, Logger log:
    fun find_user(id: Int) -> Option[User]
        // db and log are available without redeclaration

let prod_svc = UserService(using prod_db, prod_log)
let test_svc = UserService(using test_db, test_log)
```

- **Source:** `../what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md` (EDR-081)
- **See also:** [Context Parameter](#context-parameter), [Implicit Context Flow](#implicit-context-flow)

### Data Primitive

A primitive block whose primary responsibility is producing or structuring
data values. The three Data primitives are `literal` (value creation),
`identifier` (value naming), and `pack/unpack` (value composition/
decomposition). Distinguished from Data Operations Primitives which transform
or access data rather than constructing it.

- **Source:** `../what/PRIMITIVE_BLOCKS.md`
- **See also:** [Data Operations Primitive](#data-operations-primitive), [Primitive Block](#primitive-block), [Data](#data)

### Decision Log

The detailed, per-gate reasoning trail behind a Tier 1–2 decision —
one entry per validated decision, one subsection per gate applied,
each working through that gate's method against the actual proposal
and recording the verdict it produces. Distinct from an artifact's own
terse Validation summary (verdict + citation only) and from the
Decision Journal (a one-row-per-decision index); the Decision Log is
where the reasoning that produced a verdict is actually recorded, not
just its conclusion.

- **Source:** `../how/gates/DECISION_LOG.md`, established by
  [EDR-014](../how/decision_records/process/EDR-014-decision-log.md)
- **See also:** [Decision Validation](#decision-validation), [Validation Gate](#validation-gate)

### Decision Validation

A framework of seven independent validation gates that every language
design decision must pass. Each gate examines the proposal from a
different perspective: user value, logical consistency, conceptual
simplicity, architectural integrity, implementation independence,
long-term maintainability, and LLM generability.

- **Source:** `../how/gates/DECISION_VALIDATION.md`
- **See also:** [Language Design Gate](#language-design-gate), [Validation Gate](#validation-gate)

### Declaration Kind (`fun` / `proc` / `new`)

The three mutually exclusive function-declaration kinds that carry
Orthon's Mutation contract at the declaration site rather than the call
site: **`fun`** (read-only, never mutates `self`, always returns a
value), **`proc`** (mutates `self`, identity preserved, may return a
value or nothing), and **`new`** (never mutates `self`, always produces
a distinct value). There is no `mut` modifier and no caller-side
annotation — the contract lives entirely in which of the three keywords
a declaration uses.

- **Source:** `../what/SEMANTIC_MODEL.md` § Mutation
- **See also:** [Explicit Semantics](#explicit-semantics), [Semantic Dimension](#semantic-dimension)

### Declaration Model (Unified)

Variables, functions, types, classes, and modules follow the same declaration principles and modifier system. No special-case declaration syntax for different kinds of entities.

- **Source:** `../why/MANIFESTO.md` § A unified declaration model

### Derive

A declarative annotation (`@derive(TraitA, TraitB)`) that instructs the compiler to generate trait implementations automatically. The compiler consults a registry of `derive`-compatible macro implementations. If a named trait has no registered derive macro, the compiler reports an error.

```orthon
@derive(Show, Eq, Clone)
type Point(x: Int, y: Int)
```

`@derive` is syntactic sugar over the `@macro` mechanism — each derive target invokes a registered macro function that generates the corresponding `impl Trait for Type` block.

- **Source:** `../what/concepts/AST_MACROS.md` § Derive Sugar
- **See also:** [Macro](#macro), [Comptime](#comptime)

### Dependency Flow

The central invariant of the Semantic Dependency Architecture: each
semantic layer depends only on layers below it; no layer may reference
or rely on constructs from a layer above it. This creates a natural
Dependency Inversion at the language level, enforced architecturally
rather than by convention.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Language Pattern](#language-pattern), [Primitive Operation](#primitive-operation), [Semantic Layer](#semantic-layer)

### Deterministic Behavior

The same source code must produce identical observable behavior across optimization levels and implementations. Only performance characteristics may differ.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Deterministic Behavior
- **See also:** [Explicit Semantics](#explicit-semantics), [Semantics Before Optimization](#semantics-before-optimization)

---

### Declaration by Assignment

A variable declaration model where the first assignment to an identifier
introduces the variable. No separate `let`/`var`/`Type name` keyword is
needed for initial declaration. The type is inferred from the initializer.
Shadowing requires an explicit `let` keyword. Read-before-write is a
compile-time error via definite assignment analysis. Variables are
immutable by default — `mut` required for reassignment. No implicit
globals — assignment inside a function always creates a local variable.

```orthon
x = 42               # declaration by first assignment, type inferred as Int
mut counter = 0      # mutable variable
let x = transform(x) # shadowing: new variable (explicit let)
```

- **Source:** `../what/concepts/DECLARATION_BY_ASSIGNMENT.md` (EDR-074)
- **See also:** [Definite Assignment Analysis](#definite-assignment-analysis), [Type Inference](#type-inference), [Shadowing](#shadowing)

### Declarative Construct

A language or StdLib pattern where the programmer specifies *what* the
result should be, and the language/StdLib determines *how* to achieve
it. Orthon provides declarative constructs for common data
transformations: collection operations (map/filter/reduce), resource
management (using), sorting (sorted-by-key), derived serialization, and
derived structural methods (eq/hash/copy). Every declarative construct
has a documented desugaring to imperative primitives. Query expressions
are deferred to v0.2.

```orthon
let adults = users.filter(u -> u.age >= 18).map(u -> u.name)
```

Five synthesis-friendliness criteria: single intent, deterministic
output, type-checked inputs, canonical form, LLM-guessable name.

- **Source:** `../what/concepts/DECLARATIVE_CONSTRUCTS.md` (EDR-073)
- **See also:** [Combinator](#combinator), [Iterator Protocol](#iterator-protocol)

### Declarative Multi-Key Sort

A StdLib API for sorting by multiple fields using key paths, without
manual comparator chains. Builds on stable sort (SORTING) and the `Ord`
trait (EQUALITY). Supports direction modifiers (`asc`/`desc`).

```orthon
let sorted = users.sorted(by: .last_name, .first_name)
let by_age_then_salary = users.sorted(by: .age, desc(.salary))
```

All forms desugar to lexicographic `Ord` comparisons on tuples. No new
language semantics.

- **Source:** `../what/concepts/DECLARATIVE_MULTI_KEY_SORT.md` (EDR-067)
- **See also:** [Sorting](#sorting-stable), [Equality](#semantic-equality), [Ord](#ord-trait)

### Definite Assignment Analysis

A compiler-level analysis that verifies every variable is assigned on all
possible execution paths before any read occurs. Read-before-write in any
path is a compile-time error. Tracks assignment across control flow
(if/else, loop, match arms). Conservative — if the compiler cannot prove
assignment, the read is an error. Part of Orthon's Declaration by
Assignment model (EDR-074).

```orthon
# x = compute()    # Error: read before write on some paths
if condition:
    x = 1
else:
    x = 2
print(x)            # OK: assigned on all paths
```

- **Source:** `../what/concepts/DECLARATION_BY_ASSIGNMENT.md` (EDR-074)
- **See also:** [Declaration by Assignment](#declaration-by-assignment), [Flow-Sensitive Narrowing](#flow-sensitive-narrowing)

### Derive Serialization

The automatic generation of serialization/deserialization code for a
type via the `@derive(Serialize, Deserialize)` macro. Format-agnostic —
JSON, binary, and custom formats implement `Encoder`/`Decoder` traits.
Declarative annotations for field customization (`@name`, `@skip`).
Deserialization returns `Result<T>` (never panics). Cyclic reference
tracking deferred to v0.2.

```orthon
@derive(Serialize, Deserialize)
struct User:
    name: String
    @name("email_address") email: String
```

- **Source:** `../what/concepts/DERIVE_SERIALIZATION.md` (EDR-070)
- **See also:** [Derive](#derive), [Macro](#macro), [Result Type](#result-type)

## E

### EDR (Engineering Decision Record)

A unified record of a consequential engineering decision, classified
by one of 15 categorical lenses: Architecture, Technology, Tooling,
Process, Delivery, Operations, Quality, Security, Governance, Data,
AI, Documentation, Knowledge, Collaboration, or Product.

EDRs replace the earlier dual ADR (Architecture) / TDR (Tools) system.
ADR and TDR are now categories of EDR. All EDRs are stored in
`docs/how/decision_records/{category}/EDR-NNN-slug.md` with a master
index at `decision_records/INDEX.md`.

- **Source:** `../how/decision_records/process/EDR-001-edr-system.md`
- **See also:** [Architecture](#architecture), [Decision Validation](#decision-validation)

### Exclusive Access

The requirement that mutation may only proceed when no other live
reference can observe the value mid-mutation: many shared (read)
borrows may coexist, but at most one exclusive (write) borrow may
exist, and never both at once ("shared XOR mutable"). This is
Semantic Invariant 2, viewed as the single access-control rule that
both Ownership (borrowing) and Mutation (`proc` calls) enforce from
their own angle — a `proc` call is only legal when the compiler can
establish exclusive access to its receiver.

- **Source:** `../what/SEMANTIC_MODEL.md` § Semantic Invariants, § Ownership, § Mutation
- **See also:** [Semantic Invariant](#semantic-invariant), [Declaration Kind (`fun` / `proc` / `new`)](#declaration-kind-fun--proc--new), [Binding Identity](#binding-identity)

### Error Propagation

The mechanism by which errors in a `Result<T, E>` type are passed upward
through the call stack. In Orthon, propagation is **explicit** via the `?`
operator — `expr?` unwraps an `Ok` value or returns the `Error` variant
immediately from the enclosing function. There is no implicit or hidden
propagation (no exceptions, no unchecked fallibility).

```orthon
fun read_config(path: String) -> Result<Config, IOError>
    data = fs.read_file(path)?  # Error propagated upward
    parse_config(data)
```

- **Source:** `../what/concepts/ERROR_HANDLING.md` § Model
- **See also:** [Result Type](#result-type), [Option Type](#option-type)

### Error Set

The set of possible error tags that a fallible function can produce. In
Orthon's Error Union model, error sets are **inferred** by the compiler:
the function body's `!T` return type triggers automatic discovery and
union of every error tag from every fallible call inside the body.
Inferred error sets grow and shrink automatically as the implementation
changes.

```orthon
fun foo() -> !T
    bar()?   # adds bar's error tags
    baz()?   # adds baz's error tags
# error set = union of bar's and baz's error tags
```

Explicit error set declaration is available as an opt-in for
documentation purposes. Structural widening (subset → superset coercion)
is implicit — no explicit conversion required.

- **Source:** `../what/concepts/ERROR_UNION.md` § Model
- **See also:** [Error Tag](#error-tag), [Error Union](#error-union), [Structural Widening](#structural-widening)

### Error Tag

A unit-like, payload-free error value in Orthon's Error Union model.
Error tags are simple identifiers without associated data — they signal
which error occurred, not additional context about why.

```orthon
error.FileNotFound
error.AccessDenied
error.ParseError
error.Timeout
```

Tags are structurally comparable: `error.FileNotFound` is a distinct
value from `error.AccessDenied`. When an error needs associated data
(line numbers, field names, validation details), use `Result<T, E>`
explicitly with a payload-bearing error type.

- **Source:** `../what/concepts/ERROR_UNION.md` § Model
- **See also:** [Error Set](#error-set), [Error Union](#error-union)

### Error Union

A distinct type former `!T` representing a fallible operation whose
error side is an inferred set of unit-like error tags. The `!T` type is
not sugar for `Result<T, E>` — it is a distinct kind of type with
different properties: inferred error sets, structural widening, and
tag-only error values.

```orthon
fun read_config(path: String) -> !Config
    let data = fs.read_file(path)?
    return parse_config(data)
```

Error Union coexists with `Result<T, E>` for payload-bearing errors.
Both use the same `?` operator for propagation.

- **Source:** `../what/concepts/ERROR_UNION.md` (EDR-023)
- **See also:** [Error Set](#error-set), [Error Tag](#error-tag), [Error Propagation](#error-propagation), [Structural Widening](#structural-widening)

### Exhaustiveness

The compiler-enforced requirement that all cases of a sum type, enum,
or sealed trait hierarchy are covered in a `match` expression or
dispatch declaration. Missing cases are a compile-time error, not a
warning — the compiler enumerates every unhandled variant.

```orthon
match opt:
    case Some(x) => process(x)
    # Error: `None` not handled
```

Exhaustiveness applies to: all variants of an enum/sum type, `Some`/`None`
for `Option<T>`, `Ok`/`Error` for `Result<T, E>`, all cases of a sealed
trait hierarchy. A wildcard `_` satisfies exhaustiveness for remaining
cases.

- **Source:** `../what/concepts/PATTERN_MATCHING.md` § Model (EDR-025)
- **See also:** [Pattern Matching](#pattern-matching), [Pattern Matching Dispatch](#pattern-matching-dispatch), [Option Type](#option-type)

### Execution Descriptor

A declarative, first-class manifest of what a program requires to
execute. Describes language version, runtime, standard library,
dependencies, implementation strategy, target platform, permissions,
resources, and configuration. The Descriptor is explicitly declared
and versioned alongside the Program — it is never inferred from
filesystem layout or environment variables.

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Execution Descriptor
- **See also:** [Execution Program](#execution-program), [Program Enricher](#program-enricher)

### Execution Engine

Any consumer that takes an Execution Program and does something with
it — runs, debugs, tests, compiles, materialises, or deploys.
Interpreters, REPLs, notebook kernels, test runners, debuggers, JIT
compilers, AOT compilers, OCI builders, MicroVM builders, and WASM
builders are all Execution Engines. They differ only in *how* they
execute, not in *what* they execute.

```
Execution Program
    ├── Interpreter
    ├── REPL
    ├── Notebook Kernel
    ├── Test Runner
    ├── Debugger
    ├── JIT / AOT Compiler
    ├── OCI Builder
    ├── MicroVM Builder
    ├── WASM Builder
    └── Remote Executor
```

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Execution Engine
- **See also:** [Execution Program](#execution-program), [Program Enricher](#program-enricher)

### Execution Model Policy

An Implementation Policy that controls how a fully-defined Execution
Program is materialised and consumed. Part of the Strategy system
([EDR-036](../how/decision_records/architecture/EDR-036-execution-program.md)).
Defines five values: `Interpreted`, `AOT`, `JIT`, `WASM`, `Container`.
Classified as Policy, not Language — execution strategy is a HOW
decision (how to run), not a WHAT decision (what the program means).

```text
DEFAULT_STRATEGY: Execution Model = AOT
```

- **Source:** `../how/IMPLEMENTATION_POLICIES.md` § Execution Model Policy, [EDR-036](../how/decision_records/architecture/EDR-036-execution-program.md)
- **See also:** [Execution Program](#execution-program), [Implementation Strategy](#implementation-strategy), [Policy](#policy)

### Execution Program

The central abstraction of the Orthon execution model. A fully-defined,
reproducible, content-addressed canonical representation of everything
needed to execute a program. Includes source code, runtime, standard
library, dependencies, implementation strategy, configuration,
permissions, resources, and metadata.

The Execution Program is the **canonical representation** — not a
binary, not a Docker image, not a virtual machine. It is an
infrastructure-independent model of an executable program.

```
Execution Program
├── Program
├── Runtime
├── Standard Library
├── Dependencies
├── Implementation Strategy
├── Configuration
├── Permissions
├── Resources
└── Metadata
```

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md`, `../how/architecture/ARCHITECTURE.md` § Execution Program Pipeline
- **See also:** [Execution Descriptor](#execution-descriptor), [Execution Engine](#execution-engine), [Program Enricher](#program-enricher)

### Explicit Optimization

Performance-oriented execution strategies are enabled intentionally by the programmer, never applied silently. The default execution model favors predictability over performance.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Explicit Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Semantics Before Optimization](#semantics-before-optimization)

### Explicit Semantics

Whenever an operation changes the meaning, lifetime, ownership, or behavior of data, it must be expressed explicitly in the syntax. No hidden conversions or implicit semantic changes.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Explicit Semantics
- **See also:** [Deterministic Behavior](#deterministic-behavior)

---

## F

### Frame Condition

The set of memory locations a computation is allowed to modify — the
*absence* side of a function contract. A postcondition (`ensures`)
states what a function guarantees; the frame condition states what it
does *not* touch. In Orthon, the declaration kinds already provide a
coarse frame condition on `self` (`fun` and `new` never mutate `self`;
`proc` does). The residual case — free functions and non-`self` state
(globals, I/O, dependency slots) — is the subject of the
[`FRAME_CONDITIONS.md`](../how/concepts/research/deferrable/FRAME_CONDITIONS.md)
hypothesis, which proposes a `@modifies` doc annotation rather than a
language keyword. Grounded in Separation Logic's spatial separation.

- **Source:** `../how/concepts/research/deferrable/FRAME_CONDITIONS.md`, [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) § Ownership (Formal foundation)
- **See also:** [Contract (Design by Contract)](#contract-design-by-contract)

### For Loop

The iteration construct in Orthon: `for item in sequence`. The only loop
for consuming values from a sequence. Operates on any type implementing
`IntoIterator[T]`. Desugars to the iterator protocol per EDR-022.

```orthon
for item in items:
    process(item)

for i in 0..len(array):     # index-based via range
    process(array[i])
```

Key properties: no C-style `for (;;)` — index-based iteration uses range
syntax; destructuring in loop variables; `break` and `continue` available.

- **Source:** `../what/concepts/ITERATION_LOOP.md` (EDR-053)
- **See also:** [Iterator Protocol](#iterator-protocol), [While Loop](#while-loop)

### Foreign Function Interface (FFI)

A mechanism that allows Orthon programs to call functions written in
other languages (primarily C). The FFI defines the boundary between
Orthon's type system and memory model and those of foreign languages.

- **Source:** `../when/ROADMAP.md` § Milestone 8
- **See also:** [Standard Library](#standard-library)

### Future

The return type of an `async` function. `Future<T>` represents a value
of type `T` that may not be available yet. Futures are first-class values
— they can be stored, passed, and combined without forcing evaluation.

```orthon
let f = async fetch(url)     # Future[String], no suspension
let result = await f         # suspend, unwrap to String
```

Key properties: `await` resolves a `Future` to its value; calling an
`async` function without `await` produces a `Future` without suspension
(colourless model); `Future` is single-subscriber by default (one consumer
can await it).

- **Source:** `../what/concepts/ASYNC_AWAIT.md` (EDR-047)
- **See also:** [Async](#async), [Await](#await), [Spawn](#spawn)

### Flow-Sensitive Narrowing

The compiler's ability to track type information across control flow
edges, narrowing an `Option<T>` to `T` after a check that establishes
the value is present. After `match value { case Some(x) => ... }`, the
compiler knows `value` is `T` in the `Some` arm. After `if value != None
{ ... }`, the compiler narrows within the true branch.

Narrowing is **per-variable** (follows the variable's type through each
control flow path), **flow-sensitive** (different branches may have
different narrowed states), and **conservative** (if the compiler cannot
prove a value is non-null, it remains `Option<T>`). Narrowing resets on
variable reassignment.

- **Source:** `../what/concepts/TYPE_LEVEL_NULL_SAFETY.md` (EDR-028)
- **See also:** [Option Type](#option-type), [Pattern Matching](#pattern-matching), [Narrowing](#narrowing)

### Fresh-Value Exemption

The rule that an unbound temporary — a literal, a constructor call, or
any expression result not yet bound to a name — has no prior owner and
no aliasable Binding Identity, and so may be passed directly into an
ownership-consuming context (e.g. `delegate(List())`) without an
explicit transfer marker. Once a value is bound to a name, the
exemption no longer applies and any subsequent transfer must be
syntactically explicit.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity, § Ownership
- **See also:** [Binding Identity](#binding-identity), [Semantic Dimension](#semantic-dimension)

---

## G

### Generics

Trait-bounded parametric polymorphism — the ability to write functions
and types that operate uniformly across different concrete types while
preserving type safety and performance. Generic parameters are
constrained by traits that specify required operations.

```orthon
fn identity[T](value: T) -> T
    return value

fn sort[T](items: [T]) where T: Ordered + Printable
```

Static dispatch by default (monomorphisation). Dynamic dispatch via
`dyn Trait` is opt-in. Generic type parameters are invariant by default;
covariance/contravariance are declared via trait method signatures. No
HKT or negative bounds in v0.1.

- **Source:** `../what/concepts/GENERICS.md` (EDR-024)
- **See also:** [Trait](#trait), [Trait Bound](#trait-bound), [Type Inference](#type-inference), [Monomorphisation](#monomorphisation)

### Generator

A function that produces a sequence of values lazily, one at a time,
using the `emit` keyword. Generator bodies are compiled into state
machines that implement `Iterator[T]`. Generators are the production
side of the sequence model — the iterator protocol is the consumption
side.

```orthon
fun natural_numbers() -> Iterator[Int]
    let i = 0
    loop:
        emit i
        i = i + 1
```

Three equivalent canonical forms: `emit value`, `return sequence(value)`,
and `return value ->`. Generators are lazy by default (values produced
on demand) and support infinite sequences. Composition with iterator
combinators does not allocate intermediate collections.

**Bidirectional generators** (EDR-050) extend the model with `yield`,
which optionally receives a value from the consumer.

- **Source:** `../what/concepts/LAZY_SEQUENCE_GENERATORS.md`
- **See also:** [Iterator Protocol](#iterator-protocol), [Lazy Sequence](#lazy-sequence), [Yield](#yield)

### Generator Expression

A parenthesised inline syntax for producing lazy sequences without
writing a full generator function. Desugars to an anonymous generator
function.

```orthon
let squares = (x * x for x in 1..10)
let evens = (x for x in 1..100 if x % 2 == 0)
```

Generator expressions are lazy by default — they produce an
`Iterator[T]` without materialising. Optional `if` filter clause.

- **Source:** `../what/concepts/GENERATORS.md` (EDR-050)
- **See also:** [Generator](#generator), [Lazy Sequence](#lazy-sequence)

---

## I

### Invariant Classification

A three-tier taxonomy of the invariants a language can enforce, defined
in [`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md):

1. **Tier 1 — Type-level invariants:** proven by the compiler (literal
types, ADTs with exhaustive `match`, ownership, immutable-by-default).
2. **Tier 2 — Contract-level invariants:** verified at debug/test time
via `requires`/`ensures`/`invariant`. Constrained Types (EDR-080)
straddle Tiers 1–2: declared at the type level, enforced at runtime
boundaries.
3. **Tier 3 — External invariants:** outside the language ("this list is
sorted"), verified by tests or external tools.

Correctness by Construction is strongest at Tier 1.

- **Source:** `../how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md` § Invariant Classification
- **See also:** [Correctness by Construction (CbC)](#correctness-by-construction-cbc), [Constrained Type](#constrained-type), [Contract (Design by Contract)](#contract-design-by-contract)

### Implicit Context Flow

A cross-cutting concern of the Evaluation and Visibility dimensions where
some function parameters are resolved automatically from the enclosing
scope rather than passed explicitly at the call site. Context parameters
(analogous to Scala 3's `given`/`using`) are the primary mechanism:
a `using` block in the function signature declares a context dependency,
and the compiler resolves a matching `given` instance from scope.

```orthon
fun sort[A](list: List[A])(using ord: Ord[A]): List[A]
```

Context parameters are noted as a SEMANTIC_MODEL correction
([EDR-037](../how/decision_records/architecture/EDR-037-context-parameters.md))
but full specification is deferred beyond v0.1.

### Immutable Date-Time

A set of immutable, value-semantics date/time types in the Standard
Library: `Instant`, `LocalDate`, `LocalTime`, `LocalDateTime`,
`ZonedDateTime`, `Duration`, `Period`, `Offset`. All types are
immutable — "modification" methods return new instances. Formatters
are immutable objects. Parsing returns `Result<T>`. Thread-safe by
construction.

```orthon
let now = ZonedDateTime.now()
let tomorrow = now.plusDays(1)    # new instance, now unchanged
let parsed = LocalDate.parse("2026-07-27")
```

- **Source:** `../what/concepts/IMMUTABLE_DATE_TIME.md` (EDR-068)
- **See also:** [Result Type](#result-type), [Value Semantics](#value-semantics)

### Immutable Marker Trait

A compiler-recognized trait (`Immutable`) that guarantees values of the
implementing type cannot be mutated. The compiler uses this marker for
optimisations (eliding copies, allowing hash-key usage, safe concurrent
sharing). No methods — purely a guarantee. Accepted in v0.1 as an
interface contract; full persistent collection implementations deferred
to v0.2.

```orthon
trait Immutable    # marker trait, no methods
```

- **Source:** `../what/concepts/PERSISTENT_DATA_STRUCTURES.md` (EDR-069)
- **See also:** [Persistent Data Structure](#persistent-data-structure), [Value Semantics](#value-semantics)

### Intersection Type

A structural, purely compile-time type combinator that merges the shape of
two types into a new anonymous type with all members of both
(`A & B`). Intersection types produce no runtime object — they exist only
in the type checker. **NOT accepted for Orthon v0.1** — intersection types
are redundant with Orthon's product type mechanism (declaring a new named
record type carrying all fields of both is more explicit).

- **Source:** `../what/concepts/UNION_INTERSECTION_TYPES.md`, [EDR-045](../how/decision_records/architecture/EDR-045-union-intersection-types.md)
- **See also:** [Union Type](#union-type), [Algebraic Data Type](#algebraic-data-type), [Product Type](#product-type)

- **Source:** `../how/concepts/research/essential/CONTEXT_PARAMETERS.md`, [EDR-037](../how/decision_records/architecture/EDR-037-context-parameters.md)
- **See also:** [Given Resolution](#given-resolution), [Semantic Dimension](#semantic-dimension), [Evaluation Dimension](#evaluation-dimension)

### Implementation Strategy

A named set of Policies that governs how language semantics are
realised. A Strategy is a coherent profile — for example, the
Default Strategy uses `Arena` allocation while the Embedded Strategy
uses `Static` allocation. Strategies are interchangeable: replacing
one must not change program semantics.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Implementation Strategy, `../how/strategies/IMPLEMENTATION_STRATEGIES.md`
- **See also:** [Architecture](#architecture), [Core Language](#core-language), [Policy](#policy), [Standard Library](#standard-library)

### Intent Over Implementation

The programmer describes *what* should happen; the compiler decides *how* to implement it efficiently.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Intent Over Implementation
- **See also:** [Explicit Semantics](#explicit-semantics)

### Intermediate Result

A value produced by a function via `emit` during the course of a
long-running computation, distinct from the final `return` value. A
function with both `emit` and `return` produces an `Iterator[T]` of
intermediate values; the final return value is accessible via
`.final()`.

```orthon
fun process_dataset(data: Dataset) -> Iterator[BatchResult]
    for batch in data.batches():
        emit analyse(batch)
    return compute_summary(data)
```

The intermediate-result model is a specification refinement of
LAZY_SEQUENCE_GENERATORS (EDR-021): the `emit` keyword already supports
this pattern technically; this concept documents the dual use explicitly.

- **Source:** `../what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` (EDR-052)
- **See also:** [Generator](#generator), [Lazy Sequence](#lazy-sequence), [Iterator Protocol](#iterator-protocol)

### IntoIterator

A trait that enables a type to be converted into an `Iterator[T]` for
use in `for` loops and combinator chains. Collections implement
`IntoIterator[T]` to enable direct iteration. `Iterator[T]` itself
implements `IntoIterator[T]` (returning `self`), so both iterators and
collections work uniformly with `for`.

```orthon
trait IntoIterator[T]
    fn iter(self) -> Iterator[T]
```

- **Source:** `../what/concepts/ITERATOR_PROTOCOL.md` § IntoIterator[T] for Collections
- **See also:** [Iterator Protocol](#iterator-protocol), [Generator](#generator)

### Iterator Protocol

The consumption side of Orthon's sequence model. Defined by the
`Iterator[T]` trait:

```orthon
trait Iterator[T]
    fn next(self) -> Option[T]
```

Key properties: **lazy** (elements produced on demand), **single-pass**
(consumed on traversal), **composable** (combinators return new
iterators without intermediate allocation), **zero-cost** (monomorphisation
eliminates combinator overhead).

The `for` loop desugars to the iterator protocol: `IntoIterator::iter()`
+ `loop` calling `next()`. Protocol method access uses the `@` prefix
per D-07 (`iterator@next()`). Standard combinators (map, filter, take,
fold, collect, etc.) are default method implementations on `Iterator[T]`
living in the Standard Library.

- **Source:** `../what/concepts/ITERATOR_PROTOCOL.md`
- **See also:** [IntoIterator](#intoiterator), [Generator](#generator), [Lazy Sequence](#lazy-sequence)

---

## L

### Lambda

An anonymous function expression — a function value defined inline without a
name. Lambdas are first-class values and share the same call syntax as named
functions. Closure capture is explicit.

```
let double = fn (x: Int) -> Int
    x * 2

double(3)  // 6
```

- **Source:** `../how/concepts/research/essential/FUNCTIONS.md`
- **See also:** [Canonical Form](#canonical-form), [Language Pattern](#language-pattern)

### Language Design Gate

A quality checklist that every language design proposal must satisfy
before entering formal specification. Operationalises the Decision
Validation gates into a review form.

- **Source:** `../how/gates/_language-design.md`
- **See also:** [Decision Validation](#decision-validation), [Validation Gate](#validation-gate)

### Language Pattern

A construct in the Semantic Dependency Architecture (Level 2) that
composes Primitive Operations (Level 1) and Data Model (Level 0)
constructs to provide convenience without adding new semantics. A
Language Pattern can be fully expressed as a composition formula
showing which lower-level constructs produce it. Examples:
`context` = closure + callable + try/catch + defer;
`decorator` = callable + closure + metadata.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Dependency Flow](#dependency-flow), [Primitive Operation](#primitive-operation), [Semantic Layer](#semantic-layer)

### Language Consistency Principles

The set of cross-cutting design rules that ensure uniform behaviour across Orthon:

- **Representation Symmetry** — Reversible transformations use symmetric syntax (prefix creates a representation, postfix restores it).
- **Deterministic Behavior** — Same source, same observable behaviour (see [Deterministic Behavior](#deterministic-behavior)).
- **Stable Mental Model** — Programmers reason about language semantics, not compiler internals.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Language Consistency

### Language Server Protocol (LSP)

An open protocol that standardises communication between a language
server — which provides language features like autocompletion,
diagnostics, and go-to-definition — and a code editor or IDE. Orthon's
tooling implements an LSP server to provide IDE support.

- **Source:** `../when/ROADMAP.md` § Milestone 9
- **See also:** [LLM Toolchain](#llm-toolchain)

### Lazy Sequence

A sequence whose elements are produced on demand rather than eagerly
materialised. In Orthon, lazy sequences are the default: generator
functions produce values one at a time via `emit`, and iterator
combinators (map, filter, take, etc.) compose without allocating
intermediate collections. Materialisation is explicit via
`.collect()`.

Lazy by default is established by Phase 3 D-06: the `emit` keyword
guarantees lazy evaluation. Infinite sequences are valid — the
consumer controls termination via combinators like `.take(n)`.

- **Source:** `../what/concepts/LAZY_SEQUENCE_GENERATORS.md` § Principles
- **See also:** [Generator](#generator), [Iterator Protocol](#iterator-protocol)

### Literal Type

A singleton type inhabited by exactly one value — a specific string like
`"GET"`, number like `42`, or boolean like `true`. Literal types compose
with union types (`|`) into closed sets: `type Method = "GET" | "POST" | "PUT"`.

Widening rule: immutable bindings preserve literal types (`let x = "GET"`
→ type `"GET"`); mutable bindings widen to base type (`var y = "GET"`
→ type `String`). This is a single, always-applicable rule — not
context-dependent.

Literal types are restricted to primitive scalars (`String`, `Int`,
`Float`, `Bool`) in v0.1. They are input to type-level computation
intrinsics like `KeyOf<T>`.

```orthon
let port: 80 | 443 = 80
type Method = "GET" | "POST" | "PUT"
```

- **Source:** `../what/concepts/LITERAL_TYPES.md`, [EDR-043](../how/decision_records/architecture/EDR-043-literal-types.md)
- **See also:** [Union Type](#union-type), [Widening](#widening), [Type-Level Computation](#type-level-computation), [Algebraic Data Type](#algebraic-data-type)

### LLM-Native

A design property of the language and its toolchain: Orthon is
*LLM-native*, not merely *LLM-compatible*. This means every language
construct is exposed as structured, machine-readable schema (not just
free-text documentation); the toolchain is strategy-aware and adjusts
its output based on active Policies; and all interfaces are
bidirectional — LLMs both consume language metadata and produce
enriched artifacts back into the toolchain.

- **Source:** `../how/architecture/ARCHITECTURE.md` § LLM Toolchain — Design Rationale
- **See also:** [LLM Strategy](#llm-strategy), [LLM Toolchain](#llm-toolchain)

### LLM Strategy

A named set of Policies governing how LLM-based tools interact with
Orthon code — covering syntax tolerance, verbosity, style, toolchain
activation, error formatting, and inference depth. An LLM Strategy
is a specialised Implementation Strategy that applies to the
LLM Toolchain layer rather than the execution environment.

- **Source:** `../how/IMPLEMENTATION_POLICIES.md`, `../how/architecture/ARCHITECTURE.md` § LLM Toolchain
- **See also:** [Implementation Strategy](#implementation-strategy), [LLM Toolchain](#llm-toolchain), [Policy](#policy)

### LLM Toolchain

A first-class architectural layer of tools and services that enable
LLMs to generate, analyse, complete, and transform Orthon code with
high fidelity. Components include the Schema Provider, Code Completer,
Code Generator, Static Analyser, Documentation Generator, and
Refactor/Migration Tool. The Toolchain sits alongside the Language and
Execution layers with bidirectional interfaces.

- **Source:** `../how/architecture/ARCHITECTURE.md` § LLM Toolchain
- **See also:** [LLM Strategy](#llm-strategy), [LLM-Native](#llm-native), [Architecture](#architecture)

---

## M

### Minimal Core

Complex language features should emerge from composition of simple primitives rather than from introducing new keywords or execution models. The language is *grown*, not *invented*.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Minimal Core, `../why/MANIFESTO.md` § Minimal core, maximum expressiveness
- **See also:** [Orthogonality](#orthogonality)

### Macro

In Orthon, a function annotated with `@macro` that operates on typed AST nodes at compile time and returns typed AST nodes. Macros are ordinary Orthon functions — there is no separate macro-definition language. Macros execute via the comptime mechanism (EDR-031).

Key properties: **hygienic by default** (identifiers scoped to expansion), **typed AST contracts** (input/output types verified), **single-pass expansion** (no recursive macros), **side-effect-free** (no IO, no filesystem).

The `@derive` annotation is declarative sugar over the macro mechanism.

```orthon
@macro
fun json_serialize(type_def: TypeDef) -> Vec<ImplBlock>
    # inspect type_def, generate ImplBlock nodes
```

- **Source:** `../what/concepts/AST_MACROS.md`, EDR-029
- **See also:** [Comptime](#comptime), [Derive](#derive), [Hygienic Macro](#hygienic-macro)

### Metadata Protocol

The `@`-prefixed convention for accessing metadata and protocol methods
on types and values. Examples: `list@len()`, `obj@fields`, `type@name`.
Distinct from attribute access (`.`); the `@` prefix makes metadata access
syntactically visible per Semantic Purity.

- **Source:** `../what/PRIMITIVE_BLOCKS.md` § Metadata Protocol
- **See also:** [Primitive Block](#primitive-block), [Semantic Purity](#semantic-purity), [Operator Equivalence](#operator-equivalence)

---

## N

### Named and Optional Parameters

A call-site convention where function arguments can be specified by
parameter name (in any order) and parameters can declare default values
(reducing overload explosion). In Orthon, this is a StdLib/macro
feature — named arguments desugar to positional calls via the macro
layer (EDR-029). Default values are ordinary expressions evaluated at
call time.

```orthon
fn connect(host: String, port: Int = 80, useSsl: Bool = false)
connect(host: "example.com", useSsl: true)    # named, skip port
```

- **Source:** `../what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md` (EDR-065)
- **See also:** [Derive](#derive), [Macro](#macro), [Object Initialization](#object-initialization)

### Named Parameter

A function or constructor parameter that can be referenced by name at
the call site, enabling readable invocations and arbitrary argument
order. In Orthon, named parameters are part of the general function call
model and desugar to positional calls via the macro layer (EDR-029).

- **Source:** `../what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md` (EDR-065), `../what/concepts/OBJECT_INITIALIZATION.md` (EDR-054)
- **See also:** [Named and Optional Parameters](#named-and-optional-parameters), [Default Value](#default-value)

### Named Before Symbolic

Every symbolic operator must have an equivalent named function. Symbols improve brevity; named functions improve readability. Both express the same semantics.

Example: `&x` and `ref(x)` are equivalent.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Named Before Symbolic
- **See also:** [Canonical Form](#canonical-form), [Operator Equivalence](#operator-equivalence)

---

## O

### Operator Equivalence

Every language operator has an equivalent named function. Operators are syntactic sugar over ordinary functions.

```
&x          == ref(x)
x->         == sequence(x)
```

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Operators and Named Functions, `../how/DESIGN_PRINCIPLES.md` § Named Before Symbolic
- **See also:** [Canonical Form](#canonical-form), [Named Before Symbolic](#named-before-symbolic)

### Option Type

The canonical representation of optional values in Orthon. A sum type with two variants: `Some(T)` (a value is present) and `None` (no value). There is no `null` sentinel — absence is always encoded in the type. The compiler enforces exhaustive matching: pattern matching on `Option` must cover both variants.

```orthon
user = db.find_user(42)     # returns Option<User>
name = user?.name ?? "Guest" # chaining + fallback
raw = user!                  # forced unwrap (panics if None)
```

Key operators: `?.` (elvis / optional chaining), `??` (unwrap or default), `!` (forced unwrap).

- **Source:** `../what/concepts/NULL_SAFETY.md`
- **See also:** [Representation](#representation), [Trait](#trait), [Orphan Rule](#orphan-rule)

### Orphan Rule

The coherence rule in Orthon's trait system: an implementation of a trait for a type must be defined in the same compilation unit as either the trait or the type. This prevents conflicting implementations of the same trait for the same type across different modules.

```orthon
// Allowed: trait defined here
impl Printable for User { ... }

// ERROR: orphan — neither trait nor type defined here
// impl ForeignTrait for ForeignType { ... }
```

- **Source:** `../what/concepts/TRAITS.md` § Coherence: The Orphan Rule
- **See also:** [Trait](#trait), [Trait Bound](#trait-bound)

### Orthogonality

Each language construct solves exactly one problem and combines freely with other constructs. No special cases, no context-dependent syntax, no conflicting rules. What you learn in one part of the language transfers directly to every other part.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Orthogonality, `../why/VISION.md` § Comfortable by Design, `../why/ZEN.md`, `../why/MANIFESTO.md`
- **See also:** [Minimal Core](#minimal-core), [Uniformity](#uniformity)

---

## P

### Parse, Don't Validate

An idiom for replacing boolean validation with parsing into a more
precise type, so the type itself becomes the witness of validity (the
antidote to Boolean Blindness). Instead of `isValidEmail(s) -> Bool`,
write `parseEmail(s) -> Option<Email>`: the compiler then enforces
handling of both outcomes (`match` on `Some`/`None`) and downstream code
receives an `Email` that is valid by construction. Realized through
ADTs, `Option`/`Result`, exhaustive `match`, and Constrained Types
(EDR-080).

- **Source:** `../notes/parse-dont-validate-idiom.md`, [`CONSTRAINED_TYPES.md`](../what/concepts/CONSTRAINED_TYPES.md)
- **See also:** [Boolean Blindness](#boolean-blindness), [Constrained Type](#constrained-type), [Option Type](#option-type)

### Policy

See [Implementation Policy](#implementation-policy).

### Persistent Data Structure

An immutable collection that shares internal structure across "modified"
versions, making snapshots cheap (O(log n) or O(1) amortised per
operation). Orthon's `Immutable` marker trait is accepted in v0.1 as a
forward contract; concrete persistent collection types (`PersistentList`,
`PersistentMap`, `PersistentSet`) are deferred to v0.2. v0.1 uses Tuple
+ Copy-on-Write for immutable data.

```orthon
trait Immutable    # marker trait — no methods, purely a guarantee
# v0.2: type PersistentList[T] is Immutable
```

- **Source:** `../what/concepts/PERSISTENT_DATA_STRUCTURES.md` (EDR-069)
- **See also:** [Immutable Marker Trait](#immutable-marker-trait), [Copy-on-Write](#copy-on-write-cow), [Value Semantics](#value-semantics)

### Policy

See [Implementation Policy](#implementation-policy).

### Product Type

A type that combines multiple values into a single compound value ("this
AND that"). In Orthon, product types are expressed through ADT variants
with multiple fields, standalone record types, or tuples. Product types
are the structural complement of sum types.

```orthon
# Product type as ADT variant with multiple fields
type Point(x: Int, y: Int)

# Product type as tuple (anonymous product)
(42, "hello")  # type: (Int, String)
```

Field access is positional or named depending on the declaration form.
Named fields are preferred for readability.

- **Source:** `../what/concepts/ALGEBRAIC_DATA_TYPES.md`, [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md)
- **See also:** [Algebraic Data Type](#algebraic-data-type), [Sum Type](#sum-type), [Tuple](#tuple)

### Program Enricher

The component that combines a [Program](#program) with its
[Execution Descriptor](#execution-descriptor) into a fully-defined
[Execution Program](#execution-program). The Enricher is infrastructure
— it resolves dependencies, selects the Implementation Strategy, and
produces the canonical Execution Program artifact consumed by all
Execution Engines.

The Enricher is not part of the language — it is a tooling concern
in the Execution Program pipeline. The Language specification defines
only the *shape* of the Execution Descriptor and the *interface* of
the Execution Program.

```
Program + Execution Descriptor
            │
            ▼
    Program Enricher
            │
            ▼
    Execution Program
```

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Program Enricher
- **See also:** [Execution Descriptor](#execution-descriptor), [Execution Engine](#execution-engine), [Execution Program](#execution-program)

---

## R

### Refinement Type

A type carrying a value-range or predicate constraint, e.g.
`type Port = Int requires v in 1..65535`. Orthon's accepted pragmatic
form is **Constrained Types** (EDR-080): a nominal type with a
runtime-enforced predicate at entry boundaries (`Age(200)` is a
compile-time warning plus runtime error). The open hypothesis
([`REFINEMENT_TYPES.md`](../how/concepts/research/deferrable/REFINEMENT_TYPES.md))
is **static refinement** — compile-time rejection of invalid literals
for a decidable predicate subset (ranges, positivity, non-empty),
moving Tier-2 enforcement to Tier-1 proof without an SMT solver.

- **Source:** `../how/concepts/research/deferrable/REFINEMENT_TYPES.md`, [`CONSTRAINED_TYPES.md`](../what/concepts/CONSTRAINED_TYPES.md) (EDR-080)
- **See also:** [Constrained Type](#constrained-type), [Invariant Classification](#invariant-classification)

### Primitive Block

An irreducible atomic language operation from which all constructs are
composed. Each primitive serves one or more Semantic Dimensions (Identity,
Ownership, Mutation, Evaluation, Visibility, Lifetime) and must be
orthogonal to all other primitives (no overlap in responsibility). The
complete set is defined in `../what/PRIMITIVE_BLOCKS.md`.

- **Source:** `../what/PRIMITIVE_BLOCKS.md`
- **See also:** [Primitive Operation](#primitive-operation), [Composition (of primitives)](#composition-of-primitives), [Semantic Dimension](#semantic-dimension)

### Primitive Operation

An atomic language operation in the Semantic Dependency Architecture
(Level 1) that cannot be decomposed into simpler operations. Primitive
Operations define the minimal set of actions on data. Examples:
variables, functions, call `()`, pack/unpack `*`, attribute access `.`,
metadata access `@`, condition, loop, closure, exceptions.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Dependency Flow](#dependency-flow), [Language Pattern](#language-pattern), [Semantic Layer](#semantic-layer)

### Policy

A declarative constraint or preference within a single domain-specific
area of implementation (allocation, algorithm selection, evaluation
mode, lifetime management, concurrency, etc.). Each Policy makes
decisions independently in its area of responsibility. Policies are
not part of the language — they are implementation choices assembled
into named Strategies.

- **Source:** `../how/IMPLEMENTATION_POLICIES.md`
- **See also:** [Implementation Strategy](#implementation-strategy), [Architecture](#architecture)

### Principle of Least Astonishment

The behaviour of a construct must match what a competent programmer intuitively expects. Surprises belong in discovery, not in semantics.

- **Source:** `../why/VISION.md` § Comfortable by Design
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Semantics](#explicit-semantics)

### Program Enricher

The component that combines a Program with its Execution Descriptor
into a fully-defined Execution Program. Internally coordinates
dependency, runtime, strategy, platform, permission, and resource
resolvers. Externally it is a single step — the boundary between
"incomplete" and "fully-defined" program.

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Program Enricher
- **See also:** [Execution Descriptor](#execution-descriptor), [Execution Program](#execution-program)

---

## R

### Reference

A handle to data that provides indirection without implying ownership. Two
forms: shared read-only reference and exclusive mutable reference. Reference
is a Primitive Operation (Level 1) — it underlies class identity, borrowing,
and delegate semantics.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity, `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Primitive Operation](#primitive-operation), [Representation](#representation), [Binding Identity](#binding-identity)

### Representation

A specific view of data. Orthon provides several fundamental representations:

- **Value** — A single, immutable value.
- **Tuple** — A fixed-size ordered collection.
- **Reference** — A handle to data (`&x` / `ref&`).
- **Sequence** — Values produced over time (see [Sequence](#sequence)).
- **Set** — An unordered collection of unique values.
- **Option** — A value that may be absent.
- **Result** — A value that may be an error.

- **Source:** `how/concepts/research/DATA_MODEL.md` § Model (What)
- **See also:** [Data](#data), [Data Modifier](#data-modifier)

### Representation Symmetry

Reversible transformations use symmetric syntax. Prefix form creates an alternative representation; postfix form restores the original.

```R

### Region-Based Memory

A sub-policy within the Allocation Policy ([EDR-034](../how/decision_records/architecture/EDR-034-allocation.md))
that refines how Arena allocation manages memory regions. Effective
when Allocation Policy is set to `Arena`. Defines three values:
`ScopeRegion` (arena lifetime inferred from lexical scope),
`ExplicitRegion` (programmer-declared arena annotations), and
`NoRegion` (no region-based allocation).

```text
DEFAULT_STRATEGY: Region = ScopeRegion
```

Classified as Policy per D-04 — region-based allocation is an
implementation choice about how Arena allocation materialises memory,
not Language semantics.

- **Source:** `../how/IMPLEMENTATION_POLICIES.md` § Region-Based Memory Sub-Policy, [EDR-035](../how/decision_records/architecture/EDR-035-region-based-memory-management.md)
- **See also:** [Allocation Policy](#allocation-policy), [Implementation Strategy](#implementation-strategy)

### Representation Modifier

An orthogonal annotation on a primitive that controls how a type is
stored in memory without changing its semantic identity. Representation
modifiers (`struct(T)`, `boxed(T)`, `shared(T)`, `atomic(T)`, `ffi(T)`,
`packed(T)`) are annotations on the `pack` and `reference` primitives,
not new primitives themselves ([EDR-038](../how/decision_records/architecture/EDR-038-representation-modifiers.md)).

Key property: `struct(User)` and `boxed(User)` are the *same semantic
type* — they differ only in storage strategy. Assignment between
representations is safe.

- **Source:** `../what/PRIMITIVE_BLOCKS.md` § 7b. Representation Modifiers, [EDR-038](../how/decision_records/architecture/EDR-038-representation-modifiers.md)
- **See also:** [Primitive Block](#primitive-block), [Data Primitive](#data-primitive)

---

## 
&value   → Reference
ref&     → Value

*values  → Pack
pack*    → Values
```

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Representation Symmetry
- **See also:** [Explicit Semantics](#explicit-semantics), [Named Before Symbolic](#named-before-symbolic)

### Result Type

The canonical representation of fallible operations in Orthon. A sum
type with two variants: `Ok(T)` (success, containing a value of type
`T`) and `Error(E)` (failure, containing an error value of type `E`).
Functions that can fail declare `Result<T, E>` as their return type.
The compiler enforces exhaustive matching on both variants.

```orthon
fun divide(a: Int, b: Int) -> Result<Int, DivisionError>
    if b == 0 then Error(DivisionError.DivisionByZero)
    else Ok(a / b)
```

Key operator: `?` for short-circuit propagation. Combinators: `map`,
`and_then`, `or_else`, `unwrap`, `unwrap_or`, `unwrap_or_else`.
Distinct from `Option<T>` — `Result` carries diagnostic error
information, while `Option` represents mere absence.

- **Source:** `../what/concepts/ERROR_HANDLING.md`
- **See also:** [Error Propagation](#error-propagation), [Option Type](#option-type)

---

## S

### Semantic Dimension

One of six independent, orthogonal facets a Semantic Model uses to
fully characterize what an Orthon program means: **Identity** (what
does "the same" mean), **Ownership** (who is accountable for a value),
**Mutation** (how and when values change), **Evaluation** (when
expressions are evaluated), **Visibility** (what is reachable from
where), and **Lifetime** (how long a value lives). Each dimension
answers exactly one question, is defined independent of syntax and
Implementation Strategy, and composes orthogonally with the other five
— see `SEMANTIC_MODEL.md`'s Cross-Dimension Consistency section for all
fifteen pairwise interactions.

- **Source:** `../what/SEMANTIC_MODEL.md` § Semantic Dimensions
- **See also:** [Semantic Invariant](#semantic-invariant), [Core Language](#core-language), [Orthogonality](#orthogonality)

### Semantic Equality

User-defined equality via the `==` operator, implemented through the `Eq` trait. Falls back to structural value equality (`===`) if not overridden. Enables domain-specific equivalence (e.g., two `Person` objects with the same ID are equal even if other fields differ).

```orthon
impl Eq for Person
    fn ==(self, other: Person) -> Bool
        self.id === other.id
```

Distinct from [Value Equality](#value-equality) (`===`, structural) and [Identity Equality](#identity-equality) (`is`, reference identity).

- **Source:** `../what/concepts/EQUALITY.md` § Model
- **See also:** [Value Equality](#value-equality), [Identity Equality](#identity-equality), [Structural Equality](#structural-equality)

### Semantic Invariant

One of six cross-cutting rules that hold across all six Semantic
Dimensions at once, rather than belonging to any single dimension —
for example, "every value has exactly one owner at any point in the
program" or "mutation requires exclusive access; read access may be
shared." Semantic Invariants are the semantic floor every
Implementation Strategy must guarantee; a Strategy may choose its own
enforcement mechanism but may never relax an invariant.

- **Source:** `../what/SEMANTIC_MODEL.md` § Semantic Invariants
- **See also:** [Semantic Dimension](#semantic-dimension), [Implementation Strategy](#implementation-strategy)

### Semantic Layer

A level in the Semantic Dependency Architecture hierarchy. Each
Semantic Layer has a defined responsibility, a set of allowed
constructs, and a dependency rule: it may only reference constructs
from lower layers (never from higher layers). The six layers are:
Data Model (0), Primitive Operations (1), Language Patterns (2),
Standard Library (3), Frameworks (4), Applications (5).

- **Source:** `../how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture
- **See also:** [Dependency Flow](#dependency-flow), [Language Pattern](#language-pattern), [Primitive Operation](#primitive-operation)

### Semantics Before Optimization

Language semantics are independent of optimization. A correct Orthon program must produce the same observable behavior regardless of optimization level.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Semantics Before Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Optimization](#explicit-optimization)

### Sequence

A fundamental type representing a sequence of values produced over time. Unlike traditional generators or streams, a Sequence describes *what the result is*, not *how it is produced*. It is a normal object: it can be returned, stored, passed, transformed, or consumed incrementally.

- **Source:** `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` § Sequence and the `emit` Keyword
- **See also:** [Representation](#representation)

### Stream

A push-based observable type (`Stream<T>`) that emits values to
subscribed consumers asynchronously. Streams are the dual of pull-based
sequences: in a pull model the consumer calls `next()`, while in a push
model the producer calls `emit()` and the consumer reacts via a
subscription callback. Streams build on the delegate model (EDR-033)
and channels (EDR-049) for async delivery.

```orthon
let stream = Stream<Int>.create()
let sub = stream.subscribe(fn (v) print(v))
stream.emit(1)
stream.emit(2)
stream.complete()
```

- **Source:** `../what/concepts/PUSH_STREAMS.md` (EDR-051)
- **See also:** [Sequence](#sequence), [Channel](#channel), [Delegate](#delegate)

### Spawn

A keyword that creates a new concurrent task running in parallel with
the current one. `spawn` makes parallelism syntactically visible —
without it, `async` functions execute sequentially in the current
context.

```orthon
let t1 = spawn async loadImage("a.jpg")
let t2 = spawn async loadImage("b.jpg")
let img1 = await t1
let img2 = await t2
```

`spawn` returns `Task<T>` (which is also a `Future<T>`). Tasks can be
cancelled via `.cancel()`. Tasks within a `scope` block are
automatically managed.

- **Source:** `../what/concepts/ASYNC_AWAIT.md` (EDR-047)
- **See also:** [Async](#async), [Scope (Structured Concurrency)](#scope-structured-concurrency), [Future](#future)

### Stable Mental Model

Programmers should reason about language semantics, not compiler internals. Users should never need to understand implementation details to predict program behaviour.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Stable Mental Model
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Semantics](#explicit-semantics)

### Standard Library

The layer that defines public abstractions exposed by the language. It specifies *behaviour*, not *implementation*. The Standard Library is the interface between user code and the Implementation Strategy.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Standard Library
- **See also:** [Architecture](#architecture), [Core Language](#core-language), [Implementation Strategy](#implementation-strategy)

### Structural Equality

The `==` comparison semantics for value-typed data: two values are
equal exactly when their structure (fields/contents) matches,
regardless of storage location. The default, and only, meaning of `==`
for plain (non-reference) types in Orthon — it is never overloaded to
mean identity comparison, and never silently answers the Binding
Identity question ("are these the same storage").

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity
- **See also:** [Value Identity](#value-identity), [Data](#data)

### Structural Typing

A mode of trait satisfaction where a type satisfies a trait automatically
if its method signatures match the trait's declaration — no explicit
`impl Trait for Type` block required. In Orthon, structural typing is
opt-in via the `structural` keyword on a trait declaration. Nominal
(explicit `impl`) is the default.

```orthon
structural trait Show
    fn show(self) -> String
```

Explicit `impl` blocks always take priority over structural matching.
Ambiguity conflicts (a type matching two structural traits with
conflicting signatures) produce a compile-time error.

- **Source:** `../what/concepts/STRUCTURAL_TYPING.md`, [EDR-044](../how/decision_records/architecture/EDR-044-structural-typing.md)
- **See also:** [Trait](#trait), [Trait Bound](#trait-bound), [Derive](#derive)

### Sum Type

A type whose values are exactly one of a fixed set of named variants
("this OR that"). In Orthon, sum types are expressed through Algebraic
Data Types (EDR-039). A sum type's variants are enumerated in the ADT
declaration; the compiler enforces exhaustive matching via pattern
matching (EDR-025).

```orthon
# Sum type — exactly one of Circle, Rectangle, or Triangle
type Shape = Circle(radius: Float)
           | Rectangle(width: Float, height: Float)
           | Triangle(a: Float, b: Float, c: Float)
```

Orthon has exactly one sum-type mechanism (ADTs). No separate enum,
named constant, or iota construct.

- **Source:** `../what/concepts/ALGEBRAIC_DATA_TYPES.md`, [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md)
- **See also:** [Algebraic Data Type](#algebraic-data-type), [Product Type](#product-type), [Exhaustiveness](#exhaustiveness), [Tagged Union](#tagged-union)

---

## T

### Trait

A behavioural contract that types can implement. Traits define method signatures, associated types, and optionally provide default implementations. Traits support both static dispatch (via generics with `where T: Trait` bounds, monomorphised at compile time) and dynamic dispatch (via `dyn Trait`, vtable-based, opt-in).

```orthon
trait Printable
    fn format(self) -> String

impl Printable for User
    fn format(self) -> String
        return "User({self.name})"

fn print_all[T: Printable](items: [T])
    ...
```

Orthon's trait system is nominal (explicit `impl`), enforces coherence via the [Orphan Rule](#orphan-rule), supports [associated types](#associated-type) and default method implementations, and explicitly rejects [inheritance](#inheritance) in favour of composition via `where` clauses.

- **Source:** `../what/concepts/TRAITS.md`
- **See also:** [Trait Bound](#trait-bound), [Orphan Rule](#orphan-rule), [Declaration Kind (`fun` / `proc` / `new`)](#declaration-kind-fun--proc--new)

### Trait Bound

A constraint on a generic type parameter requiring that the type implements a given trait. Expressed via `where` clauses.

```orthon
fn sort[T](items: [T]) where T: Ordered + Printable
```

Multiple bounds are combined with `+`. Trait bounds enable static dispatch by default. The compiler verifies that all concrete types used in generic functions satisfy their required bounds.

- **Source:** `../what/concepts/TRAITS.md` § Model
- **See also:** [Trait](#trait), [Orphan Rule](#orphan-rule)

### Tagged Union

The default memory layout for Algebraic Data Types (ADTs): a discriminant
(tag) followed by the variant's fields. The tag identifies which variant
is active at runtime, enabling pattern matching to select the correct
arm. The compiler optimises layout by packing the tag into padding bytes
where possible (niche optimisation).

Tagged union layout is an Implementation Strategy concern — the
semantics of ADTs are defined independently of layout; tagged union is
the default strategy but flat layout (no tag when variants are
distinguishable by field types) and niche optimisation are permitted
alternatives.

- **Source:** `../what/concepts/ALGEBRAIC_DATA_TYPES.md` § Default Strategy, [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md)
- **See also:** [Algebraic Data Type](#algebraic-data-type), [Sum Type](#sum-type), [Pattern Matching](#pattern-matching)

### Type Inference

The compiler's ability to determine the type of an expression from its
context without explicit annotation. Orthon uses **local bidirectional
inference**: types are inferred within function bodies (bottom-up from
expressions, top-down from expected type), but explicit annotations
are required at public API boundaries (parameters and return types).

```orthon
fn process(items: [Int]) -> Int
    let doubled = items.map(fn (x) -> x * 2)   # all inferred
    return doubled.sum()
```

Generic type arguments are inferred at call sites. Turbofish `::<T>`
disambiguates ambiguous cases. No cross-module inference — module
consumers never need to run the inference engine. Inferred types
are inspectable via the Schema Provider.

- **Source:** `../what/concepts/TYPE_INFERENCE.md` (EDR-027)
- **See also:** [Generics](#generics), [Trait Bound](#trait-bound), [Schema Provider](#schema-provider)

### Type-Level Computation

The ability to derive new types from existing types through built-in
compiler intrinsics. Orthon provides a closed set of 8 non-recursive
intrinsics:

| Intrinsic | Semantics |
|-----------|----------|
| `KeyOf<T>` | Union of literal property-name types of `T` |
| `Pick<T, K>` | Type with only keys `K` from `T` |
| `Omit<T, K>` | Type with all keys except `K` from `T` |
| `Partial<T>` | All keys of `T` become optional |
| `Required<T>` | All keys of `T` become required |
| `Record<K, V>` | Type with keys `K` and values `V` |
| `Readonly<T>` | All keys read-only |
| `ElementOf<T>` | Element type of a collection type |

NO user-extensible type-level language. NO recursion. NO `infer`.
Intrinsics are composable (e.g., `Partial<Omit<User, "password">>`).
For custom type-level operations beyond the intrinsic set, use the
derive/macro mechanism (EDR-029).

- **Source:** `../what/concepts/TYPE_LEVEL_COMPUTATION.md`, [EDR-046](../how/decision_records/architecture/EDR-046-type-level-computation.md)
- **See also:** [Literal Type](#literal-type), [Derive](#derive), [Comptime](#comptime)

### Type-Level Null Safety

The extension of Orthon's null safety model (EDR-018) with
flow-sensitive type narrowing. `Option<T>` and `T` are distinct,
incompatible types. After a pattern match on `Some(x)`, the compiler
narrows the value's type to `T` in the matching arm. After an explicit
`if value != None` check, the compiler narrows within the true branch.

```orthon
match opt:
    case Some(x) => process(x)    # x is T — narrowed
    case None => handle_empty()
```

Narrowing is per-variable and flow-sensitive: it follows control flow
and resets on variable reassignment. Conservative by default — if the
compiler cannot prove a value is non-null, it remains `Option<T>`. The
`!` operator is the explicit escape hatch.

- **Source:** `../what/concepts/TYPE_LEVEL_NULL_SAFETY.md` (EDR-028)
- **See also:** [Flow-Sensitive Narrowing](#flow-sensitive-narrowing), [Option Type](#option-type), [Pattern Matching](#pattern-matching)

---

## V

### Validation Gate

One of seven independent perspectives used in [Decision Validation](#decision-validation)
to assess a language design proposal. Each gate has its own criteria,
pass/fail conditions, and scope of examination.

- **Source:** `../how/gates/DECISION_VALIDATION.md`
- **See also:** [Decision Validation](#decision-validation), [Language Design Gate](#language-design-gate)

### Verification Layer

One of seven ordered stages of static analysis in the compiler pipeline. Each layer depends on the layers before it:

1. **Syntax & Parsing** — well-formed tokens, valid grammar
2. **Name Resolution** — symbol resolution, duplicate detection, visibility
3. **Type Checking** — type validity, argument matching, trait bounds
4. **Ownership & Borrowing** — ownership, borrowing, lifetime correctness
5. **Effect Verification** — mutation, allocation, I/O, async boundaries
6. **Exhaustiveness & Completeness** — pattern match coverage, return paths
7. **Contract Verification** (optional) — pre/post conditions, loop invariants

Layers 1–6 are always enabled. `--relaxed` mode skips layers 6–7 for prototyping.

- **Source:** `../what/concepts/COMPILER_AS_STATIC_ANALYZER.md`, EDR-030
- **See also:** [Decision Validation](#decision-validation), [Language Design Gate](#language-design-gate)

### Value Identity

Whether two values are considered the same persistent entity across
time, independent of their current structural content. Meaningful only
for explicit, opt-in shared/reference types — a plain (structural)
value has no Value Identity beyond its structure. Distinct from
Binding Identity; Structural Equality (`==`) always answers the
value-identity question in value-semantics terms, never the
binding-identity question.

- **Source:** `../what/SEMANTIC_MODEL.md` § Identity
- **See also:** [Binding Identity](#binding-identity), [Structural Equality](#structural-equality)

### Value Equality

Structural comparison via the `===` operator: two values are equal if their data content is structurally equivalent (field-by-field, recursively). This is the default comparison for all data in Orthon. Falls back to the `Eq` trait's `==` if the trait is implemented for the type.

```orthon
a === b   # true iff a and b are structurally equal
```

Distinct from [Semantic Equality](#semantic-equality) (`==`, user-defined) and [Identity Equality](#identity-equality) (`is`, reference identity). Related to [Structural Equality](#structural-equality) which is the specific semantics of `==` for plain (non-reference) types.

- **Source:** `../what/concepts/EQUALITY.md` § Model
- **See also:** [Semantic Equality](#semantic-equality), [Identity Equality](#identity-equality), [Structural Equality](#structural-equality)

---

## U

### Union Type

A structural, untagged type combinator that forms an anonymous union of
two or more types (`A | B`). Union types require no prior declaration and
no discriminant — the runtime representation is the value itself.

```orthon
type ID = String | Int
fun print_id(id: String | Int)
    match id:
        case s: String => print(s)
        case i: Int    => print(i.to_string())
```

Union members must be named types or literal types (EDR-043) — no
anonymous structural shapes in v0.1. Narrowing follows the same
flow-sensitive rules as TYPE_LEVEL_NULL_SAFETY (EDR-028). There is no
exhaustiveness guarantee (unlike ADTs).

- **Source:** `../what/concepts/UNION_INTERSECTION_TYPES.md`, [EDR-045](../how/decision_records/architecture/EDR-045-union-intersection-types.md)
- **See also:** [Literal Type](#literal-type), [Algebraic Data Type](#algebraic-data-type), [Intersection Type](#intersection-type), [Narrowing](#narrowing)

### Uniformity

Equivalent concepts should be expressed in equivalent ways. Once a user learns a language pattern, the same pattern applies consistently throughout the language.

- **Source:** `../how/DESIGN_PRINCIPLES.md` § Uniformity
- **See also:** [Orthogonality](#orthogonality)

---

## W

### WASM (WebAssembly)

A binary instruction format for stack-based virtual machines, designed
as a portable compilation target. In Orthon, WASM is one of several
execution targets — alongside interpreters, AOT compilers, and OCI
builders — that consumes the same Execution Program artifact without
modification, following the Execution Program / Execution Engine
separation.

- **Source:** `../how/concepts/research/EXECUTION_PROGRAM.md` § Execution Engine, `../how/architecture/ARCHITECTURE.md` § Execution Program Pipeline
- **See also:** [Execution Engine](#execution-engine), [Execution Program](#execution-program)

### Widening

The process by which a literal type widens to its base type. In Orthon,
the widening rule is a single, always-applicable rule: **immutable
bindings preserve literal types; mutable bindings widen to base types.**

```orthon
let x = "GET"    # type: "GET" (literal preserved)
var y = "GET"    # type: String (widened)
```

This rule is simpler than TypeScript's context-dependent widening
(`let` widens, `const`/`as const` preserves). The single rule is
LLM-generable and eliminates hidden conversions.

- **Source:** `../what/concepts/LITERAL_TYPES.md`, [EDR-043](../how/decision_records/architecture/EDR-043-literal-types.md)
- **See also:** [Literal Type](#literal-type), [Union Type](#union-type)

---

## C (continued)

### Contract (Design by Contract)

A verifiable assertion embedded in a function signature that specifies preconditions (`requires`), postconditions (`ensures`), and type/module invariants (`invariant`). Contracts are part of Orthon's function declaration syntax — they are not comments or library annotations.

```orthon
fun sqrt(x: Float) -> Float
    requires x >= 0.0
    ensures result * result ≈ x
```

Contract expressions are **pure** (no side effects, enforced by compiler). Contracts are checked at compile time where possible; otherwise at runtime in debug builds. Release builds elide contracts unless `--enforce-contracts` is passed. Contract inheritance follows Liskov substitution: subtypes may weaken preconditions and strengthen postconditions.

- **Source:** `../what/concepts/CONTRACTS.md` (EDR-056)
- **See also:** [Function Contract](#function-contract), [Verification Layer](#verification-layer)

---

## D (continued)

### Delegation (StdLib)

A composition pattern where a type implements a trait by forwarding method calls to a contained instance, or where a property delegates getter/setter behaviour to a helper object. Orthon provides delegation via the `@delegate` macro (EDR-029) and StdLib delegate protocols (`lazy`, `observable`, `vetoable`, `map`).

```orthon
@delegate(List[T]) to inner
# Compiler generates forwarding for all List methods
```

**Note:** DELEGATION (this concept) is distinct from DELEGATE (the concurrency execution policy from EDR-036). The execution `delegate` creates concurrent execution contexts; delegation pattern composes types orthogonally.

- **Source:** `../what/concepts/DELEGATION.md` (EDR-057)
- **See also:** [Macro](#macro), [Trait](#trait), [Property](#property)

---

## E (continued)

### Extension Function

A function defined outside its receiver type but called with method-call syntax: `expr.method()`. Extension functions are resolved at compile time based on the **static type** of the receiver (static dispatch). They cannot access private members of the receiver type.

```orthon
# Definition
fun String.isEmail() -> Bool:
    contains("@")

# Callsite
email = user.email_address.isEmail()
```

Extension functions from other packages must be explicitly imported. Member functions always take precedence over extension functions of the same name.

- **Source:** `../what/concepts/EXTENSION_FUNCTIONS.md` (EDR-058)
- **See also:** [Trait](#trait), [Static Dispatch](#static-dispatch)

---

## G (continued)

### Gradual Typing

A type system where type annotations are optional — the programmer may start with unannotated code and add types incrementally. Orthon uses gradual typing with global inference: types are inferred for all expressions, explicit annotations are never required but always accepted.

```orthon
name = "Alice"       # type inferred
fun greet(person: Person) -> String:
    "Hello, {person.name}!"
```

Boundary checks are inserted at typed/untyped function interfaces. The compiler runs a global consistency pass as an optional lint. No separate declaration files — type information lives alongside code. Critical for LLM adoption.

- **Source:** `../what/concepts/GRADUAL_TYPING.md` (EDR-059)
- **See also:** [Type Inference](#type-inference), [Boundary Check](#boundary-check)

---

## N (continued)

### Narrowing

See [Flow-Sensitive Narrowing](#flow-sensitive-narrowing), [Smart Cast](#smart-cast).

---

## P (continued)

### Property

A named value on a type that unifies stored field access and computed access behind a uniform `.name` interface. Every field in a type declaration is implicitly a property with a getter and optional setter. Computed properties specify the getter body explicitly.

```orthon
struct Person:
    name: String                  # implicit getter, no setter
    var age: Int                  # implicit getter + setter
    is_adult: Bool                # computed property
        get: self.age >= 18
```

Callers use `.name` syntax uniformly — stored and computed properties are indistinguishable at the call site. Changing a stored property to a computed one never changes the call site.

- **Source:** `../what/concepts/PROPERTIES.md` (EDR-062)
- **See also:** [Attribute Access](#attribute-access), [Delegation](#delegation-stdlib)

---

## S (continued)

### Smart Cast

The compiler's ability to automatically narrow the type of a variable based on control flow — after a type check, the variable's type is refined within the relevant scope without requiring an explicit cast.

```orthon
if value is String:
    print(value.length)      # smart cast — no explicit cast needed
```

Smart cast applies after `is`/`isnt` checks, `when` branches, and short-circuit operators (`&&`, `||`). Only applies to effectively-immutable variables. Conservative — if the compiler cannot prove safety, the wider type is retained. Partially subsumed by PATTERN_MATCHING (EDR-025).

- **Source:** `../what/concepts/SMART_CAST.md` (EDR-060)
- **See also:** [Flow-Sensitive Narrowing](#flow-sensitive-narrowing), [Pattern Matching](#pattern-matching), [Option Type](#option-type)

---

## C (continued)

### Copy-on-Write (CoW)

A memory optimisation technique where assignment of a value shares the underlying data, and mutation triggers a copy only when the data is shared by multiple references. CoW is transparent to the programmer — they write value-semantics code.

```orthon
data = [1, 2, 3]
data2 = data              # shares buffer (no copy)
data2[0] = 99             # clone happens here if shared
```

CoW is the DEFAULT_STRATEGY's mechanism for implementing value semantics on standard collections. The `shared` keyword provides explicit reference semantics (RC-based sharing). CoW avoids the need for a borrow checker in common patterns.

- **Source:** `../what/concepts/COPY_ON_WRITE.md` (EDR-061)
- **See also:** [Value Semantics](#value-semantics), [Shared Keyword](#shared-keyword)

---

## Z

The four guiding aphorisms of the language:

> Every special case creates complexity.
> Orthogonality removes exceptions.
> Consistency defeats complexity.
> Simplicity is the result of orthogonality.

- **Source:** `../why/ZEN.md`
- **See also:** [Orthogonality](#orthogonality)

---

### Unpacking (Destructuring)

The syntactic extraction of values from compound data structures (tuples,
records) into individual bindings using pattern syntax. Follows the
`pack`/`unpack` symmetry principle (PRIMITIVE_BLOCKS) — the same syntax
used to construct a value is used to decompose it.

```orthon
let (x, y) = point              # tuple destructuring
let {name, age} = person        # record destructuring
let {address: {city}} = user    # nested destructuring
```

All destructuring forms desugar to `pack`/`unpack` primitives — no new
runtime semantics. Supports rest patterns (`..rest`), ignore patterns
(`_`), rename syntax, function parameter destructuring, and `for` loop
destructuring.

- **Source:** `../what/concepts/UNPACKING.md` (EDR-055)
- **See also:** [Primitive Block](#primitive-block), [Pattern Matching](#pattern-matching), [Representation Symmetry](#representation-symmetry)

---

## W

### While Loop

The condition-based looping construct in Orthon: `while condition`.
Separate from the iteration construct (`for ... in`). Repeats the body
as long as the condition evaluates to `true`.

```orthon
while queue.not_empty():
    process(queue.dequeue())
```

`break` exits the loop early; `continue` skips to the next iteration.
No C-style `for (;;)` — use `while` with explicit condition or `loop`
for infinite loops.

- **Source:** `../what/concepts/ITERATION_LOOP.md` (EDR-053)
- **See also:** [For Loop](#for-loop), [Iterator Protocol](#iterator-protocol)

---

## Y

### Yield

A keyword in generators that produces a value and suspends execution.
`yield` without a consumer interaction is equivalent to `emit` (one-way
production). `yield expr` optionally receives a value from the consumer
(bidirectional yield), enabling interactive coroutine patterns.

```orthon
# One-way (equivalent to emit)
yield value

# Bidirectional — receives value from consumer
let response = yield value
```

Bidirectional `yield` requires the generator to implement
`BidirectionalGenerator[T, U]` with a `send(value: U)` method.

- **Source:** `../what/concepts/GENERATORS.md` (EDR-050)
- **See also:** [Generator](#generator), [Lazy Sequence](#lazy-sequence), [Intermediate Result](#intermediate-result)

---

### Hygienic Macro

A macro whose identifier bindings are scoped to the macro expansion site and cannot collide with identifiers in the calling scope. In Orthon, macros are hygienic **by default** — identifiers introduced by a macro expansion are invisible outside that expansion. Unhygienic access (intentional access to the caller's scope) uses an explicit `#` prefix.

```orthon
@macro
fun counter_generator() -> Expr
    return `#counter += 1`  # unhygienic — accesses caller's `counter`
```

Hygiene-by-default prevents accidental identifier collisions, a common source of bugs in non-hygienic macro systems (C preprocessor, early Lisp macros).

- **Source:** `../what/concepts/AST_MACROS.md` § Hygiene
- **See also:** [Macro](#macro), [Derive](#derive), [Comptime](#comptime)

## How to Maintain This Glossary

### Trigger Conditions

A new glossary entry is required when:
- A new concept document is created in `docs/how/concepts/research/`
- An existing concept introduces a domain-specific term not already in GLOSSARY.md
- An EDR introduces new terminology or redefines an existing term

### Process

1. **Identify** — when creating or modifying a concept document, identify any new terms that are not yet in GLOSSARY.md
2. **Add** — insert the term in the correct alphabetical section with a definition, source link, and "See also" links to related terms
3. **Cross-reference** — add a link from the source document's Affected Documents checklist to GLOSSARY.md
4. **Verify** — ensure no term has conflicting definitions across documents (a term must mean the same thing everywhere)

### Review Cadence

At every **phase boundary** (before starting a new phase), verify that all new terms introduced during the completed phase are registered in GLOSSARY.md. This is part of the phase completion checklist.

### Consistency Check

If the same term appears in multiple documents, verify that all definitions are consistent. Inconsistent definitions must be reconciled before the phase is considered complete. The Glossary is the source of truth — if a source document disagrees, the source document must be corrected.

### Rules

1. **Add terms proactively.** When a new concept is introduced in any design document, add it here.
2. **Keep definitions in sync.** If a term's meaning evolves, update this file and all source documents simultaneously.
3. **Prefer cross-references over duplication.** The Glossary defines *what* a term means; source documents explain *why and how* it matters.
4. **Alphabetical order.** Each section uses the term's first letter as a heading anchor.
