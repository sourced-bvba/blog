---
layout: post
title: "To our managers: why writing tests actually enhances productivity"
comments: true
categories: [java]
---

Writing extra code in the form of tests at first seems contradictory to enhancing productivity. At least it is perceived as such by managers. Extra code means extra time, and extra time means either delays, higher costs or probably both. A developer trying to introduce test-driven development will have a hard time convincing the people in suits of the merits of it. There are several reasons to this:

1. Most of their software was written without test-driven development, and it works
2. Once software works to a degree management is happy, the problems that occurred during the development of that piece of software is somewhat forgotten
3. Selling software driven development is selling a concept. It’s hard to visualize the benefits of TDD without getting into technical details. And management hates technical details
4. Let’s face it, techies aren’t the best of salesmen...

But there must be some truth in all the hype, mustn’t there? Well, let me try to see at the benefits of TDD through a manager’s view.<!--more-->

## Green means go
One of the hardest things to decide is whether a product is ready to ship. There may be several factors playing in this decision process: an impeding deadline, a commitment towards the client, marketing, … But when can you decide whether a product is of such quality you’re willing to hand it over to your paying clients? “Well, we have a QA department for that”, managers might say. And they are right. Somewhat. For a developer, a QA department is of little use during active development. QA usually checks entire scenarios or user stories. But by the time something gets to QA, valuable time is lost.
That’s where automated tests come into play. QA’s task should not be testing functionality, they should actively specify that functionality and if during a review of that functionality something is lacking, refine that specification so it cover that what was lacking. Those specifications can then be converted to tests. Initially those tests will fail, which is a good thing. Now developers have a target they can work towards. They write just enough code to pass the tests. If all tests succeed (the green bar in JUnit), the product has met all the specifications at that time and can be deployed. There are now several advantages: First, QA plays an active part in development, providing (evolving) specifications. They no longer have to do the tedious testing work, as it is done by the computer. It also means that if a released product is lacking some functionality, it’s probably because they didn’t provide a specification. Second and more important, this makes a developer very productive. All work is done with one goal in mind: passing the tests, or in other words, implementing specified functionality. And last but not least, if all tests pass, in theory you can ship that piece of software. Green means stable (not complete, a product is complete when all specifications are written and passed) and safe. Need 2 week releases? It’s doable with TDD.

## Bugs don’t reappear
If a bug is found, either the specification was lacking in which case QA needs to alter it, or it’s introduced by a developer. In any case, tests are written proving the bug and will fail at first. Again, developers aim to pass the tests and fix the bug so that the bar goes green again, resulting once again in a shippable product. And you’re sure that specific bug won’t reappear several versions in the future, as it is covered through an automated test.

## You have a measure of control over your less-talented developers
Yes, it’s hard to admit that your team doesn’t consist solely out of rockstar developers. You asked for the best, so you have the best, right. Sadly, the reality is never that great and most of the time, you do have developers on your team who create an occasional ‘WTF’. This is where tests become crucial to overall project velocity: have your rockstar developers (or at least the better ones) create tests for the junior ones in order to have a standard on which their work can be measured. This serves a dual purpose: your tests will be of good quality and the code in your system will have higher quality, even if it’s produced by a less-than-competent developer. It probably won’t be great code, but it’ll at least be code that is usable.

## Eat your own dogfood
When writing tests, they will be the first to consume the API. If writing tests using the API is hard, other consumers will think the same. Writing tests results in better API’s.

Last but not least, just think about the amount of graphs you can produce when using tests: code coverage, test failures vs test successes, test failure turnover rate, ... So: have your developers start writing tests. Now.