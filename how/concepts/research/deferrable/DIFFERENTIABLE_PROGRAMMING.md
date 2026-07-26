# Differentiable Programming

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).
>
> **⚠️ Scope note:** This concept may belong in the Standard Library (M2)
> rather than the Core Language (M1). The Core Inclusion Filter analysis
> below assesses this at the research level; the final determination will
> be made during Phase 4 (Decision Pipeline).

## Issue (Why)

How does a language support gradient-based computation — training and inference of machine learning models — as a first-class capability?

Differentiable programming is relevant to Orthon for two reasons:

1. **LLM-native toolchain** — If Orthon's toolchain includes LLM-based code generation, the LLM itself is a product of differentiable programming. Orthon should be able to express the training and inference of the models it depends on.

2. **AI workload capability** — If Orthon targets AI/ML workloads, tensor operations and automatic differentiation should be expressible without resorting to foreign function interfaces (FFI) or external DSLs.

Traditional approaches:

- **Library-based differentiation** (TensorFlow, PyTorch, JAX in Python) — tensor operations + autodiff as a library. Flexible but constrained by the host language. Control flow is difficult to differentiate through, and the programming model differs from normal code.
- **DSL-embedded differentiation** (Swift for TensorFlow, Dex, Julia with Zygote) — differentiation as a language-level construct. More natural integration with the host language's control flow, but requires compiler support.
- **Standalone differentiation systems** (MLIR, Enzyme) — differentiation as a compiler pass. Language-agnostic but operates at a lower abstraction level, making debugging and interleaving with normal code harder.

The core problem: **gradient computation is not a library problem alone — it requires compiler support for efficient and correct automatic differentiation through arbitrary control flow**.

## Principles

1. **Differentiation is a language construct, not a library** — The `grad` (gradient) operator is part of the language. It can differentiate through any function composed of differentiable primitives.

2. **Differentiable primitives are first-class** — Tensors, scalar arithmetic, and activation functions are first-class standard library types with compiler-known differentiation rules.

3. **Differentiation through control flow** — The `grad` operator correctly handles conditionals, loops, and function calls within the differentiated function (using gradient tape or source-code transformation).

4. **Explicit differentiation boundaries** — The boundary between differentiable and non-differentiable code is syntactically visible. Mixing I/O or non-differentiable operations inside a differentiable function produces a compile-time error.

5. **Efficiency parity with hand-written gradients** — The compiler's automatic differentiation must produce code of comparable efficiency to hand-written backward passes. No overhead for simply using `grad` instead of manual backpropagation.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Differentiation Policy | Controls whether automatic differentiation is enabled and which mode (forward-mode, reverse-mode, or both) |
| Tensor Policy | Governs tensor representation, layout, and execution strategy (CPU, GPU, TPU) |
| Numeric Precision Policy | Specifies default floating-point precision for tensor operations |
| Autodiff Compilation Policy | Determines whether differentiation is performed at compile time (source-code transformation) or runtime (gradient tape) |

## Model (What)

### Gradient Operator

The `grad` operator takes a function and returns its gradient function:

```orthon
fn f(x: Float) -> Float
    x * x + 3.0 * x + 1.0

let df = grad(f)
# df is a function: (Float) -> Float
# df(2.0) == 2 * 2.0 + 3.0 == 7.0
```

### Multi-Parameter Gradients

For functions with multiple parameters, `grad` returns a tuple of partial derivatives:

```orthon
fn loss(predicted: Tensor, actual: Tensor) -> Float
    (predicted - actual).square().mean()

let d_loss = grad(loss)
# d_loss(pred, actual) -> (Tensor, Tensor)  # gradient w.r.t each parameter
```

### Gradient of a Function with Parameters

For functions that close over parameters (e.g., model weights), `grad` can compute gradients with respect to specific parameters:

```orthon
let weights = Tensor.randn([784, 256])
let biases = Tensor.zeros([256])

fn forward(x: Tensor) -> Tensor
    x.matmul(weights).add(biases).relu()

let d_weights = grad(forward, wrt: [weights, biases])
# d_weights(x) -> (Tensor, Tensor)  # gradients for weights and biases
```

### Higher-Order Gradients

`grad` can be composed to compute second derivatives:

```orthon
fn f(x: Float) -> Float
    x.pow(3.0)

let df = grad(f)      # first derivative: 3*x^2
let d2f = grad(df)    # second derivative: 6*x

assert d2f(2.0) ≈ 6.0
```

### Differentiable Control Flow

The `grad` operator differentiates through control flow correctly:

```orthon
fn relu(x: Float) -> Float
    if x > 0.0 then x
    else 0.0

let d_relu = grad(relu)
# d_relu(-1.0) == 0.0
# d_relu(2.0) == 1.0
```

## Core Inclusion Filter

