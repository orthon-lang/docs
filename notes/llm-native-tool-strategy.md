# LLM-Native Tool Strategy

> **Date:** 2026-08-19
> **Context:** Strategy discussion — how Orthon becomes a first-class,
> convenient, popular tool for LLMs, on par with Python, Node.js, and
> bash as execution substrates for AI agents.
>
> **Status:** Exploratory note — not a decision record. Prompts for
> follow-up design work; nothing here is accepted design.
>
> **See also:**
> [`why/VISION.md`](../why/VISION.md) § Designed for the LLM Era,
> [`why/GOALS.md`](../why/GOALS.md) § Goal 4 (LLM Readiness),
> [`notes/language-llm-comparison.md`](language-llm-comparison.md),
> [`notes/llm-generability-gate.md`](llm-generability-gate.md),
> [`how/concepts/research/deferrable/LLM_NATIVE_TOOLCHAIN.md`](../how/concepts/research/deferrable/LLM_NATIVE_TOOLCHAIN.md),
> [`how/strategies/LLM_STRATEGY.md`](../how/strategies/LLM_STRATEGY.md),
> [`notes/whitepaper-strategy.md`](whitepaper-strategy.md),
> [`when/ROADMAP.md`](../when/ROADMAP.md)

---

## Core Thesis

Orthon should not position itself as "another general-purpose language."
It should be the **execution substrate for LLM agents** — the thing an
agent produces and runs when correctness, reproducibility, and safety
matter more than raw convenience.

The one-line pitch: **"Python for people, Orthon for agents"** — not a
replacement, a second execution tier.

The LangOps framing (from `notes/whitepaper-strategy.md`): what Docker
did for deployment, Orthon does for agent-executed code. The Execution
Program collapses dev/deploy boundaries into a single, fully-defined,
portable artifact.

---

## What Orthon Must Be (Identity)

Three facets of one thing:

| Facet | Audience | Meaning |
|-------|----------|---------|
| **Language with a minimal predictable core** | LLM as generator | Orthogonality, explicit semantics, zero implicit behaviour — minimal hallucination surface |
| **Execution Program** | LLM agent as executor | Program = fully-defined artifact: source + language version + dependencies + **permissions + resource limits**. This is exactly what agent sandboxes lack today |
| **Machine-readable schema** | LLM tooling | `Schema Provider` is the single source of truth instead of guessing from a training corpus |

Positioning slogan: **"safe bash for agents"** — agents emit a
self-contained, reproducible, declaratively constrained artifact rather
than an unstructured script.

---

## What Orthon Must Do (Capabilities)

The LLM Toolchain (per `how/concepts/research/deferrable/LLM_NATIVE_TOOLCHAIN.md`):

1. **Schema Provider** — grammar + type system + stdlib contracts +
   active Strategy as structured data. Kills hallucination of
   non-existent APIs. The heart of everything.
2. **Static Analyser** — the agent's internal feedback loop: catches
   generation errors *before* compilation, not after.
3. **Structured diagnostics** (`Error Policy = LLM`) — machine-readable
   error codes + repair hints so the LLM self-corrects without
   re-prompting.
4. **Introspection API** — types, completions, visible symbols,
   diagnostics by code position; stateless, deterministic. Lets an
   agent reason about code without executing it.
5. **Code Generator / Completer / Documentation Generator /
   Refactor-Migration** — remaining toolchain components.
6. **Sandboxed execution** — permissions/resources/timeouts declared
   inside the Execution Program. Currently under-specified as a runtime
   property — a genuine gap (see Open Items).
7. **MCP server** — the natural integration point for real agents
   (Claude Code, Codex, OpenAI Agents). Noted as "natural
   implementation" in `.planning/notes/2026-07-20-orthon-llm-native-design.md`
   but not yet specced.

Plus the differentiator no competitor has: the **LLM Generability Gate**
(`notes/llm-generability-gate.md`, EDR-014) — every construct validated
for "can an LLM reliably generate this?" before acceptance.

---

## Implementation Form (Ordering Matters More Than Shape)

The repo is specification-only by design; implementation lives in a
separate repository (ROADMAP M4). Strategic recommendation: the LLM
toolchain is the **primary product**, not an add-on — the language sells
the tooling, not the reverse.

