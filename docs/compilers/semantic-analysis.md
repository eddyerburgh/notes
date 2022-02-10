---
layout: default
title: Semantic analysis
description: Notes on semantic analysis during compilation.
has_children: true
has_toc: false
nav_order: 2
parent: Compilers
permalink: /compilers/semantic-analysis
---

<!-- prettier-ignore-start -->

# Semantic analysis
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Semantic analysis** is a compiler process which validates that source code is semantically consistent with the language definition. An example is type checking. It also often includes gathering additional information for future phases (e.g. type information) {% cite the-dragon-book -l 8 %}.

Examples of validations made during semantic analysis:

- All identifiers are declared before usage
- Type checking
- Inheritance relationships are valid
- Methods in a class are defined only once

Semantic analysis is the last phase of the compiler frontend.

Most semantic analyses can be implemented as a recursive descent of an AST.

## Scope

The **scope** of a name binding is the part of a program where the name binding is valid (where the name can be used to refer to the entity). The same name may refer to different entities in different parts of the program.

Name bindings can have restricted scope, e.g. in C, where block scope restricts scope to a subset of a function.

**Lexical scope** (aka static scope) is where the scope only depends on the position of the identifier in the source text—the scope isn't based on run-time behavior. Most programming languages use static scope.

**Dynamic scope** is where the scope of an identifier depends on the execution of a program (e.g. the closest binding in the execution of the language). Lisp used to be dynamically scoped.

The "most closely nested" rule is where an identifier refers to the definition in the closest enclosing scope, such that the declaration precedes the use. C++ uses the "most closely nested rule".

## Symbol tables

A **symbol table** is a data structure that tracks the current bindings of identifiers.

When performing semantic analysis on a portion of the AST, the defined identifiers must be known.

## Types

A **type** is an attribute of data that defines the operations that can be performed on the data, the values that the data can take, and the way the data is stored.

The three main benefits of types in a compiler:

1. Safety
2. Expressiveness
3. Run-time efficiency

{% cite engineering-a-compiler -l 165 %}

**Statically-typed languages** are typechecked during compilation (e.g. C, Java). **Dynamically-typed languages** are typechecked at run-time (e.g. JS, Python).

Most statically-typed languages have escape mechanisms to circumvent the type system, like unsafe casts in C and Java.

**Implicit type conversion** is where a value of type $$T$$ is coerced into an expected type $$E$$ when $$T$$ is an invalid type for the operation being performed on it. A **strongly-typed language** typically doesn't perform implicit type conversions, whereas **a weakly-typed language** does perform implicit type conversions. e.g. `'1' + 1` throws an error in strongly-typed Python, and evaluates to `'11'` in weakly-typed JS.

A **type signature** defines the types of the parameters and the return value of a function or method.

**Type inference** is where the compiler automatically detects the type of an expression. For example, a variable could be declared without a type annotation and the compiler could infer the type at compile-time (e.g. `var` in C#).

A **sound type system** has the property that if a variable is declared with a particular type, then it will have that type at run-time. A sound type system has the ability to catch every possible bug that might happen at run-time.

A **complete type system** has the property that it will only ever catch bugs that will happen at run-time. This comes at the cost of sometimes missing errors that will happen at run-time.

### Type checking

**Type checking** is the process of verifying and enforcing type constraints. Static type checking is done at compile-time as part of semantic analysis.

Type checking can be implemented as a post-order tree walk, where each leaf node has a known type and each non-leaf node's type can be inferred from the types of its children.

Pseudo-code for typechecking an expression:

```python
def type_check(environment, node):
  if type(node) is AddExpressionNode:
    return type_check_add_expr(environment, node.e1, node.e2)
  ## .. case for each node

def typecheck_add_expr(environment, e1, e2):
  t1 = type_check(environment, e1)
  if not type(t1) == TInt:
    raise TypeCheckError('expected int')
  t2 = type_check(environment, e2)
  if not type(t2) == TInt:
    raise TypeCheckError('expected int')
  return TInt
```

### Type rules

A **type rule** is an inference rule that describes how a type system assigns a type to a syntactic construct. Type rules can be applied by a type system to verify that a program is well-typed and to determine the type of each expression.

An expression $$e$$ of type $$\tau$$ is written as $$e\!:\!\tau$$. The type environment is written as $$\Gamma $$.

The notation for inference is the same as for inference rules. In general:

$$\frac{\Gamma_1 \vdash e_1\!:\!\tau_1 \quad \cdots \quad \Gamma_n \vdash e_n\!:\!\tau_n}{\Gamma \vdash e\!:\!\tau}$$

The sequents above the line are premises that must be fulfilled in order for the rule to be applied, yielding the conclusion (the bottom sequents below the line). The turnstyle ($$\vdash$$) is read as "it is provable that ...".

An example rule:

$$\frac{\vdash e_2\!:\!Int \vdash e_2\!:\!Int}{\vdash e_1 + e_2\!:\!Int}$$

<!--
Type checking proves facts $$e\!:\!T$$. A **sound type system** is one where whenever $$\vdash e\!:\!T$$, e evaluates to type T at run-time.  -->

#### Type environment

A **type environment** is a function that maps identifiers to types, giving types for free variables in an expression.

When type checking, the environment is usually passed down the AST from the root towards the leaves.

Let environment $$\Gamma$$ be a function mapping identifiers to types. $$\Gamma \vdash e : T$$, is read "under the assumption that free variables have the type given by $$\Gamma$$, it is provable that the expression $$e$$ has type $$T$$.

$$\Gamma[T/x]$$ is a function that applied to $$x$$, returns $$T$$.

In some languages method names and identifiers exist in different namespaces, therefore you can have both a method and a variable `foo`. This is implemented by using different environments (e.g. one for identifiers, and one for method names).

### Subtyping

Subtyping is a form of type polymorphism where a subtype is related to another datatype (the supertype) by some notion of substitutability.

<figure>
  <img src="{{site.baseurl}}/assets/img/compilers/semantic-analysis/subtyping-tree.svg" alt="">
  <figcaption><h4>Figure: Subtyping hierarchy</h4></figcaption>
</figure>

If $$Y$$ is a subtype of $$X$$, the subtyping relation is written $$Y \le X$$.

In OO, subclasses can only add methods or override methods with the same type signature.

**Variance** refers to how type constructs (like function return types) use subtyping relations. An example is covariance, which is commonly used for function return type. **Covariance** of a return type $$X$$ would allow any subtype $$S$$ (so that $$S \le X$$) to be used in place of type $$X$$.

| Variance       | Supertype allowed | Subtype allowed |
| -------------- | ----------------- | --------------- |
| Invariance     | No                | No              |
| Covariance     | No                | Yes             |
| Contravariance | Yes               | No              |
| Bivariance     | Yes               | Yes             |

#### Least upper bounds

In pure OO languages, the Least upper bound (LUB) of two types $$S$$ and $$T$$ is their lowest common ancestor in the hierarchy tree.

In language where conditional expressions evaluate to a value, the type of an expression would be $$LUB(T_1, ..., T_N)$$, where $$T_1, ..., T_N$$ are the types corresponding to each consequent expression.

## References

{% bibliography --cited_in_order %}
