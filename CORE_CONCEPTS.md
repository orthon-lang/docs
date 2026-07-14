# Core Concepts

Orthon consists of two fundamental concepts:

1. **Data**
2. **Data Modifiers**

Every language construct is built from these two concepts.

## Data

Data is the primary abstraction.

For example:
```
(1, 2, 3)
```
represents a collection of values without assigning any specific semantic meaning.

It is simply data.

## Data Modifiers

A modifier transforms data into another representation.

Examples:
```
tuple(1, 2, 3)
```
→ Tuple
```
set(1, 2, 3)
```
→ Set
```
sequence(1, 2, 3)
```
→ Sequence
```
pack(1, 2, 3)
```
→ packed values

Each modifier expresses intent, while the compiler determines the most efficient implementation.

## Fundamental Data Types

Orthon provides several fundamental data representations.

- **Value**
- **Tuple**
- **Reference**
- **Sequence**
- **Set**
- **Option**
- **Result**

These are not special language mechanisms.

They are simply different representations of data.

## Sequence

Sequence is the fundamental type representing a sequence of values produced over time.

Unlike traditional generators or streams, Sequence describes what the result is, not how it is produced.

A Sequence is a normal object.

It can be:

- returned from functions;
- stored in variables;
- passed to other functions;
- transformed with library operations;
- consumed incrementally.


## Producing a Sequence

Readable syntax:
```
emit value
```
Compiler expansion:
```
return sequence(value)
```
Operator form:
```
return value ->
```
All three forms are semantically equivalent.

## Compiler Behavior

If a function contains emit, the compiler automatically:

- creates a Sequence;
- transforms the function into a resumable state machine;
- preserves execution state between emitted values;
- performs all implementation-specific optimizations.

This behavior is entirely transparent to the programmer.

## Sequence Typing

Sequence does not require explicit element types.

Example:
```
numbers():
    emit 1
    emit 2
    emit 3
```
or
```
mixed():
    emit 1
    emit true
    emit "Hello"
```
The compiler infers the most appropriate internal representation.

Static type information remains an implementation detail rather than part of the language syntax.

## Operators and Named Functions

Every language operator has an equivalent named function.

Examples:
```
&x          == ref(x)

x->         == sequence(x)
```
This principle provides:

- consistent semantics;
- easier metaprogramming;
- a minimal set of language primitives;
- readable APIs without relying on symbolic operators.

Operators are simply syntactic sugar over ordinary functions.
