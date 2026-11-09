# Learning Model

> Status: working model for future Orthon learning materials. It does not
> define language semantics or add a step to the concept design pipeline.

## Purpose

The Coding Skills Hierarchy provides a way to identify the skills a
learning objective uses, choose examples, and make prerequisite knowledge
visible. It is adapted here for Orthon. Its effectiveness for Orthon
learners remains to be evaluated on actual materials and tasks.

The model classifies learning objectives, not people or the intrinsic
difficulty of a concept. The levels are a guide to possible dependencies,
not a mandatory sequence. Skills can develop together, and a learner can
use an abstraction through its contract before learning its implementation.

## Seven Levels

| Level | Name | Learning focus | Example objective | Code example (pseudocode) |
|---|---|---|---|---|
| 1 | Primitives & Types | Values, types, and representations needed to understand data. | Explain that an optional result may be absent. | `result: optional integer = absent` |
| 2 | API & Language Constructs | Language forms, their semantics, and library interfaces. Distinguish language constructs from library APIs. | Use a documented operation and explain its contract. | `length = size(items)` |
| 3 | Coding Primitives | Small operations composed from familiar language constructs. | Count occurrences in a collection. | `for item in items: count[item] = count.get(item, 0) + 1` |
| 4 | Coding Patterns | Recurring ways to organize code and express local design choices. | Use an early exit to handle a boundary case. | `if items is empty: return no_result` |
| 5 | Algorithmic Building Blocks | Data structures and basic algorithms, their contracts and relevant costs. | Choose a collection for a stated access pattern. | `queue.enqueue(task); next = queue.dequeue()` |
| 6 | Algorithmic Patterns | General strategies combining building blocks to solve classes of problems. | Recognize when a sliding window fits a problem. | `window_sum = window_sum - outgoing + incoming` |
| 7 | Problem Solving | Clarify requirements, decompose problems, assess trade-offs, and combine solutions. | Design a solution under explicit correctness and resource constraints. | No single fragment represents the full problem-solving process. |

The code column uses illustrative pseudocode, not accepted Orthon syntax or
library specification. Each fragment highlights one learning focus; a
complete example can require skills from several levels. For example, the
sliding-window update assumes that the reader knows how the outgoing and
incoming values are chosen. Allocation details, error-handling forms, and other
language-specific claims must be checked against their canonical sources.
Knowing an interface and implementing it are different objectives and may
require different prerequisites, even when they concern the same structure.

## Applying the Model

When future learning materials are created:

1. State what the reader should understand or be able to do.
2. Identify the relevant levels and the specific prerequisite concepts and
   skills. Link to prerequisite materials when available.
3. Choose an example that isolates the objective and introduces as few
   unrelated ideas as possible.
4. Explain the topic using an appropriate method, such as the
   [Knowledge Explanation Method](KNOWLEDGE_EXPLANATION_METHOD.md).
5. Provide an exercise that checks the objective and use reader feedback to
   improve the explanation and prerequisites.

The model can help reference documentation link to prerequisite learning
materials. Reference documentation retains its own structure and purpose:
precisely describing the language.

## Future Classification

A concept may be relevant to several levels; a learning example may have
one primary level and additional levels where useful. Specific
prerequisites are separate from level labels. A thesis may be classified
when used in learning material without changing its status or source.

No tag format is prescribed yet. Existing concepts and examples do not
require tags, and this model does not add a classification step to the
current pipeline. A separate documentation phase may evaluate and apply
classification when learning materials are planned.

## Evaluation

Test the model on a small set of future materials: check whether readers
can identify prerequisites, complete the stated task, and locate the source
of difficulty. Compare explanations and gather feedback before claiming
reduced cognitive load or faster learning. Prior experience matters; a
worked example useful to a novice may be redundant for an experienced
reader. See [Cognitive Architecture and Instructional Design: 20 Years
Later](https://link.springer.com/article/10.1007/s10648-019-09465-5).

## See Also

- [Vision](../why/VISION.md) — the project's goals for understandable language design.
- [Language Design Principles](DESIGN_PRINCIPLES.md) — simplicity and a stable mental model.
- [Theses](../what/THESES.md) — source-linked claims about language semantics.