1. **Schema Provider + Static Analyser + Introspection API first — BEFORE the compiler.**
   A language without a compiler but with these three is already useful
   to agents as a verify tool. In ROADMAP the LLM Toolchain sits at M2
   (8.3); pull it forward — it is the adoption wedge.
2. **Then interpreter + LSP + MCP server** — the minimal path for an
   agent to actually *run* an Orthon program.
3. **Then AOT / OCI / WASM targets** — this closes the Execution Program
   loop (one artifact → interpreter, compiler, container, WASM).
   Portability is the long-term advantage, not the starting wedge.

---

## Making It Actually Popular (Adoption)

Design intent alone is necessary but not sufficient. Two hard problems:

### Cold-Start Corpus

LLMs write Python/JS/bash fluently because those dominate training data.
Orthon has essentially no corpus. So "LLMs can reliably generate Orthon"
must rest on the **Schema Provider + RAG**, not pretraining:

- A **compact schema pack** that fits in an LLM's context window (the
  full schema is not enough — the injectable subset is what matters).
- **Few-shot / idiom packs** and the `cookbook` (already planned in
  ROADMAP M1) injected into the agent's context.
- Long term: a **synthetic corpus generated from the schema** for
  fine-tuning.

### Distribution and Integration (the deciding factor)

"Popular for LLMs" ≠ "people write it." Popularity means **agents pick
it automatically**:

- One-command install; **preinstalled in agent sandboxes** (as bash and
  Python already are everywhere).
- **MCP server as the standard interface** so any agent harness can use
  it as a tool.
- **Native integration in Claude Code / Codex / OpenAI Agents /
  LangGraph** as the "safe executor" replacing raw bash.

### Niche Instead of Head-On Competition

Do not compete with Python/Node/bash in general. Win the narrow but
growing niche where generation errors are expensive:

- Agent tool-use and pipeline glue (what agents do via bash today).
- Embedded scripting inside agent frameworks.
- Deterministic glue where reproducibility and sandboxing matter.

---

## Open Items (Gaps Between Design Intent and a Popular Tool)

1. **Sandbox execution contract** — Execution Program permissions,
   resource limits, timeouts must be specified as a runtime property,
   not just a vision statement. This is the killer feature against bash
   for agents.
2. **MCP server spec** — define the MCP surface over Schema Provider +
   Introspection API + Static Analyser.
3. **Schema pack** — the compact, context-injectable subset of the
   schema, sized for LLM context windows.
4. **Toolchain priority shift** — ROADMAP M2 (8.3) LLM Toolchain should
   move ahead of the general compiler work.
5. **Synthetic corpus plan** — how to generate, validate, and release
   fine-tuning / few-shot data from the schema.

---

## Implication for the Roadmap

The LLM toolchain is the product; the compiler is infrastructure. If
Orthon is to reach "on par with Python/Node/bash as an LLM tool,"
planning should sequence LLM-facing components (schema, analyser,
introspection, MCP, sandbox contract) as the first implementation
milestone rather than the general-purpose compiler first.

---

## Addendum — ODCS Scoping (2026-09-02)

Reviewed whether the Open Data Contract Standard (ODCS) has a role in the
LLM-native toolchain. The conclusion is a scope boundary, not a feature:

- **Not applicable to the language layer.** ODCS describes *data* (schema,
  quality, SLA at a producer/consumer boundary), not *code*. It must not be
  used for language constructs, public module APIs, or the Schema Provider —
  API/type conformance is the Static Analyser's job against the real
  language schema, and the Schema Provider remains the single source of
  truth.
- **Only legitimate domain: the I/O boundary.** An Execution Program's
  declared inputs/outputs *as data* are the one place a data-contract
  standard could apply.
- **Deferred, not designed.** I/O contracts already live as Orthon types;
  ODCS adds nothing over them until a real external consumer (data catalog,
  quality tool, agent harness) needs a standard, interoperable contract
  document. Decide by fact when that consumer exists — do not pre-build
  ODCS integration.

**Reusable rule:** when any external standard is proposed for Orthon, first
ask whether it describes *data at the boundary* or *language/API*. The
latter is never an external standard's job.
