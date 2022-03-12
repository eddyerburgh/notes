---
layout: default
title: Compiler optimizations
description: Notes on compiler optimizations.
has_toc: false
nav_order: 4
parent: Compilers
permalink: /compilers/compiler-optimizations
---

<!-- prettier-ignore-start -->

# Compiler optimizations
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Optimizing compilers perform optimizations to improve a program's resource utilization. Generally the resource being optimized for is CPU time, but specialist compilers exist that optimize for other resources (e.g. code size, memory usage, disk accesses, etc.).

Optimization involves many subproblems that are computationally intractable. Therefore, heuristics are often used during optimization.

Typically, optimizations are run on an IR (Intermediate Representation) before the IR is passed to the code generator {% cite the-dragon-book -l 505 %}.

## Intermediate representation

IR (Intermediate Representation) is a language between the source code and the target language. It provides a layer of abstraction that:

1. Contains more details than the source
2. Contains less details than the target

An IR is designed to make processing of a program easier (e.g. optimization and translation). Some compilers translate through a series of intermediate languages during the compilation pipeline.

### Linear IRs

Linear IRs consist of sequentially-executed instructions. Linear IRs often resemble assembly code for an abstract machine.

Example LLVM IR:

```llvm
define i32 @main() #0 {
  %1 = alloca i32, align 4
  store i32 0, i32* %1, align 4
  ret i32 3
}
```

#### Three-address code

3AC (Three-Address Code) is a linear IR. In 3AC there is at most one operator on the right side of an instruction.

A source-language expression like `x - y * z` might be translated into a sequence of 3AC instructions:

```
t1 = y * z
t2 = x - t1
```

{% cite the-dragon-book -l 363 %}

3AC includes addresses and instructions.

A 3AC address can be one of:

- A name (source-code names)
- A constant
- A compiler-generated temporary

Operators are either constants or registers.

Some common 3AC instruction forms:

- Assignment (binary: `x = y op z`, unary: `x = op y`)
- Copy instructions (`x = y`)
- Unconditional jump (`goto L`, where L is a 3AC instruction with label `L`)
- Conditional jump (`if x goto L`)
- Conditional jump with relational operator (`if x relop y goto L`, relational operators: `<`, `==`, etc.)
- Indexed copy instructions (`x = y[i]`, `y[i] = x`)
- Address and pointer assignments (`x = &y`, `x = *y`, `*x = y`)
- Procedure calls (implemented using `param x` for parameter)

{% cite the-dragon-book -l 364-5 %}

3AC instructions can be represented as quadruples \<op, arg_1, arg_2, result\> {% cite the-dragon-book -l 366 %}.

#### Stack-machine code

Stack-machine code is a linear IR that models the behavior of a stack machine. It is a form of one-address code.

Most operations take their operands from the stack. For example, a `multiply` instruction would remove the top 2 instructions from the stack, multiply them, and push the result back to the stack.

Example stack machine code:

```
push 3
push b
multiply
push a
subtract
```

Many interpreters execute stack-machine bytecode on a virtual stack machine (e.g. CPython, JVM).

### Graphical IRs

Graphical IRs represent source code as a graph.

Graphical IRs can be trees:

- **Parse trees**—graphical representation of derivation
- **ASTs**—derivation with extraneous nodes removed
- **DAGs**—ASTs where nodes can have multiple parents and identical subtrees are reused

{% cite engineering-a-compiler -l 226-9 %}

#### Control-flow graphs

A **control-flow graph** represents the possible paths through basic blocks, where a basic block is a sequence of operations that always execute together (unless an operation raises an exception).

Each node represents a basic code block. A directed edge $$(B_i, B_j)$$ represents a possible transfer of control from $$B_i$$ to $$B_j$$ {% cite engineering-a-compiler -l 231 %}.

Control-flow graphs are often used with another IR where the control-flow graph represents the relationships between blocks, and the operations within blocks are represented with another IR (e.g. a linear IR) {% cite engineering-a-compiler -l 231-2 %}.

#### Data-dependence graphs

In compilers, a **data-dependence graph** represents the dependencies between individual instructions.

A data-dependence graph node $$n$$ represents an operation. An edge $$(n_i, n_j)$$ represents a definition value $$n_j$$ and an operation $$n_i$$ that uses the value {% cite engineering-a-compiler -l 233 %}.

Data dependence graphs are often supplementary data structures built from the definitive IR and discarded after use. They are used for instruction scheduling {% cite engineering-a-compiler -l 234 %}.

The edges in a graph represent hard constraints (an operation $$n_j$$ cannot run before operation $$n_i$$), the graph creates a partial order, where there are often many sequences that preserve the data dependencies of the graph. This property is exploited by out-of-order processors in order to schedule instruction efficiently {% cite engineering-a-compiler -l 233 %}.

### Static single-assignment form

**SSA (Static single-assignment form)** is a property of an IR which requires that each variable is assigned exactly once and that every variable is defined before its use.

The process for transforming ordinary code into SSA involves replacing the target of each assignment with a new variable, and replacing each use of a variable with the version of the variable reaching that point.

Since control flow can't be predicted in advance, there can be cases where a variable might refer to multiple versions. In this case, a variadic $$Φ$$ (Phi) function is used. You can read $$Φ(o_1, o_2, ..., o_n)$$ as "one of either $$o_1$$, $$o_2$$, ..., or $$o_n$$".

## Basic block

A **basic block** is a sequence of code with no branches within itself, except to the entry, and no branches out, except at the exit.

