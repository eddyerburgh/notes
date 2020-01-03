---
layout: default
title: Algorithm analysis
description: Notes on algorithm analysis.
nav_order: 1
parent: Algorithms
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/algorithms/algorithm-analysis
---

<!-- prettier-ignore-start -->

# Algorithm analysis
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Algorithm analysis involves calculating the complexity of algorithms, usually either the time-complexity or the space-complexity.

Two common tools used for algorithm analysis are the RAM model of computation and the asymptotic analysis of worst-case complexity {% cite algorithm-design-manual -l 31 %}.

## The RAM model of computation

The RAM (Random Access Machine) model is used for analyzing algorithms without running them on a physical machine.

The RAM model has the following properties:

- A simple operation (`+`, `\`, `*`, `-`, `=`, `if`) takes one time step.
- Loops and subroutines are comprised of simple operations.
- Memory is unlimited and access takes one time step (the RAM model does not consider whether data is on cache or disk).

{% cite algorithm-design-manual -l 31-2 %}

Using the RAM model, you measure the running time of an algorithm by counting the number of steps an algorithm takes on a given input {% cite algorithm-design-manual -l 32 %}.

Despite the simplifying assumptions made by the RAM model, it works well in practice.

### Best, worst, and average-case complexity

Often, an algorithm's input determines the number of steps the algorithm takes to run.

For a given input $$n$$:

- The **best-case complexity** is the minimum number of steps taken to complete.
- The **worst-case complexity** is the maximum number of steps taken to complete.
- The **average-case complexity** is the average number of steps taken to complete.

{% cite algorithm-design-manual -l 33 %}

Each of these complexities defines a numerical function that represents time vs problem size. For example, for an input of size $$n$$, an algorithm takes $$2n$$ steps to complete in the worst case. The worst-case function for time complexity here is $$T(n)=2n$$.

This can be graphed:

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithm-analysis/time-complexity-graph.svg" alt="">
  <figcaption><h4>Figure: A graph of T(n)=2n </h4></figcaption>
</figure>

Worst-case complexity is normally the most useful measure. This is because the best-case is often so unlikely that it isn't beneficial to think about. The average-case can be useful, but it is usually difficult to determine. The worst-case is both likely to happen, and easy to calculate {% cite algorithm-design-manual -l 33 %}.

## Big O notation

Big O is a mathematical notation that describes the limiting behavior of a function as its input tends towards infinity.

Big O is useful in algorithm analysis because the functions that we get from counting steps often require a lot of detail to specify. For example, a worst-case analysis might produce a function like this:

$$T(n) = 1234n² + 1228n + 92lg₂n + 8736$$

In reality, this level of detail is not much more informative than stating that "the time grows quadratically with n" {% cite algorithm-design-manual -l 34 %}.

It's easier to instead compare the upper and lower bounds of time-complexity functions with **asymptotic notation**. This includes big O, big omega, and big theta {% cite algorithm-design-manual -l 34 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithm-analysis/upper-and-lower-bounds.svg" alt="">
  <figcaption><h4>Figure: Upper and lower bounds for n > n0 {% cite algorithm-design-manual -l 35 %} </h4></figcaption>
</figure>

Big O describes the **upper bound** on a function $$f$$. $$f(n) = O(g(n))$$ means $$c · g(n)$$ is an upper bound on $$f(n)$$. There exists some constant $$c$$ such that $$f(n)$$ is always $$≤ c · g(n)$$ for large enough $$n$$ (i.e., $$n ≥ n_0$$ for some constant $$n_0$$) {% cite algorithm-design-manual -l 35 %}.

Big O notation ignores multiplicative constants. In big O analysis, $$f(n)=n$$ and $$g(n)=2n$$ are identical.

Big omega describes the **lower bound** on a function. $$f(n) = Ω(g(n))$$ means $$c · g(n)$$ is a lower bound on $$f(n)$$. There exists some constant $$c$$ such that $$f(n)$$ is always $$≥ c · g(n)$$, for all $$n ≥ n_0$$ {% cite algorithm-design-manual -l 35 %}.

Big theta defines a **tight bound** where a function $$f$$ is bound both above and below. $$f(n) = Θ(g(n))$$ means $$c_1 · g(n)$$ is an upper bound on $$f(n)$$ and $$c_2 · g(n)$$ is a lower bound on $$f(n)$$, for all $$n ≥ n_0$$. There exist constants $$c_1$$ and $$c_2$$ such that $$f(n) ≤ c_1 · g(n)$$ and $$f(n) ≥ c_2 · g(n)$$. This means that $$g(n)$$ provides a tight bound on $$f(n)$$ {% cite algorithm-design-manual -l 35 %}.

Big O is the most common notation used when analyzing algorithms.

## Growth rates and dominance relations

Big O notation creates classes of functions, where all functions in a class are equivalent in big O analysis {% cite algorithm-design-manual -l 39 %}.

There are a few common classes:

<!-- prettier-ignore-start -->

| Notation | Name        | Description    |
| -------- | ----------- | -------------  |
| $$O(1)$$     | Constant    | No matter the size of the input, the algorithm will take the same amount of time to complete. |
| $$O(log n)$$ | Logarithmic | Logarithmic algorithms grow slowly because they halve the amount of data they work with on each iteration. For example, [binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm). |
| $$O(n)$$     | Linear      | Linear algorithms grow linearly with n. For example an algorithm to calculate the exponent of a number by multiplying a value n times in a loop. |
| $$O(nlog n)$$ | Superlinear | Superlinear algorithms grow just a little faster than linear algorithms. Common sorting algorithms like Mergesort and Quicksort run in superlinear time. |
| $$O(n^2)$$    | Quadratic   | Quadratic algorithms run slowly. An example is an algorithm that checks for duplicates in an array by looping over each element, and then looping over every other element to check for matches. |
| $$O(2^n)$$ | Exponential | Exponential algorithms are very slow. An example is a recursive algorithm to find the nth term of the fibonacci sequence. |
| $$O(n!)$$ | Factorial | Factorial algorithms quickly become useless. They occur when generating all permutations of n {% cite algorithm-design-manual -l 39 %}.|

<!-- prettier-ignore-end -->

You can see the growth rate of the common classes in the following table:

| $$f(n)$$      | $$lg_n$$ | $$n$$    | $$nlg_n$$ | $$n^2$$             | $$2_n$$  | $$n!$$         |
| ------------- | -------- | -------- | --------- | ------------------- | -------- | -------------- |
| 10            | 0.003 μs | 0.01 μs  | 0.033 μs  | 0.1 μs              | 1 μs     | 3.63 ms        |
| 20            | 0.004 μs | 0.02 μs  | 0.086 μs  | 0.4 μs              | 1 ms     | 77.1 years     |
| 30            | 0.005 μs | 0.03 μs  | 0.147 μs  | 0.9 μs              | 1 sec    | 8.4 × 1015 yrs |
| 40            | 0.005 μs | 0.04 μs  | 0.213 μs  | 1.6 μs              | 18.3 min |                |
| 50            | 0.006 μs | 0.05 μs  | 0.282 μs  | 2.5 μs              | 13 days  |                |
| 100           | 0.007 μs | 0.1 μs   | 0.644 μs  | 10 μs 4 × 10^13 yrs |          |
| 1,000         | 0.010 μs | 1.00 μs  | 9.966 μs  | 1 ms                |          |                |
| 10,000        | 0.013 μs | 10 μs    | 130 μs    | 100 ms              |          |                |
| 100,000       | 0.017 μs | 0.10 ms  | 1.67 ms   | 10 sec              |          |                |
| 1,000,000     | 0.020 μs | 1 ms     | 19.93 ms  | 16.7 min            |          |                |
| 10,000,000    | 0.023 μs | 0.01 sec | 0.23 sec  | 1.16 days           |          |                |
| 100,000,000   | 0.027 μs | 0.10 sec | 2.66 sec  | 115.7 days          |          |                |
| 1,000,000,000 | 0.030 μs | 1 sec    | 29.90 sec | 31.7 years          |          |                |

{% cite algorithm-design-manual -l 38 %}

## References

{% bibliography --cited_in_order %}
