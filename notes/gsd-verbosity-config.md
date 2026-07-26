# GSD Verbosity Configuration

> How to make GSD more verbose — show decisions, progress, and reasoning in chat output.

## Current Config (baseline)

The current `.planning/config.json` has these relevant settings:

- `mode: "yolo"` — most autonomous, minimal explanation
- `granularity: "standard"` — default detail level
- `human_verify_mode: "end-of-phase"` — verify only at phase end
- `model_profile: "balanced"` — Opus (planning), Sonnet (execution/verification)

## Settings That Increase Verbosity

### 1. `mode` — currently `"yolo"`

The most impactful setting. `"yolo"` mode runs autonomously with minimal chat output. Switch to a less aggressive mode (if available):

```bash
gsd-tools query config-set mode "ask"
```

### 2. `human_verify_mode` — currently `"end-of-phase"`

Controls when GSD stops and asks for human confirmation:

| Value | Effect |
|-------|--------|
| `"end-of-phase"` (current) | Verify once at phase completion |
| `"per-task"` | Verify after each task — more checkpoints, more visibility into decisions |

```bash
gsd-tools query config-set workflow.human_verify_mode "per-task"
```

### 3. `model_profile` — currently `"balanced"`

Controls which model tier each GSD agent uses:

| Profile | Effect |
|---------|--------|
| `"balanced"` (current) | Opus for planning, Sonnet for execution/verification |
| `"quality"` | Opus for all agents — more thoughtful reasoning, more detailed explanations |
| `"adaptive"` | Role-based: Opus for planning/debug, Sonnet for execution/research, Haiku for mapping |

Higher-tier models typically produce more verbose, better-reasoned output.

```bash
gsd-tools query config-set model_profile "quality"
```

### 4. `code_review_depth` — currently `"standard"`

Controls how thorough `$gsd-code-review` is:

| Value | Effect |
|-------|--------|
| `"standard"` (current) | Per-file analysis |
| `"quick"` | Pattern-matching only |
| `"deep"` | Cross-file analysis with import graphs — most output |

```bash
gsd-tools query config-set workflow.code_review_depth "deep"
```

### 5. Already-enabled sub-agents (produce verbose output)

These are already `true` and contribute decision visibility:

- `research: true` — domain research before planning
- `plan_check: true` — plan verification before execution
- `verifier: true` — goal-backward verification after execution
- `pattern_mapper: true` — codebase pattern analysis
- `nyquist_validation: true` — test coverage analysis
- `code_review: true` — structured code review
- `intel.enabled: true` — codebase intelligence queries

## Quick Interactive Config

For a guided setup:

```bash
gsd-config          # Common settings (model, research, plan_check, etc.)
gsd-config --advanced  # Power-user knobs (timeouts, branching, language, etc.)
```

## Summary

For maximum chat verbosity, change these three settings:

```bash
gsd-tools query config-set mode "ask"
gsd-tools query config-set workflow.human_verify_mode "per-task"
gsd-tools query config-set model_profile "quality"
```