Before entering formal validation, this concept is run through the Core Inclusion Filter procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 3 (Standard Library) — differentiable programming is
            primarily a library of tensor types and autodiff combinators,
            with potential compiler-level optimisation support.

Library test:   Can Differentiable Programming be implemented as a Standard
                Library function using existing Core constructs?

                The core of differentiable programming consists of:
                - Tensor type (a multidimensional array)
                - Arithmetic operations on tensors (add, multiply, matmul)
                - Automatic differentiation (gradient computation)
                - Activation functions and loss functions

                Tensor types and operations can be implemented as a Standard
                Library module using existing Core constructs (function, call,
                pack/unpack for tensor data, operator definition for arithmetic).
                Automatic differentiation can be implemented as a library function
                using source-code transformation or gradient tape — but only if
                the language supports a built-in mechanism for inspecting and
                transforming function bodies at compile time.

                If the language has a `macro` or `compiler_transform` primitive,
                autodiff can be a library. If not, autodiff requires compiler-level
                support (a new Primitive Operation).

                → PENDING — depends on whether the language has a macro/transform
                  primitive. If yes → Level 3. If no → Level 1.

Pattern test:   Can Differentiable Programming be expressed as a composition
                of existing Primitive Operations and Data Model constructs?

                Assuming the language has function, call, arithmetic operators,
                and a macro/transform primitive:
                - Tensor arithmetic composes from `function` + `call` + `operator`
                - Autodiff composes from `macro/transform` + `function composition`
                - Control flow differentiation composes from `condition` + `loop`
                  + autodiff of each branch

                Without a macro/transform primitive: autodiff requires a new
                primitive that can inspect and transform function bodies — which
                is not composable from existing primitives.

                → PENDING — composition depends on macro/transform primitive.

Primitive test: Is Automatic Differentiation an atomic operation on data
                that cannot be decomposed?

                Automatic differentiation (reverse-mode) is a specific algorithm
                on function composition — not an atomic operation. It decomposes
                into:
                1. Tracing the computation graph
                2. Applying the chain rule backward through the graph
                3. Accumulating gradients

                Each step is a library-implementable transformation on function
                representations. The only language-level support needed is a way
                to inspect and transform function bodies at compile time.

                → FAIL (not primitive) — if the macro/transform primitive exists,
                  autodiff decomposes into simpler steps.

Filter result: Level 3 (Standard Library) — assuming the language provides
               a macro/compiler-transform primitive. If no such primitive
               exists, the gradient operator (`grad`) would need to be a new
               Primitive Operation (Level 1) to support function body inspection
               and transformation.

               For M1 (language specification), the concept is recorded as
               Standard Library design guidance. The `grad` operator may be
               syntactic sugar over the library's autodiff function, not a
               language-level built-in.
```

## Default Strategy

The Default Strategy uses **runtime gradient tape** for automatic differentiation — the computation graph is traced during execution and gradients are computed in a backward pass. This is simpler to implement and supports dynamic control flow, at the cost of higher memory usage and trace overhead.

```orthon
# Runtime tape (default):
let d_loss = grad(loss)    # traces computation at runtime
```

## Alternative Strategies

| Strategy | Description |
|---|---|
| **Source-code transformation** | The compiler transforms differentiable functions at compile time, producing an explicit backward pass. Zero runtime overhead but requires static control flow analysis. Suitable for HIGH_PERFORMANCE_STRATEGY. |
| **Forward-mode autodiff** | Computes derivatives alongside the forward pass. More efficient for functions with few inputs and many outputs. Suitable for DEFAULT_STRATEGY as a `grad_forward` complement to `grad`. |
| **Library-only** | No language-level `grad` operator. Differentiable programming is entirely through the Standard Library tensor module, with manual gradient specification or a JAX-style `grad` as a library combinator. Simpler language, more verbose user code. |
| **Deferred to M2** | Differentiable programming is not specified in M1 at all. It becomes a Standard Library design task in Milestone 2. Simpler M1 specification, but risks missing architectural hooks in the compiler. |

## Open Questions

1. Should the `grad` operator be a language built-in or a standard library function with syntactic sugar?
2. Does Orthon need a macro/compiler-transform primitive to support library-level autodiff?
3. Should tensors be part of the Core Language (Level 0 Data Model) or Standard Library (Level 3)?
4. How does differentiable programming interact with the Execution Program model (if a program's execution strategy is decoupled from its semantics, how does `grad` work across strategies)?
5. Should hardware acceleration (GPU, TPU) be part of the Tensor Policy or deferred to the platform implementation?
6. Should mixed-precision training be a language-level concern or a library-level concern?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/LIBRARY_BOUNDARY.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
- [ ] `how/strategies/HIGH_PERFORMANCE_STRATEGY.md`
- [ ] `how/strategies/LLM_STRATEGY.md`
- [ ] `how/architecture/IR.md`
