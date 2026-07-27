# Primitive Blocks

> **✅ ACCEPTED — EDR-016 (Primitive Blocks).**
> This document defines the minimal orthogonal set of primitive building
> blocks from which all language constructs are composed. It is the
> Level 1 (Primitive Operations) layer of the Semantic Dependency
> Architecture — the irreducible atomic operations serving the six
> Semantic Dimensions (Identity, Ownership, Mutation, Evaluation,
> Visibility, Lifetime).
>
> **Status:** Accepted (Phase 3 of M1).
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 3,
> [`SEMANTIC_MODEL.md`](SEMANTIC_MODEL.md),
> [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md),
> [`FOUNDATIONAL_ABSTRACTIONS.md`](../how/concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md),
> [`GLOSSARY.md`](GLOSSARY.md)

---

## 1. Purpose & Scope

This document replaces the DRAFT placeholder with the complete, validated
specification of Orthon's minimal orthogonal primitive set. It serves as
the **Level 1 (Primitive Operations)** layer of the Semantic Dependency
Architecture ([`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md) § Semantic Dependency Architecture).

**Purpose:** Provide the irreducible foundation for Phase 4 (Derived Features).
A concept that cannot decompose onto this set signals an incomplete primitive
set; a primitive that overlaps another signals a non-orthogonal set. Every
derived feature in Phase 4 must decompose into these primitives.

**Dependency:** This set is justified against the six Semantic Dimensions
defined in [`SEMANTIC_MODEL.md`](SEMANTIC_MODEL.md) (EDR-013). Each primitive
serves one or more of those dimensions. The Semantic Model is the authority
on *what programs mean*; the Primitive Blocks are *how the language makes
that meaning operational*.

**Deliverable for Phase 4:** If a Phase 4 derived feature cannot be
decomposed onto this set, the primitive set must be revisited before Phase 4
proceeds. This document is the gate between Phase 3 and Phase 4.

---

## 2. Organizing Taxonomy

Per D-02 (see [`03-CONTEXT.md`](../.planning/phases/03-primitive-blocks-identify-the-minimal-orthogonal-set-of-prim/03-CONTEXT.md) § D-02),
the **Data / Data Modifiers** hypothesis from
[`FOUNDATIONAL_ABSTRACTIONS.md`](../how/concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md)
serves as the *conceptual organizing framework* for the primitive set, but
NOT as the primitive set itself. Primitives are categorized into:

| Category | Responsibility | Primitives |
|----------|---------------|------------|
| **Data Primitives** | Producing or structuring data values | `literal`, `identifier`, `pack/unpack` |
| **Data Operations Primitives** | Transforming, accessing, or controlling data | `assignment`, `function`, `call`, `attribute access`, `scope`, `reference` |

The taxonomy is a classification aid, not a replacement for the primitive set.
Two abstractions alone would be too coarse-grained for Phase 4 decomposition
verification — a `for` loop and a function call would both be "Data Modifiers,"
losing the granularity needed to confirm orthogonality.

---

## 3. Primitive Specifications

Each primitive is defined with:
1. **Name and one-sentence definition.**
2. **Category** — Data or Data Operations.
3. **Semantic dimensions served** — which of the six SEMANTIC_MODEL.md
   dimensions this primitive addresses, with explanation.
4. **Orthogonality statement** — what it does NOT overlap with.
5. **Composition rules** — how it combines with other primitives.
6. **Examples** in abstract Orthon syntax.

### 3.1 Data Primitives

#### 3.1.1 `literal`

| Field | Value |
|-------|-------|
| **Category** | Data Primitive |
| **Definition** | Inline value notation. Produces a Data value from source text without involving a binding or computation. |
| **Semantic dimensions** | **Data** — directly creates data from source text. No other dimension is engaged because a literal has no binding (Identity), no owner beyond its immediate expression (Ownership), is immutable by construction (Mutation), is evaluated at the point of occurrence (Evaluation), has no visibility modifier (Visibility), and lives until the enclosing expression completes (Lifetime). |
| **Orthogonal to** | `identifier` (literal creates a value from text; identifier names a value that already exists). `assignment` (literal does not create a binding — it is a value that can be bound, but the binding is a separate operation). |
| **Composition** | Literals are consumed by `assignment` (to bind), by `call` (as arguments), by `pack` (as members of composites), and by `attribute access` (as default values). |

```orthon
42              # integer literal
"hello"          # string literal
true             # boolean literal
(1, 2, 3)        # tuple literal
{ "key": val }   # mapping literal
```

#### 3.1.2 `identifier`

| Field | Value |
|-------|-------|
| **Category** | Data Primitive |
| **Definition** | A named reference to a value. Binds a name to a storage location holding a value. |
| **Semantic dimensions** | **Identity** — the name is the binding point that gives a value durability beyond a temporary expression result. **Ownership** — the named binding is the owner of the value; ownership follows the name. |
| **Orthogonal to** | `literal` (identifier names existing values; literal creates from scratch). `scope` (identifiers exist within scopes but are not the scope itself — a scope containing no identifiers is still a scope). |
| **Composition** | Every named construct uses identifier: function names, parameters, variables, type names, module names. Identifiers are resolved within a `scope`. Their values are established by `assignment`. |

```orthon
let x = 42       # identifier 'x' bound to value 42
fun add(a, b)    # identifiers 'add', 'a', 'b'
    return a + b
```

#### 3.1.3 `pack` / `unpack` (symmetric pair)

| Field | Value |
|-------|-------|
| **Category** | Data Primitive |
| **Definition** | The symmetric composition/decomposition pair. `pack` combines values into a composite structure; `unpack` decomposes a composite into its constituent values. These are one primitive with two operations, sharing a single syntax (the `*` operator per Semantic Purity). |
| **Semantic dimensions** | **Data** — construction (pack) and destruction (unpack) are the two essential operations on composite data. **Representation** — the composite's structure is defined by how pack arranges its constituents and how unpack recovers them. |
| **Orthogonal to** | `literal` (pack combines existing values; literal creates from scratch). `reference` (pack is structural composition without indirection — a packed value contains its constituents directly; reference is indirection to a value elsewhere). |
| **Composition** | `pack` underlies struct, tuple, and record construction. `unpack` underlies destructuring assignment and pattern matching. Per the symmetry principle ([`UNPACKING.md`](../how/concepts/research/important/UNPACKING.md)), construction and destruction follow the same syntax. |

```orthon
*point = pack(x, y)          # pack — combine into composite
let * = point                # unpack — decompose (destructuring)
(x, y) = *point              # unpack — positional destructuring
```

### 3.2 Data Operations Primitives

#### 3.2.1 `assignment`

| Field | Value |
|-------|-------|
| **Category** | Data Operations Primitive |
| **Definition** | Bind a value to an identifier. Creates or updates a binding between a name and a storage location holding a value. |
| **Semantic dimensions** | **Evaluation** — stores a value for later use, making it available to subsequent expressions. **Ownership** — establishes ownership of the value at the binding point; the binding becomes the owner until ownership is transferred (moved) or the scope exits. |
| **Orthogonal to** | `identifier` (identifier names the slot; assignment fills it). `scope` (assignment changes what a name denotes within its scope; scope defines where that change is visible — the two are independent operations). |
| **Composition** | `let`/`var` declarations, parameter binding, field assignment, and destructuring assignment all decompose to assignment. Assignment consumes `literal` values or the results of `call`/`pack` expressions. |

```orthon
let x = 42        # immutable assignment
var count = 0     # mutable assignment
count = count + 1 # reassignment (update)
```

#### 3.2.2 `function`

| Field | Value |
|-------|-------|
| **Category** | Data Operations Primitive |
| **Definition** | Parameterized computation declaration. Defines a reusable computation with explicit parameters, an optional return type, and a body enclosed in a `scope`. |
| **Semantic dimensions** | **Evaluation** — declares a computation boundary; the function's body is evaluated only when the function is called, not when it is defined. |
| **Orthogonal to** | `call` (declaration vs. invocation is the fundamental split — function defines *what*; call triggers *how*). |
| **Composition rules** | The three declaration kinds (`fun`, `proc`, `new`) are **tags on the function primitive**, not separate primitives — they modify how the function interacts with its context (pure, mutating, transforming). Per D-04, the function-call split is clean: function addresses *what* (construct definition), call addresses *how* (evaluation trigger). Function bodies are `scope` + `assignment` + other primitives. Closures decompose to `function` + `identifier` + `scope` (captured environment). |

```orthon
fun add(a, b) -> Int        # pure function
    return a + b

proc append(item: Int)      # mutating procedure
    self.items.push(item)

new sorted() -> List        # transforming constructor
    List(self.items.sorted())
```

#### 3.2.3 `call`

| Field | Value |
|-------|-------|
| **Category** | Data Operations Primitive |
| **Definition** | Invocation of a declared function. Triggers evaluation of a function body with supplied arguments. Unified syntax regardless of declaration form (named, anonymous, closure). |
| **Semantic dimensions** | **Evaluation** — triggers computation; the function's body is evaluated with the given arguments. **Lifetime** — function scope begins at call and ends at return; the call frame's lifetime is bounded by the call. |
| **Orthogonal to** | `function` (declaration vs. invocation). `assignment` (call triggers computation that may produce a value; assignment binds that value — the two compose sequentially). |
| **Composition** | Every function execution is a call. Recursion is call composed with itself. `()` is the call syntax per Semantic Purity. Call consumes literals, identifiers, and composite values as arguments. |

```orthon
add(1, 2)               # named function call
list.append(item)       # method call (attribute access + call)
func_ptr(args)          # closure/callable call
```

#### 3.2.4 `attribute access`

| Field | Value |
|-------|-------|
| **Category** | Data Operations Primitive |
| **Definition** | Access a member of a composite value. Dereferences a named field or method on a composite using `.` syntax. |
| **Semantic dimensions** | **Visibility** — selects which part of a composite to expose; the accessible members are determined by the type's visibility rules. |
| **Orthogonal to** | `reference` (attribute access selects a named member within a value; reference points to a value without selecting a member). `pack`/`unpack` (attribute access extracts a single named member; unpack extracts multiple members by structure). |
| **Composition** | Field reads, method calls (`attribute access` + `call`), and chained access all decompose to attribute access. `.` is the syntax per Semantic Purity. |

```orthon
point.x              # field access
list.append(item)    # method access (attribute access + call)
nested.value.field   # chained access
```

#### 3.2.5 `scope`

| Field | Value |
|-------|-------|
| **Category** | Data Operations Primitive |
| **Definition** | Lexical boundary for names and lifetimes. Defines a region where bindings are valid, lifetimes are determined, and visibility is contained. |
| **Semantic dimensions** | **Visibility** — names are visible only within their scope; scope is the mechanism that implements the Visibility dimension's module-level encapsulation. **Lifetime** — values live until scope exit per Semantic Invariant 3 (see SEMANTIC_MODEL.md). |
| **Orthogonal to** | All other primitives — scope is the container, not an operation. A scope with zero bindings is still a scope (it still affects lifetime). |
| **Composition** | Function bodies, block statements (`{ }`), module boundaries, and `if`/`when` branches all define scopes. Per D-09: `{ }` blocks require explicit `return` to produce a value; expression-level constructs (`if`, `match`, `when`) use last-expression-as-value. Scopes nest: an inner scope can shadow outer bindings (a new `identifier` with the same name). |

```orthon
{                     # scope begins
    let tmp = setup() # tmp lives only within this scope
    process(tmp)
}                     # scope ends, tmp destroyed

if age >= 18:         # if branches define their own scopes
    "adult"
else:
    "minor"
```

#### 3.2.6 `reference`

| Field | Value |
|-------|-------|
| **Category** | Data Operations Primitive |
| **Definition** | Indirection to a value without ownership transfer. Creates a handle that points to an existing value without claiming ownership. |
| **Semantic dimensions** | **Ownership** — is the borrowing mechanism; a reference allows access without transfer of ownership. **Lifetime** — reference lifetime must be ≤ referent lifetime (SEMANTIC_MODEL.md § Lifetime). |
| **Orthogonal to** | `pack` (reference points to a value; pack combines values). `assignment` (reference does not create a new binding — it aliases an existing one; assignment creates or updates a binding). |
| **Composition** | **One primitive, two modes:** shared (read-only, `&T`) and exclusive (mutable, `&mut T`). Both are modes of the same reference primitive — they differ only in the kind of access they grant, not in the indirection mechanism. Per D-10: one keyword (`mut`) serves both binding-level and reference-level mutation marking. `mut x` at the binding site declares a mutable binding; `&mut` is the reference's mutation mode. Interior mutability (Cell/RefCell patterns) is NOT a primitive — it is a derived Standard Library feature built on `reference` + mutation semantics, deferrable to the Standard Library. Function parameters use reference for borrowing; class identity semantics build on reference. |

```orthon
&x                 # shared reference (read-only)
&mut x             # exclusive reference (mutable)

fun len(s: &String) -> Int   # borrowing parameter
    return s@len()
```

---

## 4. Exclusions and Decomposition

The following concepts are explicitly **excluded** from the primitive set.
Each exclusion includes its decomposition into primitives, the rationale,
and the source document that justifies the decomposition.

| Excluded Concept | Decomposition | Rationale | Source |
|---|---|---|---|
| `operator definition` | Syntactic sugar: `function` with a symbolic name | Symbols are brevity; named functions are the canonical form. The actual primitive is `function`. | [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) § Named Before Symbolic, D-01 |
| `struct` | Type-level convenience: `pack` + `identifier` + `scope` | Structs are data, not behaviour. The type keyword is a convenience that bundles pack composition, a named identifier, and a scope for member definitions. | [`STRUCT_AS_VALUE_TYPE.md`](../how/concepts/research/essential/STRUCT_AS_VALUE_TYPE.md), D-03 |
| `class` | Type-level convenience: `pack` + `reference` + `scope` + `assignment` | Classes are reference types built on indirection. The class keyword bundles pack (field composition), reference (identity semantics), scope (member visibility), and assignment (field initialization). | [`CLASS_WITH_ACT.md`](../how/concepts/research/essential/CLASS_WITH_ACT.md), D-03 |
| `delegate` | Execution policy: `reference` + `function` + ownership | Execution is orthogonal to declaration. Delegate composes a reference to a function with ownership semantics to define how something executes, not what it does. | [`DELEGATE.md`](../how/concepts/research/essential/DELEGATE.md), D-05 |
| `namespace` | Organizational: `identifier` + `scope` + visibility | Namespaces are naming convenience — a named scope with visibility rules. The identifier names the namespace; the scope defines its boundary; visibility controls access. | [`NAMESPACES.md`](../how/concepts/research/essential/NAMESPACES.md), D-05 |
| `act` (isolation) | Concurrency modifier: `function` tag (built on `reference` + scope) | Act is a concurrency modifier on the function primitive — it tags a function as isolated, but the underlying mechanism is reference (access to shared state) and scope (lifetime boundaries). | [`CLASS_WITH_ACT.md`](../how/concepts/research/essential/CLASS_WITH_ACT.md) |
| `act` fields | Isolated access: `reference` + scope | Fields marked `act` are accessed through a reference that enforces isolated (actor-style) access. The primitives are reference and scope; `act` is a policy annotation. | [`CLASS_WITH_ACT.md`](../how/concepts/research/essential/CLASS_WITH_ACT.md) |

Each excluded concept is either (a) syntactic sugar over a real primitive,
(b) a composition of multiple primitives, or (c) a meta-language annotation.
Including any would violate orthogonality or mix abstraction levels.

---

## 5. D-10 Resolutions

The following open items from Phase 2 (SEMANTIC_MODEL.md § Mutation, "Deferred to Phase 3")
are resolved for the primitive set:

### 5.1 Interior Mutability (Cell/RefCell)

**NOT a primitive.** Interior mutability is a derived Standard Library feature
(or Implementation Strategy concern) built on `reference` + mutation semantics.
The reference primitive provides the indirection; interior mutability is a
runtime pattern layered on top that uses that indirection to provide mutation
through a shared reference. It is not an irreducible atomic operation.

**Consequence:** The primitive set is complete without interior mutability.
Cell/RefCell-style types are composed of `reference` + `assignment` (mutation
through indirection) and belong in the Standard Library or as an
Implementation Strategy opt-in.

### 5.2 Mutation in Closures

Closures capture variables as **immutable by default**. Explicit `mut` on the
captured binding is required for mutable capture, consistent with the
overall immutability-by-default model (SEMANTIC_MODEL.md § Mutation).

**Decomposition:** A closure is `function` + `identifier` + `scope` (captured
environment). The captured bindings follow the same mutation rules as any
other binding — `val` (immutable) or `var` (mutable) at the capture site.

### 5.3 `mut` vs `&mut`

**One keyword (`mut`) serves both** binding-level and reference-level mutation
marking:

- `mut x` at the binding site declares a mutable binding (var-style).
- `&mut` is the reference's mutation mode — part of the reference primitive's
  two-mode design (shared `&T`, exclusive `&mut T`).

The semantic distinction (mutable binding vs. mutable reference) is separate
from the keyword count. One keyword suffices for both, consistent with
Explicitness — both uses are visibly marked at their sites.

---

## 6. Metadata Protocol

Per D-07, all metadata, protocol methods, and special operations are accessed
via the `@` prefix notation. This is called the **Metadata Protocol**.

```
list@len()          # protocol method — length
obj@fields          # reflective access — field list
type@name           # reflective access — type name
```

The `@` prefix is **not a primitive** — it is a syntactic marker for metadata
access on types and values. The operation triggered by `@` access ultimately
decomposes to `function` + `call` (the protocol method is a function defined
on the type, and invoking it is a call).

**Distinction from attribute access (`.`):**

| Syntax | Purpose | Example |
|--------|---------|---------|
| `.` | User-defined properties and methods | `point.x`, `list.append()` |
| `@` | Language-defined metadata and protocols | `list@len()`, `obj@fields` |

Per [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) § Semantic Purity:
the `@` prefix makes metadata access syntactically distinct from attribute
access, ensuring no symbol carries two meanings depending on context.

**Consequence:** System functions like `len()`, `sorted()`, `str()` are mapped
to `@`-prefixed protocol methods. Free functions (`len(obj)`) may exist as
syntactic sugar that compiles to `obj@len()` — this is a Phase 4/5 decision.

---

## 7. `emit` — Lazy by Default

Per D-06 (correction to Phase 2 D-04), `emit` is **lazy by default** — it
produces values on demand, not eagerly. For eager sequence production, use
`return` with an aggregate collection.

```orthon
# Lazy — produces values on demand
fun range(n: Int) -> Sequence<Int>
    emit i for i in 0..n

# Eager — constructs and returns a collection
fun range(n: Int) -> List<Int>
    return List(i for i in 0..n)
```

**Rationale:** Lazy `emit` aligns with Sequence as a description of *what*,
not *how* (see SEMANTIC_MODEL.md § Evaluation). Eager production is better
served by constructing a collection and returning it — the distinction is
explicit in the choice of mechanism.

**Evaluation Policy:** The evaluation mode for `emit` is settled as lazy at
the language level. Phase 4 concepts (iterators, generators) are built on
lazy `emit`.

---

## 7b. Representation Modifiers (Orthogonal Annotations)

Per [EDR-038](../how/decision_records/architecture/EDR-038-representation-modifiers.md),
representation modifiers (`struct(T)`, `boxed(T)`, `shared(T)`,
`atomic(T)`, `ffi(T)`, `packed(T)`) are **orthogonal annotations on
existing primitives** — they control *how* a primitive is materialised
without adding new primitive operations.

**Decomposition:**

| Modifier | Primitive | Effect |
|----------|-----------|--------|
| `struct(T)` | `pack` | Pack without runtime metadata (plain data layout) |
| `boxed(T)` | `reference` | Heap-allocate with full runtime type info |
| `shared(T)` | `reference` | Reference-counted shared ownership |
| `atomic(T)` | `reference` | Thread-safe atomic access guarantees |
| `ffi(T)` | `pack` | C-compatible memory layout (POD) |
| `packed(T)` | `pack` | Densely packed with minimal alignment padding |

**Key property:** `struct(User)` and `boxed(User)` are the *same semantic
type* — they differ only in storage strategy. Assignment between
representations is safe; the compiler inserts the conversion. Method
dispatch works regardless of representation.

These modifiers are **excluded from the primitive set** because they add
constraints to existing operations rather than new operations. They are
annotations on primitives, not primitives themselves. Full specification
of their syntax and semantics is deferred beyond v0.1.

---

## 8. Composition Rules

Primitives compose according to these rules:

1. **Nesting composition.** Primitives compose by nesting — one primitive's
   output feeds another's input. Example: `call` produces a value consumed
   by `assignment`; `pack` produces a composite consumed by `attribute access`.

2. **Orthogonality guarantee.** No primitive requires another primitive to
   be meaningful. Each primitive can appear independently: a `scope` with
   no bindings, a `literal` without assignment, a `call` whose return value
   is discarded.

3. **Derived features are compositions.** A derived feature (Phase 4) is
   defined by its decomposition into primitives. For example:
   - A `for` loop: `scope` + `call` (condition check) + `assignment` (loop
     variable) + `call` (body execution)
   - Pattern matching: `scope` (per branch) + `unpack` (destructuring) +
     `call` (condition evaluation)
   - Error handling (`try`/`catch`): `scope` (try block) + `call` (error
     propagation) + `scope` (catch block)

4. **Incomplete set detection.** If a derived feature cannot be decomposed
   onto this set, the primitive set is incomplete and must be revisited
   before Phase 4 proceeds.

5. **Non-overlap verification.** If two primitives can express the same
   operation in different ways without a meaningful semantic distinction,
   the set is not orthogonal and must be consolidated before Phase 4.

---

## 9. Summary: The Primitive Set

| # | Primitive | Category | Semantic Dimensions Served |
|---|-----------|----------|--------------------------|
| 1 | `literal` | Data | Data |
| 2 | `identifier` | Data | Identity, Ownership |
| 3 | `pack`/`unpack` | Data | Data, Representation |
| 4 | `assignment` | Data Operations | Evaluation, Ownership |
| 5 | `function` | Data Operations | Evaluation |
| 6 | `call` | Data Operations | Evaluation, Lifetime |
| 7 | `attribute access` | Data Operations | Visibility |
| 8 | `scope` | Data Operations | Visibility, Lifetime |
| 9 | `reference` | Data Operations | Ownership, Lifetime |

**Count:** 9 conceptual primitives (counting pack/unpack as one symmetric pair)
across two categories.

---

## 10. Verification

The primitive set defined above has been verified against every concept
research file across all tier directories in `how/concepts/research/`
(~132 files total). This section records the verification results,
confirms the set is both **complete** (no concept is inexpressible) and
**minimal** (removing any primitive makes at least one concept
inexpressible), and documents any gaps found and their resolutions.

### 10.1 Verification Methodology

1. **Essential tier (42 files)** — Verified individually by reading
   each file's Issue, Model, and Policy Footprint sections to confirm
   its features decompose onto the 9 primitives. Files already consumed
   by Phase 2 or Phase 3 (20 files) were confirmed as accounted for.
   Compiler-infrastructure meta-concepts (2 files) were verified as
   acting *upon* primitive-level operations, not requiring new
   primitives.
2. **Important tier (36 files)** — Verified in category batches
   (control flow, data structures, type system, metaprogramming,
   concurrency). Each batch's core pattern was decomposed to confirm
   the primitive set covers all needed operations.
3. **Deferrable tier (54 files)** — Verified in category batches.
   Deferrable concepts (v0.2/v0.3) were confirmed conceptually
   decomposable; none requires a primitive not in the set.
4. **Reject tier (0 files)** — The reject tier is currently empty; no
   verification needed.

### 10.2 Essential Tier — Verification Results

**Consumed by Phase 2 (Semantic Model) — 10 files:**
These files informed `what/SEMANTIC_MODEL.md` (EDR-013) and are
accounted for as sources of the semantic dimensions that primitives
serve. No further decomposition needed.

| File | Role |
|------|------|
| `DATA_MODEL.md` | Data representation — primitives produce/manipulate data |
| `OWNERSHIP.md` | Ownership dimension — `assignment`, `reference` primitives |
| `OWNERSHIP_METAPROPERTY.md` | Ownership metaproperty — `assignment` |
| `OWNERSHIP_TRANSFER_OPERATOR.md` | Ownership transfer syntax — `assignment` |
| `MUTABILITY.md` | Mutation dimension — `assignment`, `reference` |
| `VALUE_SEMANTICS.md` | Value vs reference — `pack`, `reference` |
| `IDENTITY_BASED_SAFETY.md` | Identity dimension — `identifier` |
| `VISIBILITY_AND_ENCAPSULATION.md` | Visibility dimension — `scope`, `attribute access` |
| `SCOPED_RESOURCE_LIFECYCLE.md` | Lifetime dimension — `scope` |
| `EXPRESSION_ORIENTED_LANGUAGE.md` | Evaluation dimension — `call`, `assignment` |

**Consumed by Phase 3 (Primitive Blocks synthesis) — 10 files:**
These files informed the primitive set's definition, exclusion
decisions, and organizing taxonomy. See EDR-016 § Context for full
disposition.

| File | Role | Disposition |
|------|------|-------------|
| `FOUNDATIONAL_ABSTRACTIONS.md` | Organizing taxonomy (Data/Data Modifiers) | Modified |
| `EXCLUSIVE_DECLARATIONS.md` | fun/proc/new as function tags | Modified |
| `STRUCT_AS_VALUE_TYPE.md` | struct exclusion rationale | Superseded |
| `CLASS_WITH_ACT.md` | class exclusion rationale; act decomposition | Superseded |
| `ACT_AS_FUNCTION.md` | Historical; superseded by DELEGATE.md | Superseded |
| `FUNCTIONS.md` | function + call as separate primitives | Modified |
| `FINAL_BY_DEFAULT.md` | Inheritance policy — not primitive-level | N/A |
| `NAMESPACES.md` | namespace exclusion rationale | Superseded |
| `DELEGATE.md` | delegate exclusion rationale | Superseded |
| `COMPOSITION_OVER_INHERITANCE.md` | Design pattern — not primitive-level | N/A |

**Remaining essential concepts — 22 files:**

| File | Decomposition | Gap? |
|------|---------------|------|
| `ALLOCATION.md` | `scope` (arena lifetime) + `reference` (borrowing) + `assignment` | None |
| `AST_MACROS.md` | `function` + `call` (macro = compile-time function invocation) | None |
| `COMPILER_AS_STATIC_ANALYZER.md` | Meta-concept — compiler infrastructure acting on primitives | N/A (meta) |
| `COMPILE_TIME_EXECUTION.md` | `function` + `call` (compile-time evaluation) | None |
| `COMPOSABLE_COLLECTION_OPS.md` | `function` + `call` (combinators) + `pack/unpack` (construction) | None |
| `CONCURRENCY_MODEL.md` | `reference` + `scope` + `function` + `call` | None |
| `CONTEXT_PARAMETERS.md` | `function` + `identifier` + `scope` | None |
| `EQUALITY.md` | `function` + `call` (protocol method) | None |
| `ERROR_HANDLING.md` | `function` + `call` + `assignment` + `scope` | None |
| `ERROR_UNION.md` | `pack/unpack` + `function` + `call` | None |
| `EXECUTION_PROGRAM.md` | `function` + `call` + `scope` + `identifier` | None |
| `GENERICS.md` | `function` + `identifier` (type params) + `call` (instantiation) | None |
| `ITERATOR_PROTOCOL.md` | `function` + `call` + `scope` | None |
| `LAZY_SEQUENCE_GENERATORS.md` | `function` + `call` + `emit` | None |
| `NULL_SAFETY.md` | `pack/unpack` (Option sum type) + `function` + `call` | None |
| `PATTERN_MATCHING.md` | `scope` + `unpack` + `function` + `call` | None |
| `PATTERN_MATCHING_DISPATCH.md` | `scope` + `unpack` + `function` + `call` | None |
| `REGION_BASED_MEMORY_MANAGEMENT.md` | `scope` + `reference` | None |
| `REPRESENTATION_MODIFIERS.md` | `reference` + `pack` | None |
| `TRAITS.md` | `function` + `identifier` + `call` | None |
| `TYPE_INFERENCE.md` | Meta-concept — compiler algorithm acting on types | N/A (meta) |
| `TYPE_LEVEL_NULL_SAFETY.md` | `function` + `identifier` + `pack/unpack` | None |

**Result: 20 of 22 remaining essential concepts decompose cleanly.**
2 meta-concepts (`COMPILER_AS_STATIC_ANALYZER.md`, `TYPE_INFERENCE.md`)
describe compiler infrastructure that validates programs built from
primitives, not language constructs requiring new primitives.

### 10.3 Important Tier — Verification Results (36 files)

Verified by category. All concepts decompose onto the 9 primitives.

| Category | Files | Core Decomposition Pattern | Gap? |
|----------|-------|----------------------------|------|
| **Control flow** | `ASYNC_AWAIT.md`, `EMIT_AS_INTERMEDIATE_RESULT.md`, `GENERATORS.md`, `ITERATION_LOOP.md` | `function` + `call` + `scope` | None |
| **Data structures** | `ALGEBRAIC_DATA_TYPES.md`, `COLLECTION_LITERAL_SYNTAX.md`, `PERSISTENT_DATA_STRUCTURES.md`, `SLOTS.md`, `SPAN.md` | `pack/unpack` + `identifier` + `scope` | None |
| **Type system** | `GRADUAL_TYPING.md`, `LITERAL_TYPES.md`, `SMART_CAST.md`, `STRUCTURAL_TYPING.md`, `TYPE_LEVEL_COMPUTATION.md`, `UNION_INTERSECTION_TYPES.md` | `function` + `identifier` + `pack/unpack` | None |
| **Metaprogramming** | `DECLARATION_BY_ASSIGNMENT.md`, `DECLARATIVE_CONSTRUCTS.md`, `EXTENSION_FUNCTIONS.md`, `NAMED_AND_OPTIONAL_PARAMETERS.md` | `function` + `call` + `identifier` | None |
| **Concurrency** | `ASYNC_AS_EXPLICIT_MODIFIER.md`, `CONCURRENCY.md`, `PUSH_STREAMS.md` | `reference` + `scope` + `function` | None |
| **Behavioral contracts** | `CONTRACTS.md`, `DELEGATION.md`, `PROPERTIES.md`, `COMMAND_PATTERN_VIA_DELEGATE.md` | `function` + `identifier` + `call` | None |
| **Ergonomics** | `COPY_ON_WRITE.md`, `DATACLASSES.md`, `DECLARATIVE_MULTI_KEY_SORT.md`, `DERIVE_SERIALIZATION.md`, `ENUM_ALTERNATIVES.md`, `OBJECT_INITIALIZATION.md`, `SORTING.md`, `UNPACKING.md` | Composition of `pack/unpack` + `function` + `call` | None |
| **Context-related** | `CONTEXT_LIMITED_MODULES.md` | `scope` + `identifier` + `visibility` | None |

**Result: All 36 important-tier files decompose cleanly.**

### 10.4 Deferrable Tier — Verification Results (54 files)

Verified in category batches. No deferrable concept requires a primitive
not in the set. Key category patterns:

| Category | Sample Files | Core Decomposition | Gap? |
|----------|-------------|-------------------|------|
| Actors / active objects | `ACTORS.md`, `ACT_AS_ACTIVE_OBJECT.md` | `reference` + `scope` + `function` | None |
| Metaprogramming | `COMPILE_TIME_METAPROGRAMMING.md`, `HOMOICONICITY.md`, `METAOBJECTS.md`, `REFLECTION_ALTERNATIVES.md` | `function` + `call` + `identifier` | None |
| Compile-time safety | `COMPILE_TIME_CONCURRENCY_SAFETY.md`, `SAFE_SANDBOX.md`, `TYPED_HOLES.md` | Compiler infrastructure — acting on primitives | N/A |
| Type system extensions | `DEFAULT_INTERFACE_METHODS.md`, `FUNCTION_OVERLOADING.md`, `NESTED_CLASSES.md`, `PROTOCOL_EXTENSIONS.md`, `ROOT_OBJECT.md` | `function` + `identifier` + `call` | None |
| Concurrency | `SOFTWARE_TRANSACTIONAL_MEMORY.md`, `TASK_PARALLEL_LIBRARY.md` | `reference` + `scope` + `function` | None |
| Language ergonomics | `BUILDER_PATTERN_ELIMINATION.md`, `CODE_BLOCK_AS_HOF.md`, `LAZY_INITIALIZATION.md`, `PROPERTY_WRAPPERS.md`, `STRATEGY_PATTERN_ELIMINATION.md` | `function` + `call` + `scope` | None |
| Composition patterns | `CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`, `COMPOSITION.md`, `EXPLICIT_COMPOSITION.md`, `MIXIN.md`, `PROTOTYPE.md` | `pack/unpack` + `function` + `call` | None |
| Events / observation | `EVENTS.md`, `OBSERVER.md`, `DISPOSABLE_PATTERN.md` | `function` + `call` + `reference` | None |
| Deployment / tooling | `DEVELOPER_TOOLING.md`, `DIALECTS.md`, `IDEMPOTENT_GENERATION.md`, `INTERACTIVE_DEVELOPMENT.md`, `LLM_NATIVE_TOOLCHAIN.md`, `NATIVE_COMPILATION.md`, `SCRIPT_EXECUTION_MODEL.md` | Meta-concepts — acting on primitives | N/A |
| Domain-specific | `DIFFERENTIABLE_PROGRAMMING.md`, `LITERATE_PROGRAMMING.md`, `QUERY_EXPRESSIONS.md`, `SEMANTIC_ANNOTATIONS.md` | `function` + `call` + pack/unpack | None |
| Dynamic / runtime | `DYNAMIC_COLLECTIONS.md`, `DYNAMIC_TYPING.md`, `INDEXERS.md`, `SIGNIFICANT_WHITESPACE.md`, `OPEN_CLASSES.md`, `SINGLETON_CLASS.md`, `SINGLETON_PATTERN_ANALYSIS.md`, `TOP_LEVEL_DECLARATIONS.md`, `USING_DIRECTIVES.md` | `function` + `call` + `identifier` + `scope` | None |
| Misc | `ASYNC_LAMBDA.md`, `CUSTOM_OPERATORS.md`, `METADATA_ANNOTATIONS.md`, `OBJECTS_AND_SINGLETONS.md`, `PARTIAL_CLASSES.md` | Composition of existing primitives | None |

**Result: All 54 deferrable-tier files decompose cleanly.**
No deferrable concept requires a primitive not in the 9-primitive set.

### 10.5 Completeness Verification

**The primitive set is complete.** Every concept in the research catalog
(~132 files across all tiers) decomposes onto one or more of the 9
primitives. No concept was found that requires an operation not
expressible by the set.

### 10.6 Minimality Verification

**The primitive set is minimal.** Removing any single primitive would
make at least one known feature inexpressible:

| If removed | What becomes inexpressible | Key concept |
|------------|---------------------------|-------------|
| `literal` | No way to create a value from source text | Every program |
| `identifier` | No way to name a value or refer to it later | Every binding |
| `pack`/`unpack` | No structs, tuples, records, destructuring | `STRUCT_AS_VALUE_TYPE.md` |
| `assignment` | No variable bindings, parameter passing | Every function |
| `function` | No reusable computation definition | `FUNCTIONS.md` |
| `call` | No way to invoke a computation | Every program |
| `attribute access` | No field access, no method calls | `TRAITS.md` |
| `scope` | No lexical boundaries, no lifetime management | Every block |
| `reference` | No borrowing, no indirection, no class identity | `CLASS_WITH_ACT.md`, `DELEGATE.md` |

### 10.7 Gaps Found and Resolutions

**No gaps found.** All ~132 concept research files decompose onto the
9-primitive set. No missing primitive was discovered. The set is ready
for Phase 4 (Derived Features) where each concept will receive a formal
decomposition as part of its acceptance process.

---

## See Also

- [`SEMANTIC_MODEL.md`](SEMANTIC_MODEL.md) — The six Semantic Dimensions (Identity,
  Ownership, Mutation, Evaluation, Visibility, Lifetime) that each primitive serves.
- [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) — The constitutional design
  rules that govern the primitive set: Orthogonality, Minimal Core, Semantic Purity.
- [`GLOSSARY.md`](GLOSSARY.md) — Terminology reference for primitive-related terms.
- [`ROADMAP.md`](../when/ROADMAP.md) § Phase 3 — Where this document fits in the
  engineering design pipeline.
- [`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md) § Semantic Dependency
  Architecture — The 6-level hierarchy this document occupies (Level 1).
- [`FOUNDATIONAL_ABSTRACTIONS.md`](../how/concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md) — The Data/Data Modifiers taxonomy that organizes the primitive set.
- [`UNPACKING.md`](../how/concepts/research/important/UNPACKING.md) — Symmetry
  principle for pack/unpack as a single primitive with two operations.

---

## EDR

> **EDR-016 — Acceptance of Orthon Primitive Blocks.**
> Status: Accepted (2026-07-27). This EDR formally accepts the
> primitive set defined above, recording the disposition of all source
> research documents and the verification that every concept research
> file decomposes onto this set.
>
> See: [`EDR-016-primitive-blocks.md`](../how/decision_records/architecture/EDR-016-primitive-blocks.md).
