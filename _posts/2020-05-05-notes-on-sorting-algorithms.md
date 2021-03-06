---
layout: post
title: "Notes on Sorting Algorithms"
date: 2020-05-05 12:00:00 +0800
tags: [algorithms]
navigation: true
---

Sorting algorithms can be used to handle several problems, including:
* searching
* selection
* duplicates
* distribution

This post presents notes on the following algorithms, along with links to their implementations:
* [Bubble Sort](#Bubble-Sort)
* [Insertion Sort](#Insertion-Sort)
* [Merge Sort](#Merge-Sort)
* [Quick Sort](#Quick-Sort)
* [Timsort](#Timsort)

An overview of the algorithms, in terms of time and implementation complexity, is shown in the table below:

| Name           | Time Complexity | Implementation Complexity |
|----------------|-----------------|---------------------------|
| Bubble Sort    | O(n^2)          | Simple                    |
| Insertion Sort | O(n^2)          | Simple                    |
| Merge Sort     | O(n log_2 n)    | Intermediate              |
| Quick Sort     | O(n log_2 n)    | Intermediate              |
| Timsort        | O(n log_2 n)    | Complex                   |

## Bubble Sort
Each element in a list is compared to the element to its right and they are swapped if necessary, with the larger valued element moving towards the end of the list.
Multiple passes are made through the list, with each pass resulting in the list moving closer to an order.

When two elements are swapped, also known as exchanged, Python can perform a simultaneous assignment, i.e. the swap can be performed in one statement:

```python
elements[element], elements[element + 1] = elements[element + 1], elements[element]
```

On the other hand, other languages require a temporary variable to hold one value during the swap:

```python
temporary = elements[element]
elements[element] = elements[elements + 1]
elements[element + 1] = temporary
```

Time complexity:
* best: **O(n)** - requires the sorted list check optimisation
* average: **O(n^2)**
* worst: **O(n^2)**

Characteristics:
* simple to implement
* slow to sort

View the full implementation of the [bubble sort algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/5382ba33636a8cfae2950a5ba63dbc9744e93181/sorting/bubble_sort.py#L13).

### Optimisations
The bubble sort can be optimised to prevent extra unnecessary operations being performed.
This optimisation involves checking for whether the list has reached an ordered state and terminating, and is referred to as the **short bubble sort**.

View the optimised implementation of the [bubble sort algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/5382ba33636a8cfae2950a5ba63dbc9744e93181/sorting/bubble_sort.py#L46).

## Insertion Sort
Each element in a list is taken out and compared against the elements to its left until its correct position in the list is found, at which point it is inserted back into the list.
The elements that are left of the element that is currently being sorted are ordered, with this order being maintained throughout the sort.
All elements in the ordered sublist that are greater than the element being sorted can be moved to the right, as the sorted element is first taken out of the list leaving a free slot.

Time complexity:
* best: **O(n)** - when the list is already sorted
* average: **O(n^2)**
* worst: **O(n^2)**

Characteristics:
* simple to implement
* more efficient than quadratic algorithms, such as the bubble sort
* not efficient for large datasets

View the full implementation of the [insertion sort algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/master/sorting/insertion_sort.py).

## Merge Sort
This algorithm uses recursion to sort a list of unordered elements.
In particular, the divide-and-conquer approach is used, which has the following structure:
* break the problem into smaller sub-problems
* solve each sub-problem using recursion
* combine the solution for each solved sub-problem to form the solution

The merge sort algorithm repeatedly splits a list of elements in half until only one element is left in each half, at which point each half is considered sorted.
After splitting the list, the next step is to combine the halves to form incrementally larger ordered lists.

Time complexity:
* best: **O(n log_2 n)**
* average: **O(n log_2 n)**
* worst: **O(n log_2 n)**

Characteristics:
* an efficient algorithm that scales to large datasets
* can be parallelised as it breaks the input into smaller chunks
* time penalty due to recursion which does not perform well with smaller datasets
* larger memory requirement, in comparison to other algorithms, due to copying lists

View the full implementation of the [merge sort algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/master/sorting/merge_sort.py).

## Quick Sort
Quick sort is also a recursive algorithm that utilises the divide-and-conquer approach.
First, the unordered list is repeatedly split into two lists, where one holds lower-valued elements and the other holds higher valued elements.
This split is known as partitioning and is implemented by selecting a pivot value, where the position of the pivot value in the list is called the split point.
Second, the placement of elements into the two lists is performed, and the two lists are merged until the recursion is complete.

Time complexity:
* best: **O(n)**
* average: **O(n log_2 n)**
* worst: **O(n^2)**

> The worst case can be set to **O(n log_2 n)** if the pivot value is selected by finding the median value in the list. But, in practice, implementing this only has benefits when handling large datasets.

Characteristics:
* an efficient sorting algorithm
* can be parallelised due to it breaking the input into small chunks
* cannot guarantee the average time complexity of **O(n log_2 n)**
* requires larger amounts of memory due to it creating new lists
* not efficient for smaller datasets

View the full implementation of the [quick sort algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/master/sorting/quick_sort.py).

## Timsort
This sorting algorithm utilises both the insertion and merge sorting algorithms, resulting in a hybrid approach to ordering a list.
It takes advantage of **natural runs**, i.e. elements that are already sorted.
These runs are collected and then merged to form the final solution.

Time complexity:
* best:  O(n) - lists that are close to ordered
* average: O(n log_2 n)
* worst: O(n log_2 n)

Characteristics:
* complex algorithm to implement
* performs well with small and large datasets
* time complexity is fairly consistent

View the full implementation of the [timsort algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/master/sorting/timsort.py).
