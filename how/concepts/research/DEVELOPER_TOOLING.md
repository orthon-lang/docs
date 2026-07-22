# Developer Tooling

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

What developer tools must a language provide out of the box for a productive development experience?

Languages ship with wildly varying tooling:
- **Go** — `gofmt`, `go vet`, `go test`, `go build`, built-in formatter and linter. No third-party tools needed for basic workflow.
- **Rust** — `cargo` with built-in formatter (`rustfmt`), linter (`clippy`), test runner, documentation generator, package registry (crates.io).
- **Java** — Maven/Gradle with centralised artifact repositories (Maven Central). Build configuration is XML/Groovy/Kotlin DSL. Dependency resolution downloads artifacts from central servers. Plugins for formatting, linting, and testing are third-party.
- **Python** — no built-in tooling. Rely on `pip`/`poetry`, `black`/`ruff`, `pytest` — all third-party, configuration-heavy.
- **JavaScript** — fragmented ecosystem: npm/yarn/pnpm, eslint/prettier/biome, jest/vitest. High cognitive overhead choosing and configuring tools.

The core problem: **tooling fragmentation wastes developer time on configuration instead of solving problems**. A language that ships with a complete, opinionated toolchain eliminates this overhead entirely.

### Package Management Models: Central Repositories vs. Source-Based

Two fundamentally different approaches to package management exist:

