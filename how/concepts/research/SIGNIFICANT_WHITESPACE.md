# Significant Whitespace

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ LLM generability note:** Significant whitespace introduces unique challenges for
> LLM-based code generation — indentation errors, inconsistent whitespace, and
> copy-paste corruption are common failure modes. See the LLM Generability Gate
> in `how/gates/DECISION_VALIDATION.md` and `how/strategies/LLM_STRATEGY.md` for
> evaluation criteria.

## Issue (Why)

How does the language delimit blocks of code — groups of statements that belong to the same structural unit (function body, loop body, conditional branch)?

Two fundamentally different approaches dominate:

1. **Explicit delimiters** (Java, C, JavaScript, Go) — blocks are enclosed in curly braces `{ ... }`. Statements end with semicolons `;`. Whitespace is ignored. The structure is unambiguous regardless of formatting: the delimiters are the source of truth, not the indentation.

2. **Significant whitespace** (Python, Haskell, YAML) — block structure is determined by indentation level. The start of a block is signaled by a colon `:` or similar marker; the end is signaled by a return to the previous indentation level. No braces. No explicit block-end marker.

The core problem: **should the language rely on whitespace for structural information, or should it use explicit delimiters?** The choice affects readability, toolability, error messages, LLM generability, and the learning curve for new programmers.

Significant whitespace offers visual cleanliness — code looks structured because the visual indentation *is* the structure. But it introduces ambiguities: a single space or tab character misplaced can change the meaning of a program, and the compiler's error message ("unexpected indent") is less helpful than "missing closing brace at line 42."

## Principles

1. **Structure must be unambiguous** — The block structure of a program must be determinable from syntax alone. Indentation is the source of truth, not a suggestion.
2. **Consistent indentation rules** — Either tabs or spaces, not both. The compiler rejects files with mixed indentation. The standard is spaces (4-space indentation).
3. **No implicit block continuation** — Line breaks inside expressions do not implicitly end the statement. Delimiters (`()`, `[]`, `{}`) control continuation, not indentation.
4. **LLM generability** — The whitespace rules must be simple enough that LLMs produce correct indentation reliably. Complex indentation rules (like significant whitespace inside brackets) hurt generation quality.
5. **Explicit block-end where helpful** — Long blocks should optionally support block-end markers with the block name (`end function`, `end for`) to aid readability and reduce indentation tracking errors.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Block Delimiter Policy | Determines whether blocks are delimited by indentation, braces, or keywords |
| Statement Termination Policy | Governs whether statements end at newlines, semicolons, or both |
| Indentation Policy | Specifies indentation rules (tabs vs spaces, width, continuation) |
| LLM Generability Policy | Evaluates whether the whitespace rules are LLM-generable without special tooling |

## Model (What)

Blocks are defined by indentation. A block starts after a colon `:` and continues at the same or greater indentation level. Returning to a lower indentation level ends the block. Statements end at newlines. Semicolons are available for multiple statements on one line but are not idiomatic.

```
fun greet(name: String) -> String:    # colon starts the block
    greeting = "Hello, " + name       # indented: inside the function
    return greeting                   # still inside

# Back to top level — the function block ended
value = compute()                     # top-level statement

# Nested blocks
for item in items:                    # colon starts the block
    if is_valid(item):                # nested block starts
        process(item)
        count += 1
    # end of if block (back to for block)
# end of for block (back to top level)

# Long blocks can use optional block-end markers
fun long_function() -> Int:
    # ... many lines ...
end fun
```

Key features:
- **`:` starts a block** — after a function signature, loop header, conditional, etc.
- **Indentation determines block membership** — all lines at the same indentation belong to the same block.
- **Statement ends at newline** — no semicolons needed. Line continuation is explicit (backslash `\` or wrapping in delimiters).
- **Optional `end` markers** — `end fun`, `end for`, `end if`, `end match` for long blocks where the end is visually far from the start.
- **Mixed indentation is a compile error** — the compiler rejects files with mixed tabs and spaces. Convention: 4 spaces.
- **Blank lines are allowed** — blank lines at any indentation do not change block membership.

### Comparison with Java and Python

| Aspect | Java | Python | Orthon (proposed) |
|---|---|---|---|
| Block delimiters | `{}` | Indentation (after `:`) | Indentation (after `:`) |
| Statement termination | `;` (required) | Newline (default) | Newline (default) |
| Semicolons | Required | Optional (not idiomatic) | Optional (not idiomatic) |
| Block-end marker | `}` | Nothing (dedent) | Optional `end name` |
| Mixed tabs/spaces | Allowed | Error (PEP 8) | Error (compiler) |
| LLM indentation errors | None (braces) | Common | Mitigated by `end` markers |

## Default Strategy

Indentation-based blocks with colon-start and optional `end` markers. The compiler normalizes indentation to 4-space equivalents. Mixed indentation is rejected at parse time. `end` markers are optional but recommended for blocks exceeding a configurable line count (e.g., 20 lines).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Explicit braces | Blocks delimited by `{}` (Java, C, Go, Rust). No indentation errors. More visual noise. |
| Explicit keywords | Blocks delimited by keywords like `do...end`, `begin...end` (Ruby, Lua, Pascal). Clear, no indentation sensitivity, but verbose. |
| Significant indentation without colon | No `:` marker — indentation alone determines block boundaries (Haskell, YAML). Clean but ambiguous without preceding context. |
| Indentation + mandatory `end` | Indentation determines structure, but every block must close with `end`. Redundant but catches indentation errors. |

## Open Questions

1. Should the `end fun` / `end for` marker be syntactically required or purely optional?
2. How does significant whitespace interact with the Execution Program model — can Execution Programs have different whitespace rules?
3. Should the compiler auto-indent on output (error messages, AST dump) to a standard format regardless of source indentation?
4. How does significant whitespace interact with copy-paste from web pages, LLM outputs, and email — common sources of indentation corruption?
5. Should there be a language-level auto-formatter (like `gofmt` or `ruff format`) to eliminate indentation debates?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/gates/DECISION_VALIDATION.md`
- [ ] `how/strategies/LLM_STRATEGY.md`
- [ ] `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- [ ] Other: `how/architecture/PARSER.md`
