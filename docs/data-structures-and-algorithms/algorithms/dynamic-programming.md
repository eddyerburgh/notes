---
layout: default
title: Dynamic programming
description: Notes on graph traversal algorithms.
nav_order: 7
parent: Algorithms
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/algorithms/dynamic-programming
---

<!-- prettier-ignore-start -->

# Dynamic programming
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

DP (dynamic programming) is an optimization method that involves caching earlier results in order to reduce later recomputations. It was developed by Richard Bellman in the 1950s {% cite algorithm-design-manual -l 273 %}.

DP works well for problems that have a left to right ordering, e.g. character strings and integer sequences {% cite algorithm-design-manual -l 274 %}.

There are three steps to solving a problem with DP:

1. Formulate the answer as a recursive algorithm or recurrence relation.
2. Show that the number of different parameter values your recurrence takes is bounded by a polynomial.
3. Specify the order for the recurrence so that the partial results are available to use.

{% cite algorithm-design-manual -l 289 %}

DP is a classic tradeof of space for time {% cite algorithm-design-manual -l 274 %}.

## Fibonacci sequence

A good example of dynamic programming is calculating the Fibonacci sequence.

The Fibonacci sequence is defined:

$$F_0 = 0, F_1 = 1$$

$$F_n = F_n-1 + F_n-2$$

A naive approach would be to calculate the number recursively, using 1 and 0 as the base cases:

```py
def fib(n):
  if n == 0:
    return 0
  if n == 1:
    return 1
  return fib(n - 1) + fib(n - 2)
```

This recursive approach has a time complexity of $$O(2^n)$$. You can see that fib is calculated for the same value multiple times by looking at the call tree:

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/dynamic-programming/fib-call-tree.svg" alt="">
  <figcaption><h4>Figure: The call tree of fib(4) </h4></figcaption>
</figure>

Using DP, you can reduce it to $$O(n)$$ time complexity. Instead of recursively computing the number, each result is stored and used in future calculations:

```py
def fib(n):
  dp = {}
  dp[0] = 0
  dp[1] = 1

  for i in range(2, n + 1):
    dp[i] = dp[i - 1] + dp[i - 2]

  return dp[n]
```

_Note: this approach can be improved further by storing results in two single variables._

## Approximate string matching

Approximate string matching is the problem of matching strings by their **edit distance**. That is, comparing strings by the number of operations are required to transform one string to another.

A simple set of operations include:

- **Substitution**: Replace a single character in string $$S$$ with a different character.
- **Insertion**: Insert a single character into string $$S$$.
- **Deletion**: Delete a single character from string $$S$$.

{% cite algorithm-design-manual -l 280 %}

Operations may have different associated costs depending on the characters involved. For example, replacing _a_ with _s_ might have lower cost than replacing _a_ with _h_, because on a keyboard _s_ is closer to _a_ than _h_ is to _a_.

If the problem is just to count the minimum number of operations required to transform a string, then each operation is given a cost of 1.

The edit distance can be calculated by building a $$n$$x$$m$$ matrix of the cost to transform string $$S$$ to string $$T$$ at each position $$(i,j)$$. Where $$n = \vert S \vert$$ and $$m = \vert T \vert$$.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/dynamic-programming/edit-distance-matrix.svg" alt="">
  <figcaption><h4>Figure: A complete edit distance matrix </h4></figcaption>
</figure>

At point $$(i, j)$$ you need to know the minimum cost operation to make $$(S_0,...,S_i)$$ equal $$(T_0,...,T_j)$$. If results for the positions $$(i - 1, j - 1)$$, $$(i - 1, j)$$, $$(i, j -1 )$$ have already been computed, you can calculate the operation costs as follows:

- Replace is $$d[i - 1, j - 1]$$ if $$S[i] = T[j]$$. Otherwise, the cost is $$d[i - 1, j - 1] + 1$$.
- Insertion is $$d[i - 1, j] + 1$$, because $$i$$ has been advanced but $$j$$ has not.
- Deletion is $$d[i, j - 1] + 1$$, because $$j$$ has been advanced, but $$i$$ has not.

The first step is to calculate the cost from an empty string to string $$S$$ and an empty string to string $$T$$. This will always be $$n$$, where $$n$$ is the length of the substring, because an empty string requires $$n$$ insertions to equal a substring of length $$n$$.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/dynamic-programming/edit-distance-matrix-initial.svg" alt="">
  <figcaption><h4>Figure: The initial edit distance matrix</h4></figcaption>
</figure>

After calculating the base costs, you loop over each empty matrix position and calculate the cost by following the rules listed above. The value at matrix position $$(n,m)$$ is the final edit distance.

```py
def min_distance(word1, word2):
    n = len(word1)
    m = len(word2)

    # Build m x n matrix
    d = [[0] * (m + 1) for _ in range(n + 1)]

    # Initial costs of operations
    for i in range(n + 1):
        d[i][0] = i
    for j in range(m + 1):
        d[0][j] = j

    for i in range(1, n + 1):
        for j in range(1, m + 1):
            replace = d[i - 1][j - 1]
            delete = d[i][j - 1] + 1
            insert = d[i - 1][j] + 1
            if word1[i - 1] != word2[j - 1]:
                replace += 1
            d[i][j] = min(replace, delete, insert)

    return d[i][j]
```

For a detailed explanation, see this MIT [lecture on calculating edit distance](https://youtu.be/ocZMDMZwhCY?t=1457).

## Longest increasing sequence

The longest increasing sequence is the longest monotonically increasing subsequence within a sequence of numbers {% cite algorithm-design-manual -l 290 %}.

_Note: here a sequence is different from a run. Neighbors don't have to be next to each other in a sequence._

In the sequence $$S = \left\{2,4,3,5,1,7,6,9,8\right\} $$, there are 8 monotonically increasing subsequences with length 5, for example $$\left\{2,3,5,6,8\right\}$$.

To determine the longest length of a subsequence at position $$n$$, you need to know:

1. The length of the longest increasing sequence in $$\left\{S_0, S_1, ..., S_n-1\right\} $$.
2. The length of the longest increasing sequence that $$S_n$$ will extend.

{% cite algorithm-design-manual -l 290 %}

Define $$l_i$$ to be the length of the longest sequence ending with $$S_i$$. The longest increasing subsequence at $$n$$ is formed by appending $$S_n$$ to the longest subsequence in $$n-1$$ where $$n_i$$ is less than $$n$$:

$$ l_i = \underset{0 \leq j \leq i}{\max} l_g + 1, where (S_j \lt S_i), l_0 = 0 $$

{% cite algorithm-design-manual -l 290 %}

This can be implemented in $$O(n^2)$$ by calculating the longest subsequence of each previous value:

```python
def lengthOfLIS(self, nums):
    n = len(nums)
    dp = {}

    longest = 0

    for i in range(n):
        l = 0
        for j in range(i):
            if nums[i] > nums[j]:
                l = max(l, dp[j])
            j -= 1
        l += 1
        longest = max(longest, l)
        dp[i] = l

    return longest
```

_Note: there are improved $$O(nlog n)$$ solutions to this problem. For example, see this [Leetcode longest increasing substring solution](<https://leetcode.com/problems/longest-increasing-subsequence/discuss/74824/JavaPython-Binary-search-O(nlogn)-time-with-explanation>)._

## References

{% bibliography --cited_in_order %}
