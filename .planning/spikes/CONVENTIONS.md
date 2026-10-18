# Spike Conventions

Patterns established across spike sessions. New spikes follow these unless the question requires otherwise.

## Stack

Documentation-only repository: spikes are written findings (Markdown tables) and, when needed, a throwaway script kept out of the repository. No tooling is added by a spike.

## Structure

- One directory per spike: `NNN-descriptive-name/README.md` with the workflow front matter.
- Metric definitions live in the highest-numbered spike of the idea (here 004) and earlier spikes refer to them.

## Patterns

- Unit of analysis: the idea (a semantic operation), not the concept file.
- Derive over the 10 primitives of `what/PRIMITIVE_BLOCKS.md`; whatever has no kernel element is recorded as a gap (candidate primitive), not forced into a composition.
- Report a vector of metrics; state limits (sample size, rater, sensitivity) in every README.
- Specification documents stay English (project rule), including spike READMEs.
