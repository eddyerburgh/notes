---
layout: default
title: Divide and conquer
description: Notes on divide-and-conquer algorithms.
nav_order: 6
parent: Algorithms
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/algorithms/divide-and-conquer
---

<!-- prettier-ignore-start -->

# Divide-and-conquer
{:.no_toc}

"Veni, vidi, vici. (I came, I saw, I conquered.)"― Julius Caesar

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Divide-and-conquer algorithms involve three steps:

1. **Divide** the problem into smaller subproblems.
2. **Conquer** the subproblems by solving them recursively.
3. **Combine** the solutions to the subproblems into the solution for the original problem.

{% cite clrs -l 65 %}

When a subproblem is large enough to be solved recursively, it's called the **recursive case** {% cite clrs -l 65 %}.

When a subproblem becomes small enough that the algorithm no longer recurses (the recursion "bottoms out"), it is called the **base case** {% cite clrs -l 65 %}.

Merge sort is an example of a divide-and-conquer algorithm.

## References

{% bibliography --cited_in_order %}
