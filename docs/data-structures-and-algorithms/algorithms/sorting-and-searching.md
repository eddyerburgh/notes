---
layout: default
title: Sorting and searching
description: Notes on sorting and searching algorithms.
nav_order: 2
parent: Algorithms
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/algorithms/sorting-and-searching
---

<!-- prettier-ignore-start -->

# Sorting and searching
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Sorting is the process of arranging data in a predetermined order. For example, arranging an array of integers in ascending order.

A sorted list reduces the amount of work required to solve common problems like:

- Closest pair (which pair of numbers are closest to each other).
- Element uniqueness.
- Frequency distribution.
- Selection (what is the kth largest element in an array).

{% cite algorithm-design-manual -l 104-5 %}

## Merge sort

Merge sort is a recursive divide-and-conquer sorting algorithm. It involves splitting an array into two groups, sorting the groups, and then merging them back {% cite algorithm-design-manual -l 120 %}.

It's made up of two parts: `Mergesort`, and `Merge`:

```
Mergesort(A[1,n])
  Merge(MergeSort(A[1, n/2]), MergeSort(A[ n/2 + 1, n]))
```

{% cite algorithm-design-manual -l 120 %}

The base case for `Mergesort` is when the array length is 1 (hence sorted) {% cite algorithm-design-manual -l 120 %}.

`Merge` works by comparing the first element of each sorted list. The smallest of the two is selected and added to a target array. This process continues until one array is empty, meaning all items from the other array can be added to the target array {% cite algorithm-design-manual -l 120 %}.

Merging two arrays like this uses a third array to store the results, which must be the size of both arrays. Thus, the space complexity of merge sort is $$O(n)$$ {% cite algorithm-design-manual -l 122 %}.

_Note: It's possible to implement merge sort in an array without using much extra storage (see Kronrod's algorithm) {% cite algorithm-design-manual -l 138 %}._

Merge sort runs in $$O(n\lg n)$$ {% cite algorithm-design-manual -l 122 %} in the worst-case.

The following code implements merge sort:

```c
void merge(int arr1[], int n1, int arr2[], int n2) {
  int tmp[n1 + n2];
  int i = 0;
  int j = 0;
  int k = 0;

  while (j < n1 && k < n2) {
    if (arr1[j] < arr2[k]) {
      tmp[i++] = arr1[j++];
    } else {
      tmp[i++] = arr2[k++];
    }
  }

  while (j < n1) {
    tmp[i++] = arr1[j++];
  }

  while (k < n2) {
    tmp[i++] = arr2[k++];
  }

  for (i = 0; i < n1 + n2; i++) {
    arr1[i] = tmp[i];
  }
}

void merge_sort(int arr[], int n) {
  if (n == 1) {
    return;
  }

  int n1 = n / 2;
  int n2 = n - n1;

  merge_sort(arr, n1);
  merge_sort(arr + n1, n2);
  merge(arr, n1, arr + n1, n2);
}
```

{% cite algorithm-design-manual -l 122-3 %}

Merge sort works well for sorting linked lists because it doesn't rely on random access like quicksort or heapsort {% cite algorithm-design-manual -l 122 %}.

## Quicksort

Quicksort is a recursive sorting algorithm.

Quicksort works by first selecting an element $$p$$, known as the pivot, at position $$n$$. It separates the $$n-1$$ other elements into piles: a low pile containing all elements that appear before $$p$$ at the beginning of the array, and a high pile containing all elements that appear after $$p$$ at the end of the array. This leaves a slot for $$p$$, which is $$p$$'s final position in the array {% cite algorithm-design-manual -l 123 %}.

The partitioning step can be completed in one linear scan of the array. After partitioning, the two halves can be partitioned independently because elements on either side of the pivot won’t change sides {% cite algorithm-design-manual -l 123 %}.

In the best case the median value is chosen as the pivot each time. This results in halving the problem on each partition, meaning only $$\lg n$$ partitions are needed to sort the array, so the best case is $$O(n\log n)$$.

In the worst case the pivot is always the smallest or largest element in the array. This requires n partitions to sort the array, leading to a worst case of $$O(n^2)$$.

The average case for quicksort is $$O(n\log n)$$.

You can see an implementation of quicksort:

