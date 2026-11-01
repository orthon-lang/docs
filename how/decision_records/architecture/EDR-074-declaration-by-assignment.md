# EDR-074: Declaration by Assignment — Variable Introduction via First Assignment

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

How does a variable come into existence? Two approaches: explicit declaration (declare then assign) or declaration by assignment (first assignment creates the variable). Orthon balances conciseness with safety through first-assignment-is-declaration combined with compile-time definite assignment analysis.

The semantic decisions (how variables are created, how type is determined, what constitutes a read-before-write error) are language-level. The concrete syntax (keyword choice, annotation style) belongs to Phase 5 (Syntax).

---

### Decision

Declaration by Assignment is a **Language** construct, processed here with a clear Phase 5 boundary:

**Semantic decisions (this EDR):**
- First assignment is declaration — no separate declaration keyword.
- Assigning to an undeclared variable is a compile-time error.
- Type is inferred from the initializer; explicit annotations available.
- Read before write is a compile-time error (definite assignment analysis).
- Variables are immutable by default (`mut` required for reassignment).
- Shadowing permitted with explicit re-declaration keyword.

**Deferred to Phase 5 (Syntax):**
- Concrete syntax for type annotations (`: Type` vs `as Type`).
- Exact keyword for shadowing (`let` vs `var`).
- Whether `mut` is a keyword, modifier, or annotation.
- Concrete syntax for `const` (compile-time constants).

---

### Proposed Phase 5 Amendment: `:=` as an Ownership-Transfer Assignment

**Status:** Proposed — not accepted; requires Phase 5 review and human sign-off.

This proposal reuses `:=` as an infix marker for moving ownership from an
existing named binding into a destination binding:

```orthon
destination := source
```

The intended semantics are that `destination` receives the value owned by
`source`, and `source` becomes unusable after the transfer. The operator would
therefore make the ownership change visible at its site, as required by the
semantic model, while giving declaration-by-assignment's ordinary `=` a
separate role. `:=` would not mean mutable reassignment or a general-purpose
declaration operator.

The proposal is attractive because it combines assignment and transfer in one
visible form and avoids adding a `move` keyword or prefix marker to this
variable-to-variable case. It also has costs that Phase 5 must resolve:

- The `:=` spelling is used by other languages for declaration or assignment,
  so its move meaning may surprise readers and generated code.
- It covers a destination assignment, but does not directly express transfer
  into a consuming function argument, a return, or another expression context.
- The grammar and type rules must distinguish a new destination from an
  already-declared mutable destination, and define whether a moved-from name
  may be rebound.
- It must compose with explicit type annotations, destructuring, shadowing,
  and partial moves without creating context-dependent meanings.

**Alternatives to compare:** a `move source` expression, a `$source` prefix
operator, and an ownership metaproperty. These candidates should be compared
against the same ownership semantics and all consuming contexts; this proposal
does not establish that `:=` is the sole or canonical transfer syntax.

**Gate criteria for acceptance:**

- [ ] Define the grammar and typing rules for `destination := source`.
- [ ] Specify source invalidation, destination binding/reassignment, and
      moved-from rebinding behavior.
- [ ] Show equivalent ownership-transfer forms for function arguments,
      returns, and other consuming contexts, or explicitly limit the operator's
      scope.
- [ ] Check interactions with annotations, destructuring, shadowing, and
      partial moves.
- [ ] Compare `:=` with `move`, `$`, and `@ownership` for least astonishment,
      explicitness, and LLM generability.
- [ ] Validate against the language-design gate and record human sign-off
      before accepting the amendment.

Until those criteria are resolved, the accepted declaration-by-assignment
semantics above remain unchanged, and `:=` is only a syntax hypothesis.

---

### Consequences

- **Positive:**
  - Concise — no `let`/`var` for the common case (first assignment).
  - Safe — definite assignment analysis catches read-before-write errors.
  - Immutable by default — mutation requires explicit `mut`.
  - Shadowing visible — requires an explicit keyword.
- **Negative:**
  - Definite assignment analysis adds compiler complexity.
  - Phase 5 must resolve concrete syntax — risk of design re-opening.

---

### Compliance

- The compiler must implement definite assignment analysis.
- `mut` must be required for variable reassignment.
- Shadowing must require an explicit keyword.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Keyword declaration (Rust `let`) | Adds ceremony for every variable — redundant with first-assignment pattern. |
| Mandatory type annotation (Java) | Verbose — contradicts type inference design (EDR-027). |
| No shadowing | Forces awkward naming (`count`, `count_2`, `count_final`) — poorer code quality. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Write `x = 1` and it works — minimum ceremony. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Definite assignment analysis is well-defined — no ambiguity. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One rule — first assignment declares — covers all cases. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with type inference (EDR-027) and immutability-by-default. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Declaration semantics independent of concrete syntax. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Proven from Python and Julia — well-understood semantics. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs already generate `x = 1` patterns — no new syntax to learn. |
