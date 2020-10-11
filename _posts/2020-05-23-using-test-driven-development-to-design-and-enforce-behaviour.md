---
layout: post
title: "Using Test-Driven Development to Design & Enforce Behaviour"
date: 2020-05-23 14:30:00 +0800
tags: [testing]
---
There is some speculation as to the origin of **test-driven development** with some saying it started in the [1940s or 50s](https://www.quora.com/Why-does-Kent-Beck-refer-to-the-rediscovery-of-test-driven-development-Whats-the-history-of-test-driven-development-before-Kent-Becks-rediscovery).
Also, it has been closely linked with **test-first development**, which is a concept of [**extreme programming**](http://www.extremeprogramming.org) that started in 1996 and formalised in 2000 with the release of the book Extreme Programming Explained: Embrace Change by Kent Beck.
However, its formal introduction can be considered in 2002 with Kent Beck releasing another book titled Test-Driven Development: By Example.

Since test-driven development has been around a long time, a lot of information is available.
However, while researching this topic I found that there is some discrepancy about the goals of test-driven development, with some sources focusing on the testing and others on the design of production code.
Therefore, this post will examine this discrepancy, the **design methodology**, and the perspective that should be taken when undertaking this approach.

## A misunderstanding
Many people believe that test-driven development is a **testing methodology**, i.e. a systematic approach to checking production code.
This is likely due to the word test being in the name and its approach being centred around establishing tests.
Therefore, many people view it as a weight that gets in the way of developing and completing a project.
As a result, it is often sidelined until after development has complete, at which point it is difficult to implement and forgotten altogether.

However, this negative view is due to a misunderstanding of what test-driven development is trying to achieve.
Rather than it being a [methodology for testing, it is a methodology for design](https://wiki.python.org/moin/TestDrivenDevelopment), where a test is used to define the desired **behaviour** and enforce that behaviour throughout the lifetime of the production code so that you have confidence in its actions.
Additionally, some have equated test-driven development to [**test-first development** with **refactoring**](https://tcagley.wordpress.com/2016/07/26/test-first-and-test-driven-development-is-there-a-difference/), as it enables improvements in the design of production code over time, making it an evolutionary process.
Below looks at how to undertake this design approach by first showing the general principles and then detailing the steps involved.

## The design methodology
### Rules
[Uncle Bob](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd) condensed the approach of test-driven development to three rules:

1. you are not allowed to write any production code unless it is to make a failing unit test pass;
2. you are not allowed to write any more of a unit test than is sufficient to fail; and compilation failures are failures;
3. you are not allowed to write any more production code than is sufficient to pass the one failing unit test.

[Javier Trevino Saldana](http://www.javiersaldana.com/articles/tech/refactoring-the-three-laws-of-tdd) reduced these rules further to the following:

1. write only enough of a unit test to fail;
2. write only enough production code to make the failing unit test pass.

These rules are clear and concise, start by establishing a test, then write production code to fulfil the test.
However, these rules do not explain the design aspect of test-driven development, therefore, let's go into more detail about the methodology.

### Red, green, refactor cycle
The [red, green, refactor cycle](https://www.jamesshore.com/Agile-Book/test_driven_development.html) is often mentioned as the approach to test-driven development.
This cycle is a series of sequential phases that are repeated until all required behaviour has been implemented.

#### Red - defining the behaviour
The **red phase** relates to Javier's first rule, writing a test, and involves a similar approach to [establishing the correctness of an algorithm]({% link _posts/2020-05-11-notes-on-algorithms.md %}).
During this phase, the perspective should be from a user interacting with an application programming interface and the behaviour, or requirement, they would like from it.
[James Shore](https://www.jamesshore.com/Agile-Book/test_driven_development.html) breaks this into a separate phase called "think", which places more emphasis on the design of the production code.
The steps involved in this phase are the following:

1. proof statement;
2. assumptions;
3. chain of reasoning;
4. [QED](https://en.wikipedia.org/wiki/Q.E.D.).

> "This is the phase where you design how your code will be used by clients." - [freeCodeCamp](https://www.freecodecamp.org/news/test-driven-development-what-it-is-and-what-it-is-not-41fa6bca02a2/)

Therefore, to clearly define the desired behaviour, the following should be established: the behaviour; the allowed input instances; and, the expected output.
Additionally, any assumptions that are being made should be stated.

Now that the behaviour has been described in detail, the chain of reasoning can be shown by writing a test in a programming language.
The test should make a series of expectations, i.e. checks, from the input instances to the expected output to state the required actions of the behaviour, making sure all potential scenarios are covered.

Once the test has been completed the behaviour has been established and, therefore, step 4 is complete.

> Remember Javier's first rule, only write enough of a test to fail

#### Green - implement the behaviour
Now that the required behaviour is known and a test is in place to enforce it, the development of production code can start.
Only enough code should be developed to pass the test so that Javier's second rule is complied with.
During this phase, [best practices and elegance can be ignored](https://www.jamesshore.com/Agile-Book/test_driven_development.html) as the focus is on passing the test.

> "you need to act like a programmer who has one simple task: write a straightforward solution that makes the test pass" [freeCodeCamp](https://www.freecodecamp.org/news/test-driven-development-what-it-is-and-what-it-is-not-41fa6bca02a2/)

Additionally, a to-do list of improvements and optimisations should be kept for the refactor phase. This can be kept in line with the production code as follows:

```python
# TODO - Optimise
```

#### Refactor - optimise the code
The refactor phase can be used to clean any production code written during the last green phase, while also maintaining the test status.
Additionally, any duplicate code should be removed.

> "you play the picky programmer who wants to fix the code to bring it to a professional level." [freeCodeCamp](https://www.freecodecamp.org/news/test-driven-development-what-it-is-and-what-it-is-not-41fa6bca02a2/)

Often the removal of duplicate code involves abstraction, e.g. moving code to a helper function.
Keep in mind that a new test should not be started until this phase has been completed.

After finishing this phase, the cycle can be repeated until all necessary behaviour has been implemented.

## Levels of test-driven development
Test-driven development can be extended to feature [two levels](http://www.agiledata.org/essays/tdd.html): acceptance test-driven development; and, developer test-driven development.

### Acceptance
This level involves taking a higher-level view of the required behaviour and writing a test, following the process discussed above.
Once the test is in place, the developer level can be started.

### Developer
The developer level moves the focus to a lower level and on smaller behavioural requirements, again following the steps stated above.

When the red, green, refactor cycle has been completed for this level, the developer can move back to the acceptance level and complete the cycle.
If necessary, the developer may need to move back down to the developer level.

## Conclusion
To summarise, test-driven development is an approach to software development that allows for the clear definition of required behaviour from production code and the enforcement of this behaviour throughout its lifetime using tests.
This approach, i.e the red, green, refactor cycle, starts by taking the view of a user interacting with an application programming interface and having a behavioural requirement.

During the red phase, a test is created which defines the behaviour, the range of input instances, and the expected output.
The green phase changes the perspective to a developer focusing on producing code to make the test pass, while not worrying about good coding practice.
With the final phase, refactoring, the developer changes focus again to optimising the implementation of the behaviour, while keeping the test green.

This approach can be summarised with two rules: write only enough of a unit test to fail; and, write only enough production code to make the failing unit test pass.
This makes test-driven development evolutionary in nature, with the cycle repeating until all desired behaviour has been implemented.

Test-driven development can be expanded to include several levels called acceptance and developer.
The acceptance level looks at the higher-level behaviour of the code, which may encompass several smaller behaviours found at the developer level.
This all adds to the confidence in which developers have in the actions taken by production code when following the process of test-driven development.

