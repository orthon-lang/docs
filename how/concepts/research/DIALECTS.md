# Dialects

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

Every programming language imposes a fixed surface syntax on its users.
Developers coming from other ecosystems must learn a new set of keywords
and punctuation, even when the underlying concepts are familiar. The core
problem: **how can Orthon accommodate multiple surface syntaxes without
fragmenting the language's semantics, tooling, and canonical
representation?**

Languages that offer only one syntax create friction for newcomers,
complicate migration from existing ecosystems, and limit the use of the
language in domain-specific or educational contexts. Yet languages that
allow unbounded syntax customization (C++ templates, Scala implicits,
Lisp macros) risk creating fragmented dialects that are mutually
incomprehensible.

Orthon's approach is to define a single canonical language internally
while allowing projects to choose alternative keyword mappings at the
lexical level. Dialects change how code is written, not what it means.

## Principles

1. **Single canonical language** — Orthon defines exactly one official
   syntax. Documentation, compiler diagnostics, formatter output,
   examples, and generated code always use the canonical keywords.
2. **Dialects are lexical only** — A dialect changes keyword mappings
   during lexical analysis. It does not change grammar, semantics, AST,
   type system, or runtime behavior.
3. **Profiles over ad-hoc aliases** — Official dialect profiles
   (Python, Rust, JavaScript, Educational) are preferred over arbitrary
   custom mappings. Custom dialects are permitted but restricted to
   keyword substitution.
4. **Preserved LLM representation** — LLMs benefit from a stable
   canonical representation: training data becomes more consistent,
   code search and retrieval improve, generated code follows one
   standard style, and reasoning operates on a single language rather
   than many surface variants.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Lexer Policy | Defines how keyword translation is applied during lexical analysis |
| Keyword Resolution Policy | Governs which identifiers are reserved and how aliases interact with keyword lookup |
| Diagnostic Policy | Controls whether compiler diagnostics display in canonical or dialect keywords |

## Model (What)

Dialects allow projects to customize the surface syntax of Orthon while
preserving a single canonical language internally.

A dialect maps alternative keywords to the canonical Orthon syntax
during lexical analysis. The compiler, parser, formatter, AST, and
tooling always operate on the canonical representation.

```orthon
[dialect]
profile = "custom"

[keywords]
fn = "def"
fun = "def"
```

Both forms are equivalent — they produce the same AST:

```orthon
def add(a, b) {
    return a + b
}
```

```orthon
fn add(a, b) {
    return a + b
}
```

### Profiles

Orthon may provide official dialect profiles for common language
backgrounds:

| Profile | Example mapping |
|---|---|
| Canonical | `def`, `if`, `else`, `return` |
| Python | `def` → `def` (same), `elif` → `else if` |
| Rust | `fn` → `def`, `match` → `match`, `let` → `let` |
| JavaScript | `function` → `def`, `const` → `let` (immutable), `=>` → `fn` shorthand |
| Educational (Russian) | `функция` → `def`, `если` → `if`, `иначе` → `else` |
| Educational (Spanish) | `funcion` → `def`, `si` → `if`, `sino` → `else` |

```orthon
[dialect]
profile = "rust"
```

Result: `fn` is accepted as an alias for `def`, `match` maps to `match`.

### Custom Dialects

Projects may define custom keyword mappings for internal DSLs:

```orthon
[keywords]
task = "def"
workflow = "module"
publish = "export"
```

### Compiler Pipeline

Dialect translation happens before parsing:

```
Source
    ↓
Lexer
    ↓
Dialect Translation (keyword mapping)
    ↓
Canonical Tokens
    ↓
Parser
    ↓
AST
```

All later compiler stages — semantic analysis, type checking,
optimization, code generation — operate only on canonical tokens.

### Reserved Names

Every canonical keyword and every active dialect alias becomes a
reserved identifier in the scope where the dialect is active.

Invalid when `fn = "def"` is active:

```orthon
let fn = 10   # Error: 'fn' is a reserved alias
```

### Advantages

- Familiar syntax for developers from other ecosystems — reduces
  onboarding friction.
- Easier migration from existing languages — teams can adopt Orthon
  while keeping a familiar keyword set.
- Better support for internal DSLs — small custom keyword sets make
  domain-specific code more readable.
- Educational localization — non-English keywords lower the barrier
  for learners in regions where English is not the primary language.
- Single compiler implementation — dialect translation is a thin
  lexer-level layer; no grammar, AST, or backend changes needed.
- Single AST, formatter, and LLM representation — tools always
  work with the canonical language regardless of source dialect.

### Disadvantages

- Reduced consistency across projects — developers reading an
  unfamiliar dialect project must mentally translate keywords.
- More complex IDE support — syntax highlighting, autocomplete,
  and diagnostics must respect the active dialect configuration.
- Harder to read unfamiliar projects — a Rust-profile project
  and a Python-profile project look like different languages.
- Community fragmentation risk — if unrestricted, groups may adopt
  divergent dialects that harm cross-project comprehension.

## Default Strategy

The default dialect is **Canonical** — Orthon's native keyword set.
No dialect configuration file is needed; the compiler assumes canonical
keywords unless a `[dialect]` section is present in the project
configuration.

```orthon
# No dialect config → canonical keywords
def greet(name) {
    return "Hello, " + name
}
```

## Alternative Strategies

### Official Profile

The project chooses one of the official dialect profiles. The compiler
loads the predefined keyword mapping from its built-in profile registry.

```orthon
[dialect]
profile = "python"
```

Use when: the team has a strong background from a specific language
and wants to minimize cognitive overhead during adoption.

### Custom Keyword Mapping

The project defines its own keyword substitutions for DSL-specific
terminology.

```orthon
[keywords]
task = "def"
emit = "return"
```

Use when: building a domain-specific language on top of Orthon where
domain vocabulary differs from programming-language keywords.

### No Dialect (Pure Canonical)

The project uses only canonical Orthon keywords. No dialect
configuration is present or required.

Use when: the team is new to Orthon and learning the canonical syntax,
or when cross-project consistency is the top priority.

## Open Questions

1. **Community fragmentation** — Should Orthon place limits on custom
   dialects (e.g., only keyword substitution, no grammar changes)?
   How should the project governance handle dialect proliferation?

2. **IDE support complexity** — How should editors discover the active
   dialect configuration? Through a project file, a comment pragma,
   or a file extension?

3. **Diagnostic language** — Should compiler errors and warnings display
   using the source dialect or the canonical keywords? A hybrid approach
   (canonical keywords in error codes, dialect in source snippets) may
   be most useful.

4. **Dialect vs. file scope** — Is dialect a per-project, per-module, or
   per-file setting? Per-project seems simplest, but DSL use cases may
   require per-file granularity.

5. **Migration path** — If a team starts with a non-canonical profile
   and later switches to canonical, should the compiler offer an
   automatic migration tool to rewrite sources?

## Decision History

This is the first research document for the Dialects concept. No
decisions have been recorded yet. Consequential design choices will
be recorded as EDRs in `how/decision_records/architecture/` when this
concept advances past the Concept Design Review.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/architecture/PARSER.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
- [ ] Other: `how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`
