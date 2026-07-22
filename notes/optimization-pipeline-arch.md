# Optimization Pipeline Architecture

> **Note:** Exploratory analysis from 2026-07-21 conversation.
> Proposes modeling every optimisation as a transformation pipeline
> operating under a semantic contract — rather than a flat list of
> compiler passes.
>
> **See also:** [`OPTIMIZATION_MODEL.md`](../what/OPTIMIZATION_MODEL.md),
> [`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md)
> (§ Semantics Before Optimization, Explicit Optimization),
> [`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md),
> [`IR.md`](../how/architecture/IR.md),
> [`IMPLEMENTATION_POLICIES.md`](../how/IMPLEMENTATION_POLICIES.md)

---

## Core Idea

> **Every optimisation is a transformation pipeline operating under a
> semantic contract.**

The optimizer is a module that receives a program at one representation
level (AST, IR, or Runtime) and returns an equivalent program. It must
not change semantics — only the execution strategy.

String interning becomes one module among many (memoization, escape
analysis, vectorization, domain-specific IR rewriting), rather than an
ad-hoc special case.

---

## Fit with Existing Model

### ✅ What Already Aligns

1. **Semantic contract:** `DESIGN_PRINCIPLES.md` § *Semantics Before
   Optimization* already states: "Any transformation that preserves this
   behavior is an optimisation." The pipeline operationalises this
   principle.

2. **Module readiness:** `IR.md` (Invariant #1) says: "A strategy choice
   must be representable as a transformation *over* the IR, not baked
   into it." The IR is already a tree-graph hybrid designed for
   analysis/transform passes.

3. **Pipeline thinking:** The existing Language → Implementation
   Strategy → Execution stack is already a pipeline. The proposal
   names optimisation as the explicit layer between Strategy and
   Implementation.

### 🔧 What the Proposal Adds

**A. Optimisation as an explicit architectural layer**

```
Source AST
    ↓
[Semantic Analysis → Typed IR]
    ↓
[Optimization Pipeline]
    ├── Module: Constant Folding    (IR → IR, level-independent)
    ├── Module: Escape Analysis      (IR → IR, allocation-specialized)
    ├── Module: Inlining            (IR → IR, call-graph-informed)
    ├── Module: Vectorization       (IR → IR, loop-specialized)
    ├── Module: String Interning    (IR → IR or Runtime → Runtime)
    └── Module: Domain-Specific     (IR → IR, plugin-based)
    ↓
[Code Generation → Target-specific lowering]
```

**B. Policy → Optimisation Module mapping**

Existing Policies are *declarative preferences* (`Allocation Policy = Arena`).
The proposal maps them to *procedural modules*:

```
Implementation Strategy
    ├── Allocation Policy = Arena
    ├── Algorithm Policy = Adaptive
    └── Evaluation Policy = Lazy Defaults
            ↓
Optimization Pipeline
    ├── [Escape Analysis]          ← activated by Allocation Policy
    ├── [Stack Allocation Pass]    ← activated by Allocation Policy
    ├── [Adaptive Sort Selection]  ← activated by Algorithm Policy
    ├── [Lazy Thunk Insertion]     ← activated by Evaluation Policy
    └── [Dead Code Elimination]    ← always active (benign)
```

This makes the optimisation pipeline **strategy-driven** without changing
semantics — a natural extension of the existing Policy model.

**C. String interning as a concrete test case**

- **Level:** IR → IR (compile-time) or Runtime → Runtime (lazy at startup)
- **Activation:** `Allocation Policy ≠ Static`
- **Contract:** `"hello" ++ "world"` → one allocation, not two
- **Semantics preserved:** strings remain immutable, equality by value

### ⚠️ Tensions

1. **Evaluation order** is already classified as **Semantics** (not
   Optimisation) in the existing table. A pipeline module that reorders
   evaluation must fail the semantic contract check.

2. **Memory layout** is classified as **Depends on strategy**. An
   optimisation pass that reorders struct fields may change FFI/unsafe
   behaviour — must be modelled as a safe-only pass or gated.

3. **Module ordering may matter.** If Vectorization after String
   Interning produces a different result than the reverse, the pipeline
   is a partial order, not a sequence.

---

## What to Do Next (if adopted)

1. **Add architectural diagram** to `OPTIMIZATION_MODEL.md` — pipeline
   with modules, representation levels (AST / IR / Runtime), and
   semantic contract as gateway.

2. **Add Policy → Optimisation Module mapping table** — bridge from
   `IMPLEMENTATION_POLICIES.md` to the new model.

3. **Clarify IR invariants** — ensure IR supports plugin-style
   transformation passes (currently tree-graph hybrid, which is
   necessary but not sufficient).

4. **Add String Interning row** to the classification table as a
   concrete test case.

5. **Document partial-order dependencies** between optimisation modules,
   if any.

6. **Create Architecture-category EDR** — this adds an architectural
   layer (Optimization Pipeline) not currently described.
