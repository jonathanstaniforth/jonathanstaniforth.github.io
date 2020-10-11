---
layout: post
title: "Notes on Algorithms"
date: 2020-05-11 20:00:00 +0800
tags: [algorithms]
---
An **algorithm** is a defined procedure to accomplish a specific task.
It must accept input, also known as instances, and transform the input to achieve the desired output.
The transformation applied by an algorithm is defined by a series of steps which it must follow.
Often, algorithms are used to solve **algorithmic problems** which can be formally defined as follows:

```
Problem: <description of the problem>
Input: <data provided to the algorithm>
Output: <what the algorithm must produce>
```

When creating an algorithm, several goals should be achieved:
* correctness - always produces the correct output
* efficiency - produces the output with the least amount of computation
* ease - trivial to implement

> It may not always be possible to achieve all these goals simulatenously and, therefore, a balance between them may be required.

## Heuristics
**Heuristics** are used in place of algorithms when it is not possible to achieve the correct output as they can provide approximations.
Different heuristics will provide output with varying degrees of **correctness** and, therefore, the heuristic used for a problem should be selected carefully so that it matches as closely as possible.

## Algorithm correctness
To establish the correctness of an algorithm, a goal discussed above, **proof** is required.
To show proof, the following is required:
* proof statement - statement on what is being proven
* assumptions
* chain of reasoning - the process from assumptions to the proof statement
* QED - shows that the proof has finished, "thus it is demonstrated"

The following sections will discuss how to achieve each of the parts shown above.

### Proof statement
To define the proof statement, i.e. describe the problem, the following parts must be stated:
* the set of allowed input
* properties required from the ouput

Two traps are often encountered when defining the proof statement:
* ill-defined problems - ambiguous words are used in the statement
* compound goals - multiple goals are selected

> A technique to achieving a correct and efficient algorithm is to reduce the set of allowed input.

### Chain of reasoning
Several notation methods can be used to show the sequence of steps taken by an algorithm, i.e. show the chain of reasoning.
The different notation methods are the following:
* English
* pseudocode
* programming language

These methods describe an algorithm in different ways, i.e. different languages, and have varying degrees of accuracy and ease of understanding.
While programming languages have the highest accuracy, they can be the most difficult to understand.
On the other hand, English can be easily understood, but, the words used may introduce ambiguity and, therefore, reduce the accuracy of the description.
When selecting a notation method, a decision between the level of accuracy and understanding needs to be made.

### Proof of incorrectness
**Counter-examples** are inputs that result in an algorithm producing incorrect output and have the following properties:
* verifiability
  * show the output produced by the algorithm
  * show the correct answer for the given input
* simplicity - remove all unnecessary details

When searching for weaknesses in an algorithm, first look at the problem in detail and think about what issues/difficulties may occur when solving.
Second, systematically cover all possible inputs, including small, extreme, and tie input.

> If incorrectness is not found in an algorithm, proof of the correctness of the algorithm is still required.

### Proof of correctness
**Mathematical induction** is a proof technique which can be used as proof of the correctness of an algorithm.
It involves proving a base case and then assuming it is correct up to a value.
Induction has the following steps:
* proved a formula for a base case, such as 1
* assume the formula is correct up to ```n - 1```
* show the formula is correct for ```n```

**Recursion** is similar to mathematical induction and has the following steps:
* the recursive case breaks the problem down into smaller sub-problems
* the base case stops the recursion and returns a value
* show that the algorithm is correct for the case ```n```

## Modeling a problem
Algorithms are designed to work on rigorously defined abstract data structures, discussed below.
Therefore, when describing a problem, it must be done abstractly, such that it becomes a sequence of transformations on abstract data structures.
The following sections discuss the different abstract data structures that are available.

#### Combinatorial objects
The following are common data structures:
* permutations - arrangements/orderings of items, e.g. {1, 3, 2, 4}
* subsets - a selection of items from a set of items
* trees - hierarchical relationships between items
* graphs - relationships between arbitrary pairs of objects
* points - locations in a geometric space
* polygons - regions in a geometric space
* string - sequences of characters/patterns

#### Recursive objects
Recursion involves the combination of a set of smaller items, resulting in a larger item.
The abstract data structures discussed above can be handled recursively, shown below:
* permutations - removing an element results in a permutation with ```n - 1``` elements
* subsets - removing an element in subset results in a subset of ```n - 1``` elements
* trees - the combination of nodes, smaller items, results in a larger tree, larger item
* graphs - the combination of nodes results in a larger graph
* points - points can be combined into a larger set or broken down into smaller sets
* polygons - similar to the points object
* strings - the characters of a string can be combined to produce a larger string

Recursive objects require the decomposition rules, and base cases.
Decomposition rules describe how a recursive object may be broken down into a smaller object, and is known as the [recursive case](#Proof-of-correctness).
Base cases define the smallest object that can be achieved from a recursive object, and results in the decomposition of the object terminating.

> Further discussion about recursion can be found on my post [Notes on Recursive Algorithms]({% link _posts/2020-05-07-notes-on-recursive-algorithms.md %})

## Conclusion
This post briefly looked at algorithms, firstly what they are and how they can be used to solve problems by transforming given input to a required output.
Secondly, how to prove the correctness of an algorithm was discussed, including tools and techniques that can be used.
Finally, the reasoning for modeling a problem abstractly so that an algorithm can perform transformations on abstract data structures was presented.

These notes where prepared from the book "The Algorithm Design Manual" by Steven S. Skiena.
This book provides an excellent starting point for understanding algorithms and how they work.
You can find the book over at the [Springer website](https://www.springer.com/us/book/9781848000698).