1. **Central artifact repositories (Java's Maven Central, C#'s NuGet, Rust's crates.io)** — packages are published as versioned artifacts (JARs, DLLs, crate files) to a central registry. The build tool downloads artifacts from the registry. The registry is the authoritative source; the source code may live elsewhere or be inaccessible. Maven Central is the canonical example: artifacts are published via a rigorous process (staging, signing, verification). This provides reproducibility and security (signed artifacts) but introduces publishing friction and centralised control.

2. **Source-based distribution (Go's `go mod`, Node's npm)** — dependencies are fetched directly from version control repositories (GitHub, GitLab, etc.) and built from source. There is no central artifact repository for binaries. Go's approach is the most radical: the `go.mod` file lists module paths and versions; `go mod download` fetches the source from the module's repository. The source IS the artifact. This eliminates the publish step entirely — pushing a tag to a Git repository makes a new version available — but shifts trust from a central registry to each source repository independently.

Key differences:

| Aspect | Central Repositories (Maven/Gradle) | Source-Based (Go mod) |
|---|---|---|
| **Publish step** | Required: build, sign, upload artifact | None: push tag to Git |
| **Trust model** | Centralised: registry verifies publishers | Decentralised: trust per-repository |
| **Reproducibility** | High: artifact is immutable once published | Medium: depends on tag stability and Git history |
| **Supply chain** | Signed artifacts, checksums, verified by registry | Checksums in `go.sum`, no signing requirement |
| **Offline builds** | Cached artifacts | Cached source, needs build toolchain |
| **Build from source** | Optional (source JARs available) | Required (dependencies are always built) |

The Orthon approach follows Go's model: source-based distribution with a single authoritative registry (`wvy` registry, like crates.io) providing discovery and checksums. No central artifact repository for binaries — the source IS the artifact. This eliminates the publish-build-sign-upload cycle and makes releasing a new version as simple as pushing a Git tag.

## Principles

1. **Batteries included** — Package manager, formatter, linter, test runner, and build tool are part of the language distribution. No third-party tools required for standard workflows.
2. **Zero-config by default** — Tools work without configuration files. Configuration is additive, not mandatory.
3. **One package registry** — A single, unified package registry (`wvy`) — no namespace fragmentation.
4. **Script and binary modes** — Scripts run instantly (interpreted / JIT). The same code compiles to a standalone binary with no changes.
5. **REPL with IDE-level intelligence** — REPL provides autocomplete, type hints, and documentation lookup — not just a bare read-eval-print loop.
6. **LLM-native** — The toolchain exposes machine-readable language schemas so LLMs can generate correct code by contract, not by pattern-matching.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Toolchain Policy | Defines the set of tools included in the language distribution |
| Package Resolution Policy | Governs how packages are resolved, cached, and updated |
| Build Policy | Controls compilation modes (interpreted, JIT, AOT) |
| Formatting Policy | Determines canonical code style (opinionated, no options) |

## Model (What)

The language ships with a complete toolchain under the `wvy` command (or a single entry point):

```bash
# Package management
wvy init                    # create a new project
wvy add serde               # add dependency
wvy build                   # build project
wvy run                     # run project

# Quality tools
wvy fmt                     # format code (opinionated, no config)
wvy lint                    # static analysis
wvy test                    # run tests

# Compilation
wvy build --release         # compile to native binary
wvy script run.orthon       # run a single script

# REPL
wvy repl                    # interactive REPL with autocomplete
```

### Package manager (`wvy`)

```orthon
# The package manifest (auto-generated, editable)
# wvy.toml
[package]
name = "my-app"
version = "0.1.0"

[dependencies]
serde = "1.0"
http = "2.3"

# Usage in code
import serde
import http
```

Key features:
- **Single registry** — one authoritative package source (like crates.io or pkg.go.dev).
- **Semantic versioning** — dependency resolution uses semver ranges.
- **Lock file** — deterministic builds via `wvy.lock` (auto-generated).
- **No build script DSL** — build configuration is minimal. Complex builds use the language itself.

### Formatter (`wvy fmt`)

- **Opinionated** — no configuration options. One true style (like `gofmt`).
- **No debates** — formatting is not a topic for code review. It's automatic and uniform.
- **Edits in place** by default with `--check` flag for CI.

### Linter (`wvy lint`)

- **Rule set** — curated set of lint rules (like `clippy`). No configuration needed.
- **Fix suggestions** — auto-fix where possible (unused imports, redundant patterns).
- **Gradual strictness** — new projects get all rules. Legacy code can opt out per-rule.

### Test runner (`wvy test`)

- **Built-in** — no external test framework required.
- **Discovery** — tests are discovered by convention (files ending in `_test.orthon`, functions annotated with `@test`).
- **Coverage** — built-in coverage reporting (`wvy test --coverage`).

### REPL (`wvy repl`)

- **Autocomplete** — tab completion for all visible symbols.
- **Type hints** — shows inferred types as you type.
- **Documentation** — `?name` shows doc for `name`.
- **History** — persistent history across sessions.

### Interactive REPL with Hot Reload

The REPL supports an **interactive development mode** where code changes are applied to a running program without restarting:

```bash
wvy repl --hot-reload app.orthon
```

Behaviour:

1. **Session persistence** — The REPL session state (variable bindings, type environment, loaded modules) is preserved across code edits. Changing a function redefines it in the live session; dependent values may need manual re-evaluation.

2. **Hot reload boundaries** — Code is organised into hot-reloadable units (functions, types, module-level definitions). A hot reload replaces a unit atomically. State that crosses a reload boundary (e.g., an in-memory cache populated by a now-replaced initialisation function) is the programmer's responsibility to manage.

3. **REPL as notebook** — The REPL supports multi-line editing, embedded output (tables, charts from data), and cell-by-cell execution — similar to a Jupyter notebook but in a terminal. Output is rendered inline with ANSI formatting or rich terminal protocols.

4. **Interactive type inspection** — `:type <expr>` shows the type of any expression. `:docs <name>` shows documentation. `:holes` lists all unfilled typed holes in the current session (see [`TYPED_HOLES.md`](TYPED_HOLES.md)).

5. **LLM integration** — The REPL exposes an API endpoint for LLMs to inspect and modify the session state. An LLM can query current bindings, execute expressions, and observe results — all within the same session.

```orthon
# REPL session example:
> x = 42
> :type x
Int
> fn greet(name) = "Hello, {name}!"
> greet("World")
"Hello, World!"
> # (edit greet to return "Hi, {name}!" in editor)
> greet("World")       # hot-reloaded
"Hi, World!"
```

### Script mode

```orthon
# hello.orthon — runs as a script, no project needed
print("Hello, World!")

# Terminal
# $ wvy run hello.orthon
# Hello, World!

# Or compile to binary:
# $ wvy build hello.orthon --output hello
# $ ./hello
# Hello, World!
```

## Default Strategy

The toolchain is bundled with the language compiler. `wvy` is the single entry point for all operations. The formatter has zero configuration options. The linter has a single `--strict` flag. The package manager uses the official registry.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Ecosystem-only (Python, JS) | No built-in tools. Third-party ecosystem provides alternatives. Maximum choice, maximum fragmentation. |
| Minimal tooling (C, C++) | Compiler only. Build systems (CMake, Make) and package managers (vcpkg, Conan) are external. |
| Hybrid (Rust `cargo`) | Built-in package manager + build tool. Formatter and linter are separate installs but from the same organisation. |
| SDK bundling (Go) | Built-in `gofmt`, `go vet`, `go test`. Package manager is built-in but registry is decentralised. |

## Open Questions

1. Should `wvy` support workspaces (monorepos with multiple packages)?
2. Should the formatter support a minimal configuration file for edge cases (line width, indent style)?
3. How to handle private package registries (enterprise use case)?
4. Should the REPL support multi-line editing and notebook-style output?
5. Should the test runner support property-based testing (QuickCheck style) natively?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/LIBRARY_BOUNDARY.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
- [ ] Other: `how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`
