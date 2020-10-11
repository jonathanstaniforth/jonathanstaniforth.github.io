---
layout: post
title: "Notes on Recursive Algorithms"
date: 2020-05-07 20:00:00 +0800
tags: [algorithms,python]
navigation: true
---

Certain problems have characteristics that allow it to be broken down into smaller sub-problems.
This is attractive as the solution to the sub-problems are trivial in comparison to the original problem.
This type of problem can be handled using **recursive algorithms**, which allow for more elegant solutions that would otherwise be complex.
The general structure of recursive algorithms are as follows: break the problem into smaller sub-problems until it cannot be broken any further; then, solve each of the sub-problems and combine their solutions to form the overall solution for the original problem.

This post presents notes on recursive algorithms including the rules of such algorithms, an overview of how they work when implemented, and optimisations using dynamic programming.

## Rules
For an algorithm to be classed as recursive it must follow several rules, shown below:
1. must have a base case
2. must change its state and move towards the base case
3. must call itself recursively

The following sections will discuss each rule in more detail and show how they build on top of each other to achieve the final solution.

### Base case
The base case is the state at which a problem can no longer be broken down into further sub-problems.
At this point, the recursion stops and solutions to the sub-problems can be developed.
Therefore, this rule means that there must be a point at which the algorithm stops breaking the problem into sub-problems.

### Change of state
For the algorithm to start forming a solution, it must reach the base case and can be achieved by changing the algorithms state.
Therefore, this rule requires changing the data which the algorithm is handling so that it can move towards the base case.

### Recursion
Finally, to get the algorithm to perform the movement to the base case it must repeatedly call itself, checking if the base case has been reached and changing the state if necessary.

## Implementation
In practical terms, when a recursive algorithm calls itself, a new **stack frame** is allocated.
This stack frame handles the variables that are local to the algorithm, i.e. creates a scope, and is added to the **call stack**.

> This post focuses on the implementation of recursive algorithms in Python, which may be different in other languages.

The call stack behaves like a stack data structure, with the latest stack frame being at the top of the stack and accessed first (Last-In-First-Out).
Once the latest stack frame has complete, the value returned is left at the top of the call stack for use by the next stack frame, and the solution to the original problem is updated.
Each stack frame is processed until no frames are left on the call stack.

> A recursive algorithm may be implemented as a function, method, or class.

View the full implementation of a [recursive algorithm here](https://github.com/jonathanstaniforth/developer-notes/blob/master/recursion/recursion.py).

## Dynamic programming
When using recursion to solve a problem, the efficiency in producing the solution may reduce to an unacceptable level as the scale of the problem grows.
This inefficiency can be caused by the growing number of recursive calls made to re-compute values, i.e. overlapping sub-problems.
Therefore, remembering past computed values can help to reduce the number of recursive calls made.

In practical terms, this means storing computed values in memory and checking to see if a required value is available before computing, which is known as dynamic programming.
To implement dynamic programming, two systematic approaches can be taken, shown below:
* tabulation
* memoization

### Tabluation
This approach involves starting at the base state and transitioning to the destination state. Therefore, all values, up to the target value, are cumulatively computed, with each value being stored in memory.

> When the sub-problems must be solved at least once, this approach performs better than memoization.

View an example of the [tabulation approach here](https://github.com/jonathanstaniforth/developer-notes/blob/e36ae251a0fcaad23f680bf0533153c53ced1e8e/recursion/dynamic_programming.py#L59).

### Memoization
This approach starts from the destination state and transitions to the base state.
Therefore, all values that are required to establish the final solution are computed and is advantageous when not all of the sub-problems require solving to form the solution.

View an example of the [memoization approach here](https://github.com/jonathanstaniforth/developer-notes/blob/e36ae251a0fcaad23f680bf0533153c53ced1e8e/recursion/dynamic_programming.py#L28).

## Conclusion
In conclusion, recursive algorithms reduce the complexity of a problem by breaking it up into smaller sub-problems and solving them to build the final solution.
For an algorithm to be recursive, it must break a problem down into sub-problems until it reaches a state where it stops and starts building the final solution.
In implementation terms, this means placing a function, method, or class repeatedly onto the call stack, and then solving each of the frames in the stack until none are left.

While recursive algorithms can produce elegant solutions to problems, the efficiency of the algorithm may become unacceptable due to unnecessary re-computation of values.
Dynamic programming can be used to optimise a recursive algorithm by caching computed values for later use.

