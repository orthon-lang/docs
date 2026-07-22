# Explicit Composition and Dependency Management

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language support assembling complex systems from components — should it provide built-in dependency injection, service location, or framework-level wiring, or leave composition entirely to manual code?

Two opposing philosophies:

1. **Framework-managed composition (Java/Spring, C#/ASP.NET, Guice, Dagger)** — a dependency injection container manages object creation and wiring. Components declare their dependencies (via constructor injection, field injection, or service location). The container resolves the dependency graph, manages lifecycle (singleton, prototype, request-scoped), and often provides AOP (aspect-oriented programming) for cross-cutting concerns. The benefit: loose coupling at scale, centralized wiring, testability through mock injection. The cost: "magic" — implicit behaviour that is hard to trace, slow startup due to context scanning, XML/annotation configuration that drifts from code, and a framework dependency that becomes part of the application.

2. **Manual explicit composition (Go, Rust, Zig)** — dependencies are wired manually. A struct that needs a dependency receives it via a constructor parameter (explicit dependency injection without a container). No framework, no annotations, no magic. Go's philosophy is the starkest: "a little copying is better than a little dependency." The Go community favours small libraries over heavy frameworks, and manual wiring over DI containers.

The core problem: **framework-managed composition reduces boilerplate in large codebases but introduces implicit behaviour and framework lock-in**. Manual composition keeps everything explicit and debuggable but requires more code at assembly points. The choice determines how the language positions itself on the "convention vs. configuration" spectrum.

## Principles

1. **Explicit over implicit** — Dependency wiring should be visible in code. A reader should be able to trace how an object is created and what dependencies it receives without understanding a framework's injection rules.

2. **No framework lock-in** — The language should not require a framework for basic composition. DI containers and AOP frameworks are third-party choices, not architectural requirements.

3. **Constructor injection as the pattern** — If injection is needed, it should use the same mechanism as any other function call: pass dependencies to the constructor. No special injection syntax, no field injection, no magic.

4. **Testability through design, not tools** — Testability comes from designing composable components, not from a framework's mock injection. If a component is hard to test without a DI container, the component design is the problem.

5. **Small over large** — Prefer small, focused libraries over large frameworks. Composition happens at the application level, not the framework level.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Composition Policy | Determines whether the language provides built-in DI mechanisms or leaves composition to manual code |
| Module Resolution Policy | Governs how modules are resolved at build time (build system concern, not runtime) |
| Initialization Policy | Controls object creation patterns — constructors, factories, builders |

## Model (What)

The explicit composition model:

- **No built-in DI container.** Components receive their dependencies through constructors (or factory functions).
- **No injection annotations.** No `@Inject`, `@Autowired`, `@Singleton` — these are framework concerns, not language concerns.
- **Plain constructors.** A service that needs a repository receives it as a constructor parameter:

  ```orthon
  type UserService {
      repo: UserRepository
  }

  func NewUserService(repo: UserRepository) -> UserService {
      return UserService { repo }
  }
  ```

- **Manual wiring at the application entry point.** The `main` function (or equivalent) creates all components and wires them together:

  ```orthon
  func main() {
      repo = NewUserRepository(db)
      service = NewUserService(repo)
      handler = NewUserHandler(service)
      server = NewServer(handler)
      server.Listen()
  }
  ```

This is explicit, traceable, and debuggable. Every dependency is visible. There is no framework to learn, no annotations to remember, and no implicit behaviour to debug.

## Default Strategy

**Explicit composition with no built-in DI.** All wiring is manual. Constructors receive dependencies as parameters. The application entry point is the composition root.

## Alternative Strategies

1. **Built-in DI container** — Provide a language-level mechanism for declaring and resolving dependencies. Reduces boilerplate at scale but adds complexity and magic.

2. **Service locator** — A global registry that components query for dependencies. Simpler than DI but creates hidden dependencies and makes testing harder.

3. **Zero-framework with library support** — No built-in DI, but a small, optional standard library module for common wiring patterns (e.g., lifecycle management, factory defaults).

## Open Questions

1. Should Orthon provide a lightweight, optional wire-up helper in the standard library (like Go's `wire` project but simpler)?
2. How does explicit composition work with the module/package system — does every public type need a constructor function?
3. Should the language support a composition root convention (e.g., a standard `init` function pattern) that LLMs can generate reliably?
4. Does explicit composition scale to large applications, or does it become unmanageable beyond a certain component count?

## Cross-References

- See `OBJECT_INITIALIZATION.md` for constructor patterns and named arguments
- See `FUNCTIONS.md` for function types and factory function patterns
- See `VISIBILITY_AND_ENCAPSULATION.md` for how visibility affects public API design
- See [`../../why/MANIFESTO.md`](../../why/MANIFESTO.md) § Minimal Core for the principle against framework magic