A **control-flow graph** is a directed graph where the nodes are basic blocks and an edge $$(B_1, B_2)$$ exists if execution can pass from the last instruction in $$B_1$$ to the first instruction in $$B_2$$.

The body of a method (or procedure) can always be represented as a control-flow graph. There is one initial node, and all return nodes are terminal.

## Optimizations

For languages like C, there are three optimization levels:

- Local optimizations—applied to a basic block.
- Global optimizations—applied to a single function, optimized across all basic blocks of that function.
- Inter-procedural optimizations—applied across method boundaries.

Commonly, applying optimizations enables new optimizations (e.g. running a copy propagation optimization enables dead code elimination). Optimizing compilers repeat optimizations until no new optimizations are found (or a limit is reached).

### Local optimizations

Local optimizations only consider information local to a single basic block.

#### Common subexpression elimination

**Common subexpression elimination** is where identical expressions are replaced with a single variable holding the computed value. This optimization is easy to do when the IR is in SSA.

#### Copy propagation

**Copy propagation** is where occurrences of direct assignments are replaced with their values. e.g. if `y = x` then `z = 3 + y` can become `z = 3 + x`. Copy propagation can enable dead code elimination and constant folding.

#### Constant propagation

Similar to copy propagation, **constant propagation** is the process of substituting values of known constants in expressions. e.g. if `x = 14` then `y = 3 + x` becomes `y = 3 + 14`.

#### Dead code elimination

**Dead code elimination** is the process of removing dead code, where **dead code** includes unreachable code and dead variables (variables written to but never read).

#### Constant folding

**Constant folding** is when the compiler reorganizes and evaluates constant expressions at runtime.

Example: `x := 2 + 2` becomes `x := 4`.

### Peephole optimization

**Peephole optimization** is an optimization technique applied to a short sequence of (usually contiguous) target language instructions. The sequence is known as the peephole.

The optimizer replaces the sequence with another sequence that produces the equivalent result but is faster.

Peephole optimizations are generally written as replacement rules:

$$i_1, ..., i_n \rightarrow j_1, ..., j_n$$

### Global Dataflow analysis

**Dataflow analysis** is a variety of techniques that derive information about the flow of data along program execution paths. This enables global optimizations, e.g. global constant propagation {% cite the-dragon-book -l 597 %}.

In dataflow analysis, a **program point** is a point in the program that is either before an instruction (the input state of an instruction) or after an instruction (the output state of an instruction). Dataflow analysis must consider all possible paths through program points that a program can take {% cite the-dragon-book -l 597 %}.

Although Dataflow analysis can be run on program points, it can also be run on the boundaries of basic blocks (requiring less computation).

There are two main forms of dataflow analysis:

- Forward flow analysis
- Backward flow analysis

In forward flow analysis, the exit state of a program point is a function of the program point's entry state. In backward flow analysis, the entry state of a program point is a function of the program point's exit state.

In forward flow analysis you initialize an entry point before running the analysis. In backward flow analysis, you initialize exit points.

In forward flow analysis, the value of a block's exit ($$\text{out}_b$$) is:

$$\text{out}_b = \text{transfer}_b(\text{in}_b)$$

Where $$\text{transfer}\_b$$ is an output function of the block $$b$$ (a transfer function).

And the value of a block's entry ($$\text{in}_b$$) is:

$$\text{in}_b = \text{join}_{p \in \text{predecessor}_b}(\text{out}_p)$$

Where the join operation combines the exit states of the predecessors of $$b$$, yielding the entry state of $$b$$.

Each data flow analysis has its own transfer function and join operation.

Backward flow analysis is the inverse:

$$\text{in}_b = \text{transfer}_b(\text{out}_b)$$

$$\text{out}_b = \text{join}_{p \in \text{successor}_b}(\text{in}_p)$$

#### Reaching definition analysis

Reaching definition analysis is a forward flow analysis that statically determines which definitions may reach a certain point.

In the following example, `d2` is a reaching definition for `d3` but `d1` is not:

```
d1 : y := 3
d2 : y := 4
d3 : x := y
```

Reaching definition analysis can be defined as:

$$\text{out}_b = \text{gen}_b \bigcup (\text{in}_b - \text{kill}_b)$$

$$\text{in}_b = \bigcup_{p \in \text{predecessor}_b} \text{out}_p$$

Where $$gen_b$$ is the set of all definitions introduced by $$b$$, and $$kill_b$$ is the set of all definitions that are overwritten by $$b$$.

#### Liveness analysis

Liveness analysis is a backward dataflow analysis used to calculate whether variables are live at each point in the program.

Liveness analysis can be used during register allocation to determine which registers should be favored {% cite the-dragon-book -l 608-9 %}.

A variable is live at statement $$s$$ if:

- There exists a statement $$s'$$ that accesses $$x$$.
- There is a path from $$s$$ to $$s'$$.
- The path has no intervening assignment to $$x$$.

A dead variable is one that is not live. A statement `x = ...` is dead code if x is dead, and can therefore be deleted.

Liveness analysis can be defined as:

$$\text{in}_b = \text{use}_b \bigcup (\text{out}_b - \text{def}_b)$$

$$\text{out}_b = \bigcup_{s \in \text{successors}_b} \text{in}_s$$

$$\text{in}_\text{exit} = \emptyset$$

Where $$def_b$$ is the set of variables defined in $$b$$ prior to any use of that variable in $$b$$, and $$use_b$$ is the set of variables whose values may be used in $$b$$ prior to any definition of the variable

## References

{% bibliography --cited_in_order %}
