# Literate Programming

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).

## Issue (Why)

How do you keep documentation synchronised with code?

Traditional approaches:
- **Comments in code** — inline explanation, but no structure; easy to forget to update; no executable examples.
- **Separate documentation** (Doxygen, Sphinx, TeX) — always out of date; examples are not tested; the "what" and "why" drift from the implementation.

The core problem: **code and its explanation live in separate files** with no synchronisation mechanism. Any change to one creates a lie in the other.

## Principles

1. **Single source of truth** — Code and documentation coexist in the same file; extraction tools produce publishable docs and compilable code.
2. **Executable documentation** — Examples in documentation must be runnable and tested by CI.
3. **Ordre naturel** — The document is organised for human reading (explanation first, code interleaved) rather than compiler-required order.
4. **Tangling and weaving** — Tools extract (tangle) code for compilation and weave text for publication.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Documentation Policy | Governs whether literate programming is required, optional, or not supported |
| Extraction Policy | Determines which tool and format is used for tangling/weaving |
| Test Policy | Defines whether code blocks must pass CI tests |

## Model (What)

Literate programming flips the relationship: the primary document is human-readable prose with embedded code blocks, rather than code with embedded comments. A **tangle** phase extracts executable code; a **weave** phase produces formatted documentation.

```
The `greet` function takes a name and returns a greeting string:

<<greet function>>=
func greet(name: String) -> String
    return "Hello, \{name}!"
@

The function uses string interpolation to construct the greeting.
```

Key features:
- **Code blocks** with named chunks (Knuth's WEB/noweb style).
- **Tangling** extracts code in compiler-required order.
- **Weaving** produces human-readable documentation (HTML, PDF, Markdown).
- **Executable examples** — all code blocks are tested.
- **Interactive mode** (Jupyter-style) for exploratory programming with embedded output.

## Default Strategy

Code-first literate programming: Markdown source with fenced code blocks. Tangle extracts only blocks marked with a language tag. Weave produces enriched Markdown or HTML. No mandatory literate programming — developers may use traditional code files.

## Alternative Strategies

| Strategy | Description |
|---|---|
| noweb-style | Named chunk system (Knuth's original approach). |
| Jupyter notebooks | Interactive cells with mixed prose/code/output. |
| Org-mode (Babel) | Emacs-based literate environment with multiple language support. |
| Doc-tests | Code examples in doc comments that are compiled and tested. |

## Open Questions

1. Should the language have mandatory literate programming support in the standard toolchain?
2. Should "tangle" produce one file per module, or a single merged output?
3. How to handle literate programming in a module/import system?
4. Should literate editing be supported in the IDE?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