```c
void quicksort(item_type s[], int l, int h) {
  int p;

  if ((h - l) > 0) {
    p = partition(s, l, h);
    quicksort(s, l, p - 1);
    quicksort(s, p + 1, h);
  }
}

int partition(item_type s[], int l, int h) {
  int i;
  int p;
  int firsthigh;
  p = h;
  firsthigh = l;

  for (i = l; i < h; i++) {
    if (s[i] < s[p]) {
      swap(&s[i], &s[firsthigh]);
      firsthigh++;
    }
  }

  swap(&s[p], &s[firsthigh]);
  return(firsthigh);
}
```

{% cite algorithm-design-manual -l 124 %}

Despite quicksort having worst case $$O(n^2)$$ running time, in practice it is often 2-3 times faster than merge sort or heapsort. This is mainly because the operations of the inner loop in quicksort are simpler {% cite algorithm-design-manual -l 129 %}.

## Heapsort

Heapsort works by constructing a [min-heap]({{ '/data-structures-and-algorithms/data-structures/heap' | relative_url }}), an operation which takes $$O(n\log n)$$ in the worst-case. The next step is to extract the minimum element from the heap using an `extract_min()` function, which is also an $$O(\log n)$$ operation in the worst-case {% cite algorithm-design-manual -l 113 %}.

```c
heapsort(item_type arr[], int n) {
  priority_queue q;
  make_heap(&q, arr, n);

  for (int i = 0; i < n; i++) {
    s[i] = extract_min(&q);
  }
}
```

In total, heapsort has a worst-case running time of $$O(n\log n)$$.

## Sorting in practice

The best way to define how a sorting algorithm behaves (e.g., whether it should sort in descending or ascending order) is with an application-specific **comparison function**.

A comparison function receives pointers to two items and uses the return value to determine which item should be positioned before the other {% cite algorithm-design-manual -l 107 %}.

The standard C library contains the `qsort()` function for sorting, which takes a comparison function:

```c
void qsort(void *base, size_t nel, size_t width, int (*compare) (const void *, const void *));
```

`qsort()` "sorts the first `nel` elements of an array (pointed to by `base`) where the data type is `width`-bytes long" {% cite algorithm-design-manual -l 108 %}.

`compare` determines the order elements are sorted in. It takes two pointers, which point to the elements being compared. It should return a negative integer if the first item belongs before the second, a positive integer if the first item belongs after the second, and zero if they are equivalent {% cite algorithm-design-manual -l 108 %}.

You can see an example that sorts integers in ascending order:

```c
int int_compare(int *i, int *j) {
  if (*i > *j) {
    return 1;
  }
  if (*i < *j) {
    return -1;
  }
  return 0;
}
```

This can be used to sort an array:

```c
qsort(a, n, sizeof(int), int_compare);
```

## Binary search

Binary search is an algorithm for searching a sorted array in $$O(\log n)$$ time {% cite algorithm-design-manual -l 132 %}.

Binary search works by checking the middle of an array for a value. If the item exists in the middle of the array, then binary search can return early. If the middle item is larger than the value, binary search stops searching in the upper half of the array. If the middle item is lower than the value, binary search stops searching in the lower half of the array. This approach halves the array that’s being searched each iteration, meaning binary search runs in worst-case $$O(\log n)$$ {% cite algorithm-design-manual -l 132 %}.

```c
int binary_search(int arr[], int n, int val) {
  int min = 0;
  int max = n - 1;
  int mid;

  while(min <= max) {
    mid = min + (max - min) / 2;
    if (arr[mid] > val) {
      max = mid - 1;
    } else if (arr[mid] < val) {
      min = mid + 1;
    } else {
      return mid;
    }
  }

  return -1;
}
```

_Note: Use `min + (max - min) / 2` to calculate the middle value. This avoids overflow which could occur from using `max + min / 2`._

### Counting occurrences

You can count the number of occurrences of a value using a variation of binary search.

This works by running two binary searches: one to find the position of the first occurrence of a value, and one to find the position of the last occurrence. The difference between the last position and the first position (plus 1) will be the total occurrence {% cite algorithm-design-manual -l 133 %}.

The binary search for finding a minimum or maximum value saves the position of an element when it finds an element that matches the value. Instead of returning, it keeps running to failure, and then returns the last seen position:

```c
int binary_search_max_position(int arr[], int n, int val) {
  int min = 0;
  int max = n - 1;
  int mid;
  int max_position = -1;

  while(min <= max) {
    mid = min + (max - min) / 2;
    if (arr[mid] > val) {
      max = mid - 1;
    } else if (arr[mid] < val) {
      min = mid + 1;
    } else {
      max_position = mid;
      min = mid + 1;
    }
  }

  return max_position;
}
```

## References

{% bibliography --cited_in_order %}
