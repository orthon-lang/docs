# Language Astonishment Catalog

> Exploratory note. A catalog of documented cases in which seven mainstream
> languages (Python, Java, Go, Rust, TypeScript, Swift, Zig) break the
> Principle of Least Astonishment, break their own internal consistency, or
> contradict the programmer's model of the language; the recurring causes
> behind those cases; and an analysis of how far a language designer can
> reduce them.

**Status:** *Analysis — not a decision record. Section 9 contains observations
for Orthon, not design decisions.*

**Date:** 2026-10-25

**See also:** [`how/DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) § POLA,
[`what/LANGUAGE_PROBLEMS.md`](../what/LANGUAGE_PROBLEMS.md),
[`orthogonality-pain.md`](orthogonality-pain.md),
[`language-llm-comparison.md`](language-llm-comparison.md),
[`plt-classical-problems.md`](plt-classical-problems.md)

---

## 1. Questions

1. Which behaviors do the language teams and the expert community document as
   astonishing, internally inconsistent, or contradicting the language's own
   model, in Python, Java, Go, Rust, TypeScript, Swift, and Zig?
2. How can a language designer reduce the number of such cases?
3. Can they be eliminated entirely, and by what means?

## 2. Definitions and assumptions

**Astonishment** is a divergence between the behavior a programmer predicts and
the behavior the language defines. The prediction is produced by the
programmer's mental model of the language, built from four inputs: the
surface form of the code, prior knowledge (mathematics, natural language,
other languages), explicit instruction (documentation, tutorials), and
accumulated experience. The principle itself is old: the "Law of Least
Astonishment" first appeared in the *PL/I Bulletin* in 1967, and a 1972
formulation reads "every construct in the system should behave exactly as its
syntax suggests" [S29]. Its corollary in Cowlishaw's REXX design is that a
necessary feature with a high astonishment factor may need to be redesigned
[S29].

Assumptions that bound the catalog:

- **A1. Reader.** A competent programmer who knows at least one mainstream
  language and is new to the language under study.
- **A2. Evidence standard.** An entry is included only if it is documented by
  a specification, a language team, an accepted language-change proposal, an
  edition migration guide, or recurring expert literature. Popularity on
  social media is not evidence. This biases the catalog toward problems that
  were noticed and written up, so older and larger ecosystems (Python, Java)
  are better documented than newer ones (Zig, Swift 6).
- **A3. Depth is not ranking.** Seven to nine entries per language in
  §4.1–4.7, plus 10 notation entries in §4.9 (one or two per
  language). The count does not rank languages: a larger user base and a
  longer history produce more documented cases.
- **A4. Astonishing is not wrong.** Many entries are deliberate trade-offs. The
  Status column records whether the language team accepted the behavior,
  fixed it, or left it open.
- **A5. Versions.** As of October 2026, with the sources listed in §10. Entries
  marked *fixed* describe the change and the version that carries it.
- **A6. Verification.** Source tags are `[V]` (page fetched and read for this
  note) or `[K]` (cited from established knowledge; not fetched; to be checked
  against the primary source before an EDR relies on it). An entry with only
  `[K]` and no source id is widely documented but not traced to one primary
  source. This follows the convention already used in
  [`what/LANGUAGE_PROBLEMS.md`](../what/LANGUAGE_PROBLEMS.md).
- **A7. Audience dependence.** "Least astonishing" depends on the expected
  audience and on convention [S29]. Matz states that "the principle of least
  surprise means principle of least *my* surprise" and that it applies after
  one has learned the language well [S30]. The catalog therefore uses A1 as its
  reference reader and treats astonishment as a property of a
  language-and-reader pair, not of a language alone.

## 3. Classification

Every entry carries one or more classes, a primary pattern (and optional
secondary ones), a visibility, and a status.

| Class | Meaning | Detecting question |
|---|---|---|
| **EXP** | A rule differs from an expectation imported from mathematics, natural language, or other languages. | Does the same notation mean something else elsewhere? |
| **CON** | The language is inconsistent with itself: one form with several meanings, several forms with one meaning, or an exception to a rule the language set. | Does a rule the language itself established fail here? |
| **MOD** | The programmer's intuition about the language's *stated* model contradicts what the language does. | Does applying the language's own model ("values are copied", "closures capture", "types describe values") predict a different result? |

| Visibility | Meaning |
|---|---|
| silent | A different result with no signal. |
| loud-rt | A run-time error or panic. |
| loud-ct | A compile-time error. |

Visibility matters because the cost of a surprise depends on when and how it
announces itself (§7). The twelve patterns (P1–P12) are named in §5; they
describe *what the surprise is about*, and §6 derives a design lever from each.

## 4. Catalog

65 entries (55 in §4.1–4.7, 10 on syntax in §4.9). Class totals (an entry may carry two classes): CON 29, EXP 15, MOD 31.

### 4.1 Python

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| PY-1 | Mutable default argument. Expected: the default expression is evaluated on every call. Actual: it is evaluated once at `def` time, and every call that omits the argument shares the same object. | MOD, CON | P3, P2 | silent | open; linters warn | S2 [K] |
| PY-2 | Late-binding closures. `[lambda: i for i in range(3)]` yields closures that all see the final `i`: a closure captures the variable, not its value at creation. | MOD | P3 | silent | open; workaround: bind through a default argument | S1 [V] |
| PY-3 | Assignment anywhere in a function makes the name local for the whole function, so a read that precedes the assignment raises `UnboundLocalError` even when a global of that name exists. | EXP, CON | P3 | loud-rt | open; `global` / `nonlocal` | S1 [V] |
| PY-4 | Sequence repetition copies references. `[[0] * 3] * 3` yields three references to one row; mutating one row changes all. | MOD | P2 | silent | open | S1 [V] |
| PY-5 | Augmented assignment is not always `a = a + b`. For a list inside a tuple, `t[0] += [1]` mutates the list in place and then raises `TypeError` on the rebinding, leaving the state changed. | CON | P7 | loud-rt (after the mutation) | open | S1 [V] |
| PY-6 | Observable caching. `a is b` for equal small integers or strings depends on interpreter caching and interning; the data model allows an operation on immutable values to return an existing object. | MOD | P1, P10 | silent | accepted; `SyntaxWarning` flags `is` with a literal | S3 [K] |
| PY-7 | Truthiness doubles as a test for emptiness, zero, and absence: `if not result` treats a legitimate `0` or `[]` as missing. | EXP | P5, P7 | silent | accepted | S3 [K] |
| PY-8 | A mutable class attribute is shared by all instances; `self.n += 1` reads the class attribute but writes a new instance attribute, so one name is read and written in different places. | MOD, CON | P2, P3 | silent | open | S3 [K] |
| PY-9 | `else` on `for` / `while` runs when the loop was not left by `break`. The keyword means something other than `if`'s `else`; a core developer has said `nobreak` would have been the clearer name. | EXP, CON | P7 | silent | accepted | S4 [K] |

### 4.2 Java

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| JV-1 | `==` compares values for primitives and identities for references. Boxing blurs the line: `Integer.valueOf` caches -128..127, so separately boxed 127s are `==` and separately boxed 128s are not. The JLS identity guarantee for boxing is narrow (limited to constant expressions after JDK-7190924). | MOD, CON | P1, P7 | silent | being fixed: JEP 401 (preview) makes `Integer` a value class whose `==` compares state | S5, S6 [V] |
| JV-2 | Two collection models with different rules: arrays are covariant and reified, generics are invariant and erased. `Object[] o = new String[1]; o[0] = 1;` compiles and throws `ArrayStoreException`; `new T[n]` and `x instanceof List<String>` are rejected. | CON | P6, P8 | loud-rt and loud-ct | open; erasure was kept for migration compatibility | S7 [K] |
| JV-3 | Every reference type is implicitly nullable. `Optional` arrived later, but it is itself a nullable reference and is discouraged for fields and parameters. | EXP, MOD | P5 | loud-rt | mitigated (annotations, static analysis, `Optional`) | S9 [K] |
| JV-4 | Checked exceptions do not compose with lambdas: the standard functional interfaces declare no `throws`, so stream pipelines need wrapping code. A feature of the original language is not orthogonal to one added in 2014. | CON | P9 | loud-ct | open | [K] |
| JV-5 | Integer arithmetic wraps silently: `Integer.MAX_VALUE + 1` is negative and `Math.abs(Integer.MIN_VALUE)` is negative. Checked variants (`Math.addExact`) are opt-in. | EXP | P4 | silent | accepted | S8 [K] |
| JV-6 | `list.remove(1)` on a `List<Integer>` removes by index, while `list.remove(Integer.valueOf(1))` removes by value; overload resolution prefers the signature that needs no boxing. | CON | P7 | silent | accepted | S8 [K] |
| JV-7 | Legacy date API: `java.util.Date` constructors take years offset from 1900 and 0-based months, and instances are mutable. `java.time` (Java 8) replaced it, but the old API stays in the platform. | EXP, CON | P8 | silent | fixed by replacement; legacy remains | [K] |
| JV-8 | A superclass constructor that calls an overridable method runs the subclass override before the subclass's fields are initialized. | MOD | P9 | silent | mitigated (lint rules) | S7 [K] |

### 4.3 Go

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| GO-1 | Typed nil in an interface. `var p *T = nil; var e error = p` gives `e != nil`: an interface value is nil only when both its dynamic type and its value are nil. | MOD | P1, P6 | silent | open | S12 [V] |
| GO-2 | Before Go 1.22 the loop variable of `for` was one variable per loop; closures and goroutines started in the loop saw its last value. The Go team calls it one of the most common Go mistakes. | MOD | P3 | silent | fixed in Go 1.22, enabled per module by the `go` line in `go.mod` | S10 [V] |
| GO-3 | A slice is a small header pointing into a shared array. Re-slicing does not copy; whether `append` writes into the shared array or reallocates depends on capacity; a small sub-slice keeps the whole array alive. | MOD | P2, P11 | silent | open | S11 [V] |
| GO-4 | `nil` names six different things (pointer, slice, map, channel, function, interface) with different behavior: appending to a nil slice works, writing to a nil map panics, receiving from a nil channel blocks forever. | CON | P5, P7 | mixed | accepted | S13 [K] |
| GO-5 | `x, err := f()` in an inner scope declares a new `err` that shadows the outer one; `:=` declares or assigns depending on which names already exist in the current scope. | CON | P7, P3 | silent | mitigated (shadow analyzer for `go vet` is opt-in) | S13 [K] |
| GO-6 | `for _, v := range xs` binds `v` to a copy of each element; assigning to `v` does not modify `xs`. | MOD | P3, P2 | silent | accepted | S13 [K] |
| GO-7 | `defer f(x)` evaluates `x` when the statement executes but runs `f` at function exit, not block exit; a `defer` in a loop accumulates until the function returns. | MOD | P3 | silent | accepted | S13 [K] |
| GO-8 | `==` on interface values compiles for any dynamic types but panics at run time if the dynamic type is not comparable (slice, map, function); the same holds for interface-typed map keys. | CON | P4, P1 | loud-rt | accepted | S13 [K] |

### 4.4 Rust

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| RS-1 | Integer overflow panics where overflow checks are enabled (debug builds by default) and wraps in two's complement otherwise; the Reference says builds "may result in panics or silently wrapped values on overflow, at the implementation's discretion". The same source behaves differently by build profile. | MOD, EXP | P4, P10 | silent in release | accepted; explicit `wrapping_*`, `checked_*`, `Wrapping<T>` | S16 [V] |
| RS-2 | Before Edition 2024, temporaries in the scrutinee of `if let` lived to the end of the whole `if let ... else`; a lock guard from `value.read()` was still held in the `else` branch, and a `write()` there deadlocked. | MOD | P3 | silent | fixed in Edition 2024 (`if_let_rescope` lint) | S14 [V] |
| RS-3 | Before Edition 2024, temporaries in a block's tail expression outlived the block's local variables: `c.borrow().len()` as the last expression failed with E0597. The fix breaks code that relied on the old extension and has no general semantics-preserving automatic rewrite. | MOD | P3 | loud-ct | fixed in Edition 2024; migration partly manual | S15 [V] |
| RS-4 | Temporary lifetime depends on the enclosing syntactic form: `match` scrutinee temporaries live to the end of the statement, `if let` (2024) drops them before `else`, tail-expression temporaries die before locals. The same expression gets a different lifetime in different positions. | CON | P7, P3 | silent | open | S14, S15 [V] |
| RS-5 | `async fn` is lazy: calling it does nothing until the future is polled or awaited. Programmers from JavaScript, C#, or Kotlin expect the work to start. | EXP | P12, P3 | silent (a `must_use` warning exists) | accepted (zero-cost futures) | [K] |
| RS-6 | Dropping a future at an `.await` cancels it at that point. In `select!` the losing branches are dropped, so work that was half done is lost; correctness depends on each function being "cancellation safe". | MOD | P9 | silent | open | S17 [K] |
| RS-7 | Generic arguments are written `::<>` in expressions (`"5".parse::<i32>()`) but `<>` in types (`Vec<i32>`), because `<` is also less-than. One construct, two spellings. | CON | P7 | loud-ct | accepted (parse ambiguity) | S17 [K] |
| RS-8 | Orphan rule: a crate may implement a trait for a type only if it owns the trait or the type. Traits look open to extension, but coherence closes them, forcing newtype wrappers for a foreign trait on a foreign type. | EXP | P9 | loud-ct | accepted (coherence) | S17 [K] |

### 4.5 TypeScript

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| TS-1 | Static types do not guarantee runtime values: `any`, `as` assertions, and inaccurate `.d.ts` declarations let typed code receive values of another type with no compiler complaint. A sound type system is a stated non-goal. | MOD | P6 | silent | accepted by design goals | S18, S19 [V] |
| TS-2 | Arrays are covariant: an array of `string` is assignable to an array of `string` or `number`, and pushing a number through the wider reference corrupts the original. | MOD | P6, P2 | silent | accepted | S19 [V] |
| TS-3 | Indexed access is not bounds-checked: `arr[i]` has type `T`, not `T` or `undefined`, unless the opt-in flag `noUncheckedIndexedAccess` is set. | EXP | P4, P6 | silent | mitigated by an opt-in flag (not part of `strict`) | S19 [V]; flag details [K] |
| TS-4 | Narrowing survives function calls: after `if (o.x !== null)`, a call to `f()` that resets `o.x` does not invalidate the narrowed type. | MOD | P6 | silent | accepted | S19 [V] |
| TS-5 | Excess-property checks apply to fresh object literals but not to the same value passed through a variable, so structurally identical values are treated differently. | CON | P7 | loud-ct or none | accepted | S20 [K] |
| TS-6 | `Object.keys(o)` returns `string[]`, not an array of `keyof T`, because structural subtyping lets `o` carry extra properties; the type is deliberate but contradicts the intuition that a typed object has known keys. | MOD | P6 | loud-ct | accepted | S20 [K] |
| TS-7 | Enums, namespaces, and parameter properties emit runtime code, in tension with the goal of a "fully erasable" type system; numeric enums historically accepted arbitrary numbers. | CON | P8, P7 | silent | mitigated (`erasableSyntaxOnly`) | S18 [V]; details [K] |
| TS-8 | JavaScript semantics pass through by design ("preserve runtime behavior of all JavaScript code"): coercing `==`, `typeof null === "object"`, dynamic `this`, `["1", "2", "3"].map(parseInt)`. | EXP | P8, P12 | silent | accepted by goal | S18 [V] |

### 4.6 Swift

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| SW-1 | `String` has no integer subscript. `Character` is a variable-width grapheme cluster, so integer indexing would cost O(n); the API exposes `String.Index` to keep the cost visible. Users report that basic tasks become hard. | EXP | P11, P12 | loud-ct | accepted | S23 [V] |
| SW-2 | A protocol name served both as a generic constraint and as a type (existential) with the same spelling; programmers confused the two and paid hidden costs. SE-0335 introduced explicit `any`; the rollout caused friction. | CON, MOD | P7, P11 | silent (cost) | fixed by migration (SE-0335) | S21 [V] |
| SW-3 | Concurrency model: understanding `Task { ... }` requires isolation inheritance, `Sendable`, and global actors; users report "async contamination" and fragmented documentation, and some argue that Swift 6 hard errors arrived before the frameworks were updated. | MOD | P9 | loud-ct | evolving (steering-group vision on approachable data-race safety) | S22 [V] |
| SW-4 | `let` is deep for value types and shallow for reference types: a `let` struct cannot be mutated, a `let` reference to a class instance can. A struct holding a class reference copies only the reference. | MOD, CON | P2, P7 | silent | accepted | [K] |
| SW-5 | Implicitly unwrapped optionals (`T!`): the type is optional but is used with non-optional syntax; absence traps at run time. | CON | P5 | loud-rt | accepted (interop, two-phase initialization) | S24 [K] |
| SW-6 | Property observers (`didSet`) do not fire for assignments made in the type's own initializer. | EXP | P3 | silent | accepted | S24 [K] |
| SW-7 | "The compiler is unable to type-check this expression in reasonable time" on modest expressions that mix literals and overloaded operators. Whether code compiles depends on search cost, not meaning, and the error does not explain the model. | MOD | P11, P9 | loud-ct | open | [K] |

### 4.7 Zig

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| ZG-1 | Parameters: the reference text says the compiler may pass large aggregates by reference ("whichever way Zig decides will be faster"), which would make aliasing observable; forum participants say the optimization was removed and the documentation is outdated. | MOD | P10, P2 | silent | open (documentation debt) | S25 [V] |
| ZG-2 | Result location semantics (initializing a destination in place) is described by a frequent contributor as a design that "didn't work out"; initializing in place lets the destination alias operands of the initializer. | MOD | P2, P10 | silent | evolving | S25 [V]; mechanism [K] |
| ZG-3 | Unused locals, unused parameters, and never-mutated `var` are hard compile errors, not warnings; users work around with `_ = x`. It is consistent with the language having no warning category [K]. | EXP | P12 | loud-ct | accepted (editor tooling auto-fixes) | S26 [V]; policy [K] |
| ZG-4 | Integer overflow is "illegal behavior": checked in Debug and ReleaseSafe, undefined in ReleaseFast and ReleaseSmall. Wrapping (`+%`) and saturating (`+\|`) operators are explicit. The meaning of one source depends on build mode. | MOD | P4, P10 | silent in unsafe modes | accepted | S28 [K] |
| ZG-5 | `==` does not work on slices and strings; element-wise comparison is `std.mem.eql`. The language forbids hidden loops and operator overloading, so `==` never does O(n) work. | EXP | P12, P11 | loud-ct | accepted | S28 [K] |
| ZG-6 | `anytype` parameters state no requirements; the contract is discovered when the function is instantiated, and errors appear inside the callee rather than at the call site. | MOD | P6 | loud-ct | open | S28 [K] |
| ZG-7 | Pre-1.0 churn: `std.io` was replaced in an "extremely breaking" change; async/await was removed and is planned to return; documentation lags the compiler. What users learned from tutorials stops being true. | MOD | P8, P10 | loud-ct | open until 1.0 | S27, S25 [V] |

### 4.8 Items shared by all seven languages

Some surprises belong to the machine or to a standard, not to one language, so
they are listed once rather than per language: IEEE 754 comparisons (`NaN` is
not equal to itself, so equality is not reflexive on floats, and floats are
not totally ordered); `0.1 + 0.2 != 0.3`; fixed-width integer overflow and
signed/unsigned mixing; operator precedence inherited from C (Go is the
exception, with a deliberately smaller precedence table); Unicode (bytes,
code points, and grapheme clusters give three different "lengths" of one
string). Each language chooses a different place to put the surprise (see SW-1
for one extreme and PY-6 for another). [K]

### 4.9 Syntax and notation

The entries in §4.1–4.7 are mostly about semantics. This section collects a
different kind of astonishment, in which the same characters mean different
things depending on where they stand, what is inside them, or where the line
ends. The cause is the one named by P7: a small stock of delimiters is shared
by many constructs, and the parser, not the reader, decides which construct is
meant.

**Symbol roles.** The table counts the constructs that share each bracket pair.
A role is a construct the reader must tell apart, as named in the language's
reference. The counts are a rough indicator; I did not test how sensitive they
are to the counting rule. Only Python's roles were checked against a fetched
reference (S36); the other rows are `[K]`. The Orthon row describes documents,
not behavior (§9).

| Language | `( )` | `[ ]` | `{ }` |
|---|---|---|---|
| Python | 4: grouping, tuple, call, generator expression | 4: list display, list comprehension, subscription or slice, type parameterization (`list[int]`) | 4: dict display, set display, comprehension, replacement field in f-strings |
| Java | 4: grouping, call, cast, lambda parameters | 3: array type, array creation, element access | 4: block, class body, array initializer, lambda body |
| Go | 3: grouping, call, conversion | 4: index, slice type `[]T`, array type `[N]T`, type arguments `F[T]` | 2: block, composite literal |
| Rust | 3: grouping, tuple or unit, call | 5: array literal, array type `[T; N]`, slice type `[T]`, indexing, attribute `#[...]` | 3: block, struct literal, placeholder in format strings |
| TypeScript | 3: grouping, call, arrow-function parameters | 5: array literal, element access, array type `T[]`, tuple type, destructuring pattern | 4: block, object literal, object type, destructuring pattern |
| Swift | 3: grouping, tuple, call | 4: array literal, dictionary literal, subscript, type sugar `[T]` and `[K: V]` | 2: code block, closure |
| Zig | 2: grouping, call | 3: array type `[N]T`, slice type `[]T`, indexing or slicing | 3: block, `void` value, anonymous literal `.{}` |
| Orthon (as documented) | 1+: call `fn(args)` (EDR-085); other roles not located | 4: list literal (candidate), indexing, array types, slices (the last three reserved by EDR-086) | 3: map literal, set literal, scope block; the empty case is an open question |

**Entries.** Ten corner cases, at least one per language. The class is mostly
CON, because the clash is between two roles of one form.

| ID | Problem | Class | Patterns | Visibility | Status | Sources |
|---|---|---|---|---|---|---|
| PY-S1 | Empty braces `{}` denote an empty dictionary, not an empty set, although `{1, 2}` is a set. The reference states that an empty set cannot be constructed with `{}`; it is spelled `set()`. | CON | P7 | silent until a set method is called | accepted | S36 [V] |
| PY-S2 | Parentheses do not make a tuple; the comma does. `(1)` is the integer 1, `(1,)` is a tuple, and `x = 1,` also binds a tuple, so a stray trailing comma changes a value's type with no error on that line. Only the empty tuple `()` is made by parentheses alone. | CON | P7 | silent | accepted | S36 [V] |
| JV-S1 | Braces without a keyword mean different things by position: `int[] a = {1, 2};` is an array initializer, but `a = {1, 2};` is rejected and must be written `new int[]{1, 2}`. The same literal is legal in a declaration and illegal as an expression; elsewhere `{ }` is a block, a class body, or a lambda body. | CON | P7 | loud-ct | accepted | S41 [K] |
| GO-S1 | A composite literal whose type is a bare name cannot appear in the header of `if`, `for`, or `switch` without parentheses: `if y := S{5}; y.value != 5 {` is rejected because the literal's `{` is taken as the opening of the block, while `(S{5})` compiles. The specification resolves the ambiguity by requiring the parentheses. | CON | P7, P9 | loud-ct | accepted (known parsing ambiguity) | S38 [V] |
| RS-S1 | A struct literal cannot appear unparenthesized in the condition of `if`: the grammar excludes struct expressions there, because the `{` of the literal would be read as the start of the block. The same restriction applies to `while` and `match` scrutinees [K]. The Reference marks the exact extent of the exception as needing clarification. | CON | P7, P9 | loud-ct | accepted (parse ambiguity) | S39 [V]; other positions [K] |
| TS-S1 | After `=>`, a `{` starts a function body, not an object literal: `() => { foo: 1 }` is a block containing the label `foo:` and returns `undefined` with no error. The object must be parenthesized, `() => ({ foo: 1 })`. Other bodies of the same shape are syntax errors whose messages name neither blocks nor literals. A declared return type turns the silent case into a type error [K]. | CON | P7 | silent (or a misleading `SyntaxError`) | accepted (inherited from JavaScript) | S37 [V]; return-type remark [K] |
| TS-S2 | Automatic semicolon insertion makes a line break significant in some positions and not in others: `return` followed by a newline returns `undefined`, and the expression on the next line becomes unreachable code, while the same newline inside an expression changes nothing. | CON | P7 | silent | accepted (inherited from JavaScript; linters flag unreachable code) | S40 [K] |
| SW-S1 | `[ ]` is an empty array literal, `[:]` is the empty dictionary literal, `[k: v]` is a dictionary, and `[T]` and `[K: V]` are type sugar. The empty dictionary has its own spelling because `[]` is already the empty array. The ambiguity that PY-S1 leaves in place is removed here by an extra token, at the price of one more role for the bracket pair. | CON | P7 | loud-ct | resolved by a dedicated spelling | S24 [K] |
| SW-S2 | A trailing closure cannot be written unparenthesized in the condition of `if`, `guard`, or `while`: the brace would be ambiguous with the body, and the compiler asks for parentheses [K]. | CON | P7, P9 | loud-ct | accepted | S24 [K] |
| ZG-S1 | `{}` is an empty block and also a value of type `void` (`foo({})` passes it), while `.{}` is an anonymous literal that becomes a struct, a tuple, or an array according to the type the context expects. One spelling gives several results, chosen by type inference. | MOD | P7, P6 | loud-ct | accepted | S28 [K] |

**What the entries share.**

The entries differ along one axis: what decides the role. In GO-S1, RS-S1,
TS-S1, SW-S2, JV-S1 and ZG-S1 it is the position or the surrounding context (a
`{` that follows an expression in a header, follows `=>`, or follows a
declaration). In PY-S1, PY-S2 and SW-S1 it is the contents (an empty pair of
braces, a parenthesized expression with or without a comma). In TS-S2 it is a
line break.

The axis predicts visibility. Where the parser can see two valid readings at one
position, three languages independently refuse to choose and require
parentheses (GO-S1, RS-S1, SW-S2): the failure is loud at compile time and the
fix is local. The silent entries (TS-S1, PY-S2, TS-S2, and PY-S1 until the
first set method is called) are those in which both readings are valid
programs, so nothing refuses. This follows from the definition: an ambiguity
can be loud only if the parser rejects at least one of its readings at that
position.

The class is also measurable from the grammar, without readers. Language
grammars record their ambiguities as explicit exclusions: the Rust grammar for
conditions says `Expression` *except* struct expressions (S39), the Go
specification has a dedicated paragraph for composite literals in headers
(S38), and ECMAScript restricts what an expression statement may start with
[K]. Counting such exclusions, and counting the roles per symbol as in the
table above, needs only the grammar.

The catalog does not measure how often readers confuse brackets; in most
positions the grammar settles the role without effort. It records the
positions at which the grammar makes two roles possible.

Three ways out are visible in the entries, and they trade different things.
Give each role its own token (Swift's `[:]`, SW-S1): the price is one more
symbol to learn, and the token count grows with the number of roles. Refuse
the ambiguous form and require a marker at that position (Go, Rust, and Swift
with parentheses): the price is a parenthesized literal in headers and a
diagnostic that must name the cause. Never give one delimiter to two
constructs that can start at the same position (the P7 lever in §6): the price
is the shortcut. Which of the three fits depends on the rest of the grammar and
is not derivable from this catalog.

## 5. Recurring patterns

The entries repeat a small number of causes. Every one of the 12
patterns below occurs in at least three of the seven languages; this is
evidence that the causes are structural properties of language design, not
accidents of one language or one team.

| Pattern | Name | Entries (primary in bold) | Languages |
|---|---|---|:---:|
| P1 | Identity vs. value | **PY-6**, **JV-1**, **GO-1**, GO-8 | 3/7 |
| P2 | Aliasing of shared mutable state | PY-1, **PY-4**, **PY-8**, **GO-3**, GO-6, TS-2, **SW-4**, ZG-1, **ZG-2** | 5/7 |
| P3 | Binding time, scope, and lifetime rules that have no surface marker | **PY-1**, **PY-2**, **PY-3**, PY-8, **GO-2**, GO-5, **GO-6**, **GO-7**, **RS-2**, **RS-3**, RS-4, RS-5, **SW-6** | 4/7 |
| P4 | Partial operations with silent or build-dependent results | **JV-5**, **GO-8**, **RS-1**, **TS-3**, **ZG-4** | 5/7 |
| P5 | Absence represented in several ways | **PY-7**, **JV-3**, **GO-4**, **SW-5** | 4/7 |
| P6 | Static types vs. runtime values | **JV-2**, GO-1, **TS-1**, **TS-2**, TS-3, **TS-4**, **TS-6**, **ZG-6**, ZG-S1 | 4/7 |
| P7 | One form with several meanings (or several forms with one meaning) | **PY-5**, PY-7, **PY-9**, JV-1, **JV-6**, GO-4, **GO-5**, **RS-4**, **RS-7**, **TS-5**, TS-7, **SW-2**, SW-4, **PY-S1**, **PY-S2**, **RS-S1**, **GO-S1**, **TS-S1**, **TS-S2**, **SW-S1**, **SW-S2**, **JV-S1**, **ZG-S1** | 7/7 |
| P8 | Compatibility debt and churn | JV-2, **JV-7**, **TS-7**, **TS-8**, **ZG-7** | 3/7 |
| P9 | Feature interaction | **JV-4**, **JV-8**, **RS-6**, **RS-8**, **SW-3**, SW-7, RS-S1, GO-S1, SW-S2 | 4/7 |
| P10 | Specification gaps and implementation-dependent semantics | PY-6, RS-1, **ZG-1**, ZG-2, ZG-4, ZG-7 | 3/7 |
| P11 | Cost and resource model mismatch | GO-3, **SW-1**, SW-2, **SW-7**, ZG-5 | 3/7 |
| P12 | Convention mismatch with neighboring languages and tools | **RS-5**, TS-8, SW-1, **ZG-3**, **ZG-5** | 4/7 |

Two observations follow from the table.

First, the fixes that were shipped follow one mechanism. Four of them are
gated by an explicit version, edition, or flag: GO-2 by the `go` line of each
module, RS-2 and RS-3 by the Rust edition, and JV-1 by a preview flag. Gating
did not make migration free: for RS-2 and RS-3 the guide itself says that an
automatic rewrite may reintroduce the old behavior or does not exist in
general. A language therefore needs a versioning mechanism *before* it needs
its first semantic fix. [S10, S14, S15, S6]

Second, several entries are deliberate exchanges of predictability for another
goal: TS-1 for compatibility and convenience (a sound type system is a stated
non-goal [S18]), SW-1 and ZG-5 for visible cost, RS-5 for zero-cost futures.
Where a language makes the exchange knowingly, the surprise is documented and
the Status column says *accepted*. The harder cases are those in which nobody
chose the surprise: it appeared from the interaction of two sound decisions
(P9) or from a gap between specification and implementation (P10).

## 6. From causes to design levers

Each lever is derived from the cause of its pattern, not collected from
folklore: if the cause is *X*, the lever removes *X* or makes it visible.

| Pattern | Lever | Mechanism | Price | Evidence in the catalog |
|---|---|---|---|---|
| P1 | Identity is not observable on values | Value types have no identity; the only identity operation is explicit and exists only for reference types; each type has one meaning of `==` | Pointer-equality idioms and identity-keyed caches need explicit reference types | JV-1: JEP 401 does this for `Integer` [S6]; PY-6 and GO-1 are the same disease |
| P2 | Sharing is either visible or impossible | Immutable by default, value semantics (copy-on-write or persistent structures), or tracked ownership; mutation marked at the use site | Copying or persistence cost, or annotation burden | PY-1, PY-4, PY-8, GO-3 vanish when values cannot be mutated through a second name |
| P3 | One binding rule, evaluated where it is written | Fresh binding per iteration; defaults evaluated per call or immutable; explicit capture; scope introduced by a binding form, never by assignment; one temporary-lifetime rule independent of syntactic position | Slightly longer code at capture and declaration sites | GO-2, RS-2, RS-3 were fixed in exactly these directions |
| P4 | No operation has an unspecified or build-dependent result | Total operations, or an explicit failure value, or a defined trap; wrapping and saturating arithmetic as separately named operations; indexing returns an option or traps | Run-time checks; verbosity | RS-1, ZG-4, JV-5 differ only in which default they picked |
| P5 | Exactly one representation of absence | Option / sum types, no null, no overloaded falsiness, explicit unwrap | Interop boundaries need explicit conversion (SW-5 exists for interop) | JV-3, GO-4, PY-7 |
| P6 | Check where static meets dynamic | Casts and typed/untyped transitions are checked at run time (gradual typing with blame); no unchecked assertion in safe code; any retained unsoundness is named and enumerated | Run-time checks at boundaries | TS-1, TS-4; the Effective TypeScript list [S19] is such an enumeration |
| P7 | Orthogonality | One concept, one form; one form, one meaning; a context-free grammar; no symbol reused across roles unless the roles are mirror images | Fewer shortcuts; occasionally longer spellings | RS-7 exists because `<` has two roles; GO-5, SW-2; the `{`-as-literal-or-block family GO-S1, RS-S1, TS-S1, SW-S2 (§4.9) |
| P8 | Versioned semantics | Language version is a property of each module; semantic changes ship behind it with migration tooling; deprecation is a process, not an event | The compiler carries several semantics at once | GO-2, RS-2, RS-3, JV-1; ZG-7 shows the cost of having no process |
| P9 | Interaction matrix | Before a feature is accepted, its behavior with every existing feature (pairwise, plus selected triples) is written down; a pair with no uniform rule blocks the feature | Design effort grows quadratically in the number of features | JV-4, RS-6, SW-3, SW-7 |
| P10 | One normative semantics | Behavior independent of implementation, optimization level, and build mode; documentation validated against conformance tests; differential testing across implementations | Forgoes optimizations that rely on undefined behavior | ZG-1, RS-1, ZG-4 |
| P11 | Cost is part of the model | Cost visible in API shape or types (index types, explicit allocators); nothing that looks O(1) and is O(n) | Convenience | SW-1, ZG-5 choose this; GO-3 and SW-7 show the opposite |
| P12 | Choose the audience and force the explicit form at conflicts | State the target reader; where the priors of two populations conflict, refuse the shorthand and require a named operation; record the choice | Friction for the population whose prior lost | ZG-3, TS-8, RS-5 |

Process levers complement the language levers:

- **M1. Name the reference reader.** A1-style reader, stated once, so that
  EXP-class decisions are decisions rather than accidents.
- **M2. Measure instead of intuit.** User-centered methods exist for language
  design: PLIERS adapts need finding, low-fidelity prototyping, risk analysis,
  empirical refinement, and summative studies to languages [S31]. Predict-the-
  output questionnaires on small programs and mining of "why does this print"
  questions are cheap probes of EXP and MOD. Notional-machine research from
  computing education treats the programmer's model of the language as an
  object of study [S32].
- **M3. Make loud failures informative.** When a surprise cannot be removed,
  the error message should state the model, not only the symptom (SW-7 is the
  counter-example).
- **M4. Make hidden mechanisms visible in tooling.** Editor hints for inferred
  types, captures, conversions, and lifetimes move an implicit rule closer to
  the surface without changing the language.

## 7. Can the list reach zero?

**Short answer.** Not as an absolute. For two of the three classes the list can
be driven to zero under stated conditions; for the third it can only be
minimized and measured. The reasons follow from the structure of the problem.

**A cost model.** Treat the cost of one case as the product of four factors:
the probability that the construct is used, the probability that the reader
mispredicts it, the delay before the failure is noticed, and the damage done
meanwhile. This is an explanatory model, not a measured quantity; it is used
only to order the levers. Language levers act on different factors: removing
a construct acts on the first, aligning form and meaning acts on the second,
making failures loud and early acts on the third (a compile-time error is
noticed earliest, a run-time error later, a silent wrong result latest), and
immutability or isolation acts on the fourth.

**Obstacles to zero.**

- **O1. Conflicting priors.** Real reader populations hold contradictory
  expectations, so any total choice surprises some population: `/` on
  integers (truncating in C, true division in Python 3), `^` (exclusive-or in
  C-family languages, exponentiation in mathematical notation), the index base
  (0 in C-family languages, 1 in Lua, Julia, MATLAB, and Fortran), and equality
  of `NaN` (IEEE 754 against the expectation that equality is reflexive). [K]
  The only escapes are to choose a population (M1), to refuse the shorthand
  and require an explicit named form (P12), or to avoid the construct. This
  obstacle is a pigeonhole argument, so no design technique removes it.
- **O2. Undecidability.** By Rice's theorem every non-trivial semantic property
  of programs in a Turing-complete language is undecidable [S35], so no static
  analysis rules out all surprising dynamic outcomes (overflow, aliasing,
  non-termination) without either restricting the language (total languages,
  linear or refinement types) or accepting false rejections. A conservative
  analysis moves the surprise from "why did it run wrong" to "why will it not
  compile"; it does not delete it. What *can* be done is to turn silent
  outcomes into loud ones.
- **O3. Interaction growth.** With *n* features there are *n(n-1)/2* pairs and
  more triples, and each combination needs a defined behavior. Orthogonality
  keeps those behaviors uniform, but whether a uniform behavior *makes sense*
  to a reader is an empirical question: RS-6 combines async and drop, both
  principled, and still surprises.
- **O4. Goals that require leaks.** Performance, hardware fidelity, and FFI
  expose the machine: cost cliffs, resource exhaustion, scheduling
  nondeterminism, floating point. These cannot be abstracted away; they can
  only be made explicit (P11). Removing them changes the language's goal (for
  example, arbitrary-precision integers by default), so "elimination" in this
  case means choosing different goals.
- **O5. Drift.** Expectations change with experience and with the ecosystem
  [S29, S30]: an expert is not astonished by PY-1, a novice is. Astonishment
  is a distribution over a population, not a boolean.

**What is achievable.**

| Class | Zero reachable? | Condition | Irreducible remainder |
|---|---|---|---|
| CON | Yes, relative to the written specification | The surface language is defined by total desugaring into a small core; each concept has one mechanism; the pairwise interaction matrix is complete; properties such as "any expression of type T is accepted wherever T is expected" are checked mechanically | Errors in the specification itself; the reader's perception of what counts as "the same" |
| MOD | Yes for semantic leaks; no for physical leaks | The observable semantics are complete and independent of implementation, optimization level, and build mode; values have no observable identity; sharing is visible | Cost cliffs, resource exhaustion, scheduling nondeterminism, floating point (O4); these can only be made explicit |
| EXP | No | O1 guarantees that some population's prior loses | A residual that can be measured (M2) and minimized by choosing the reference reader (M1) and forcing the explicit form at conflicts (P12) |

Two corollaries. First, the target metric should change from "number of
surprises" to "cost of surprises" (the model above): a language can drive the
number of *silent* surprises toward zero while loud ones remain. Second, a
language that reaches zero for CON and for the semantic part of MOD still has
a nonzero EXP list, so the honest claim is "zero by construction for what the
language controls, minimized and measured for what the reader brings".

## 8. What is derivable and what is not

**Derivable from the structure above:**

- The lever for each pattern, given its cause (§6).
- Zero internal inconsistency relative to a specification, given a small core
  and a complete interaction matrix (§7).
- The ordering of failures by cost: silent is worse than run-time error, which
  is worse than compile-time error.
- The impossibility of zero EXP surprises (O1) and the need to choose a
  reference reader.

**Not derivable; needs data or a value judgment:**

- Who the reference reader is, and what that reader's priors are.
- The weights among goals that conflict with predictability (speed, brevity,
  compatibility, low-level control).
- Which of two conflicting priors to follow (for example 0-based or 1-based
  indexing).
- The acceptable residual (an "astonishment budget").
- Whether a particular construct will be noticed and mispredicted; this is an
  empirical question (M2).
- How expectations will drift as the language and its ecosystem change.

### 8.1 Measuring what remains

Section 7 moves the target from a count of surprises to their cost. A scheme
for measuring that cost has three tiers and one counterweight. This is a
proposal: nothing here has been run on Orthon, and the Orthon fitness
functions in [`FITNESS_FUNCTIONS.md`](../how/architecture/FITNESS_FUNCTIONS.md)
do not contain it.

- **Tier 1, laws.** Forms that the reader treats as equal must be equal:
  rewrite equivalence (`x op= y` means `x = x op y`), values versus places
  versus identity, totality of operations, invariance to build mode and
  strategy, soundness of types, scope. The score is the number of
  (law, construct) pairs that are violated *or not specified*; the target is
  zero. Counting an unwritten law as a violation is a policy, not a derivation.
- **Tier 2, structural indices** computed from the specification and the
  grammar: roles per symbol (§4.9), exclusions in the grammar, mechanisms per
  concept, ways to express absence, and interaction coverage (pairs with a
  stated rule over *n(n-1)/2*). The target is no worse than the previous
  revision, with polysemy driven to zero except for mirror pairs.
- **Tier 3, empirical.** The Surprise Rate is the share of wrong predictions of
  program behavior by the reference reader (A1, M1), compared with the same
  measure on neighboring languages for the same constructs. It needs readers;
  a language model is a cheap proxy only for the corpus it was trained on (C5).
- **Counterweight.** Coverage of the target scenarios, as a constraint: a
  language with fewer constructs trivially scores better on every tier above.

Whether syntax and semantics need different metrics depends on the tier. The
instruments differ at tiers 1 and 2; the outcome measure at tier 3 and the
counterweight are shared.

| | Syntax | Semantics |
|---|---|---|
| Instrument | Grammar: roles per symbol, lookahead exclusions, dedicated tokens | Stated laws, completeness of desugaring, interaction matrix |
| Why | Decidable from the grammar | Non-trivial properties are undecidable (O2), so laws and tests stand in |
| Typical visibility | Loud at compile time, unless both readings are valid programs (§4.9) | Mostly silent or loud at run time |
| Main cost | Learning and reading (a hypothesis, not measured) | Late detection and damage at run time (a hypothesis, not measured) |
| Goodhart risk | More tokens and lost brevity | A complete specification that nobody can read |

The two meet at desugaring. `x op= y`, `() => { foo: 1 }`, and an empty `{}`
are neither purely syntax nor purely semantics: each is a surface form whose
meaning is defined by a rewrite. The rewrite-equivalence laws of tier 1 live
there, which is why the specification of every surface form should include its
total desugaring.

Not derivable from this structure: the weight of syntax against semantics in a
single score (it depends on the reference reader), the thresholds, and the size
of the reader panel.

## 9. Observations for Orthon

*These are observations, not decisions. Orthon has no compiler yet, so the
mapping below describes documents, not behavior.*

| Pattern | Orthon artifact as documented |
|---|---|
| P1 | Three equality operators `===`, `==`, `is` ([`EQUALITY.md`](../what/concepts/EQUALITY.md), [`equality-operator-comparison.md`](equality-operator-comparison.md)); "Deterministic Behavior" in [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) names object interning as something program behavior must not depend on |
| P2 | "Data Is Immutable by Default" ([`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md)); [`COPY_ON_WRITE.md`](../what/concepts/COPY_ON_WRITE.md) |
| P3 | Explicit closure capture ([`CLOSURE_CAPTURE.md`](../how/concepts/research/essential/CLOSURE_CAPTURE.md)); "no accidental creation" and explicit shadowing ([`DECLARATION_BY_ASSIGNMENT.md`](../what/concepts/DECLARATION_BY_ASSIGNMENT.md)) |
| P4 | No dedicated arithmetic-overflow or indexing-failure rule was found by keyword search in `what/`, `how/concepts/`, `how/architecture/`, and `notes/`; hits were unrelated |
| P5 | No `null`, `Option<T>` ([`NULL_SAFETY.md`](../what/concepts/NULL_SAFETY.md), EDR-018); `NaN` equality deferred to the standard library to protect transitivity ([`EQUALITY.md`](../what/concepts/EQUALITY.md)) |
| P6 | Boundary checks at typed/untyped transitions ([`GRADUAL_TYPING.md`](../what/concepts/GRADUAL_TYPING.md), EDR-059) |
| P7 | "One concept — one syntax" ([`MANIFESTO.md`](../why/MANIFESTO.md)); Semantic Purity and POLA ([`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md)); bracket roles in the candidate collection-literal syntax and in generics syntax are compared in §4.9 (see C7, C8) |
| P8 | [`EVOLUTION_MODEL.md`](../how/EVOLUTION_MODEL.md) (versioning, deprecation, feature gates) |
| P9 | [`CROSS_CUTTING.md`](../what/CROSS_CUTTING.md) is a placeholder; `LOGICAL_CONSISTENCY_GATE` in [`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md) |
| P10 | "Semantics Before Optimization" and "Deterministic Behavior" ([`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md)); [`OPTIMIZATION_MODEL.md`](../what/OPTIMIZATION_MODEL.md) |
| P11 | Explicitness requires allocation and ownership transfer to be visible in syntax ([`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md)); no general cost-model rule located |
| P12 | Conscious audience choices: 1-based indexing ([`INDEXING.md`](../what/concepts/INDEXING.md), EDR-082) and the `NaN` deferral above |

Checks that follow from the catalog:

- **C1. Default-value evaluation.** The draft [`DEFAULT_VALUES.md`](../how/concepts/research/important/DEFAULT_VALUES.md)
  lists an Evaluation Policy that controls whether a default is evaluated
  once (shared) or per call (copied). With immutable data the difference is
  unobservable, but if a default can be `mut` it would recreate PY-1, and a
  Policy whose choice changes observable behavior would conflict with
  "Semantics Before Optimization". An explicit invariant ("the choice of
  policy must not be observable") would close the gap.
- **C2. Numeric semantics.** P4 is the pattern with the most divergent
  defaults (RS-1, ZG-4, JV-5). If the numeric model is still to be written, the
  three defaults in the catalog form a ready-made table of alternatives, and
  the requirement that meaning not depend on build mode or Strategy is
  already implied by P10 and by Deterministic Behavior.
- **C3. Interaction matrix as the P9 lever.** [`CROSS_CUTTING.md`](../what/CROSS_CUTTING.md)
  is empty. The catalog suggests concrete pairs to test first, since each
  mirrors a documented failure: error handling with generators and with
  async/await (JV-4, RS-6), and ownership or mutation with async (SW-3).
- **C4. Breadth of the POLA principle.** POLA in
  [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) is stated for operator
  meaning. The twelve patterns could serve as a review checklist for the
  `LOGICAL_CONSISTENCY_GATE`; this would be a process change and would need an
  EDR.
- **C5. Measuring EXP.** To the extent the
  [LLM Generability Gate](llm-generability-gate.md) uses model predictions as a
  proxy for reader expectations, its reference population is the training
  corpus. For deliberate departures from the mainstream prior (1-based
  indexing, `NaN` deferral) it will report astonishment by construction, so
  its results on EXP-class constructs need that reading. Human-subject methods
  [S31] are not present in the repository.
- **C6. Audience statement.** The EXP-class decisions are currently recorded
  one by one in EDRs. A single stated reference reader (M1) would let them be
  checked against one criterion instead of being justified case by case.
- **C7. Empty braces.** [`COLLECTION_LITERAL_SYNTAX.md`](../what/concepts/COLLECTION_LITERAL_SYNTAX.md)
  lists the candidate forms `{key: value}` for maps and `{a, b}` for sets,
  and names scope blocks as the reason `{}` needs disambiguation (open
  question 1: empty map or empty scope block). That is the clash recorded in
  PY-S1 (empty braces and sets), and it sits where TS-S1, GO-S1, and RS-S1 sit
  (a `{` that may start a literal or a block). The three ways out in §4.9 (own
  token, required marker, no shared delimiter) are the axes along which the
  open question can be answered. Which block syntax Orthon finally uses is not
  settled in the documents I read:
  [`INVOCATION_SYNTAX.md`](../what/syntax/INVOCATION_SYNTAX.md) shows
  colon-and-indentation blocks (`using file = open("data.txt"):`), while the
  open question above speaks of scope `{}` blocks.
- **C8. A stale statement.** The candidate-syntax table in
  [`COLLECTION_LITERAL_SYNTAX.md`](../what/concepts/COLLECTION_LITERAL_SYNTAX.md)
  describes `[expr, ...]` as "Unambiguous — brackets not used elsewhere".
  [`GENERICS_SYNTAX.md`](../what/syntax/GENERICS_SYNTAX.md) (EDR-086) says that
  `[ ]` is freed for indexing, array types, and slices, so the pair has at
  least four roles in the documents as they stand (§4.9, Orthon row). The
  statement is likely to predate EDR-086; whether the roles stay distinguishable
  by position is the same question as C7.

## 10. Sources

`[V]` fetched and read for this note; `[K]` cited from established knowledge,
not fetched (see A6).

**Python**

- S1 [V] Python Programming FAQ (closures in loops; `UnboundLocalError`; list
  repetition; augmented assignment on tuples).
  https://docs.python.org/3/faq/programming.html (read via the mirror
  https://documentation.help/Python-3.4/programming.html)
- S2 [K] Python Design FAQ, "Why are default values shared between objects?".
  https://docs.python.org/3/faq/design.html
- S3 [K] Python Language Reference: data model; truth value testing.
  https://docs.python.org/3/reference/datamodel.html
- S4 [K] R. Hettinger, "Transforming Code into Beautiful, Idiomatic Python"
  (PyCon talk), on `for`/`else`.

**Java**

- S5 [V] OpenJDK discuss, "Should JLS 5.1.7 require that boxing uses
  `valueOf`?" (March 2024).
  https://mail.openjdk.org/pipermail/discuss/2024-March/006342.html
- S6 [V] Inside Java, "Try JEP 401 value classes".
  https://inside.java/2025/10/27/try-jep-401-value-classes/
- S7 [K] J. Bloch, *Effective Java* (3rd ed.), Items 13, 19, 28; Bloch and
  Gafter, *Java Puzzlers*.
- S8 [K] Java Language Specification, §15.12 (method invocation) and §15.18.2
  (additive operators).
- S9 [K] T. Hoare, "Null References: The Billion Dollar Mistake" (QCon, 2009).

**Go**

- S10 [V] Go Blog, "Fixing For Loops in Go 1.22". https://go.dev/blog/loopvar-preview
- S11 [V] Go Blog, "Go Slices: usage and internals" (read via
  https://go.googlesource.com/blog/+/master%5E/_content/slices-intro.article)
- S12 [V] yourbasic.org, "Nil is not nil".
  https://yourbasic.org/golang/gotcha-why-nil-error-not-equal-nil/
- S13 [K] The Go Programming Language Specification; Go FAQ.

**Rust**

- S14 [V] Rust Edition Guide, "if let temporary scope".
  https://doc.rust-lang.org/edition-guide/rust-2024/temporary-if-let-scope.html
- S15 [V] Rust Edition Guide, "Tail expression temporary scope".
  https://doc.rust-lang.org/edition-guide/rust-2024/temporary-tail-expr-scope.html
- S16 [V] The Rust Reference, "Behavior not considered unsafe" (integer
  overflow). https://doc.rust-lang.org/reference/behavior-not-considered-unsafe.html
- S17 [K] Tokio documentation on `select!` cancellation safety; The Rust
  Reference on generic arguments; RFC 1023 (rebalancing coherence).

**TypeScript**

- S18 [V] TypeScript Design Goals.
  https://github.com/microsoft/TypeScript/wiki/TypeScript-Design-Goals
- S19 [V] D. Vanderkam, "The Seven Sources of Unsoundness in TypeScript"
  (Effective TypeScript, 2021). https://effectivetypescript.com/2021/05/06/unsoundness
- S20 [K] TypeScript Handbook and FAQ (excess property checks; `Object.keys`).

**Swift**

- S21 [V] SE-0335, "Introduce existential `any`".
  https://github.com/apple/swift-evolution/blob/main/proposals/0335-existential-any.md
- S22 [V] Swift Forums, "Has Swift's concurrency model gone too far?".
  https://forums.swift.org/t/has-swifts-concurrency-model-gone-too-far/77468
- S23 [V] Swift Forums, "Why are String offsets so complicated?".
  https://forums.swift.org/t/why-are-string-offsets-so-complicated/19753
- S24 [K] The Swift Programming Language (Properties; Optionals).

**Zig**

- S25 [V] Ziggit, "Pass by value semantics".
  https://ziggit.dev/t/pass-by-value-semantics/15710
- S26 [V] Ziggit, "Can someone compile a fork of Zig to turn errors for unused
  things into warnings?"
  https://ziggit.dev/t/can-someone-compile-a-fork-of-zig-to-turn-errors-for-unused-things-into-warnings/11422
- S27 [V] DevClass, "Zig lead makes extremely breaking change to std io ahead
  of async and await's return" (2025-07-07).
  https://devclass.com/2025/07/07/zig-lead-makes-extremely-breaking-change-to-std-io-ahead-of-async-and-awaits-return/
- S28 [K] The Zig Language Reference.

**Syntax and notation**

- S36 [V] The Python Language Reference, "Expressions" (parenthesized forms,
  set and dictionary displays; read via the mirror
  https://python-documentation-cn.readthedocs.io/en/latest/reference/expressions.html,
  whose Python version I did not determine).
- S37 [V] MDN, "Arrow function expressions" (returning object literals).
  https://developer.mozilla.org/docs/Web/JavaScript/Reference/arrow_functions
- S38 [V] Go issue 35932, a closed report of the composite-literal parsing
  ambiguity that quotes the specification.
  https://github.com/golang/go/issues/35932
- S39 [V] The Rust Reference, "if expressions" (conditions exclude struct
  expressions; the page carries a TODO about the exception).
  https://rust.googlesource.com/rust-lang/reference/+show/refs/heads/master/src/expressions/if-expr.md
- S40 [K] ECMAScript Language Specification, "Automatic Semicolon Insertion" and
  the lookahead restriction on expression statements.
- S41 [K] Java Language Specification, §10.6 (array initializers).

**Theory and method**

- S29 [V] Wikipedia, "Principle of least astonishment".
  https://en.wikipedia.org/wiki/Principle_of_least_astonishment (history,
  Cowlishaw's corollary, audience dependence)
- S30 [V] "The Philosophy of Ruby", interview with Yukihiro Matsumoto, Artima.
  https://www.artima.com/articles/the-philosophy-of-ruby
- S31 [V] M. Coblenz et al., "PLIERS: A Process that Integrates User-Centered
  Methods into Programming Language Design". https://arxiv.org/abs/1912.04719
- S32 [V, abstract only] Fincher et al., "Notional Machines in Computing
  Education: The Education of Attention" (ITiCSE 2020 working group report).
  https://luce.si.usi.ch/publications/2020-06-01-notional-machines.html
- S33 [K] A. Stefik and S. Siebert, "An Empirical Investigation into
  Programming Language Syntax" (ACM TOCE, 2013).
- S34 [K] F. Brooks, *The Mythical Man-Month* (1975), on conceptual integrity.
- S35 [K] H. G. Rice (1953), on non-trivial semantic properties of programs;
  J. Siek and W. Taha (2006), on gradual typing.
