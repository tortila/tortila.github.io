---
layout: single
title:  "DRAFT: How I learned to stop worrying and love failing tests"
permalink: draft-why-failed-tests-are-awesome
slug: draft-why-failed-tests-are-awesome
comments: true
---
### Embrace the failure
I love failed tests. The sight of a red light on continuous integration system on Monday morning is my favorite way to start a week. Whenever I spot a failed test, I'm thrilled - mostly because it means there's something I'm going to learn. There's something that went wrong, something that got broken, or flaw that was hidden until now just resurfaced - and I have been given a chance to solve the riddle and make the world a better place.

The core value of tests is all about information. Successful results give you a very simple piece of information: "All is fine". If you live by the motto "No news is good news", you might be satisfied with it. But in this scenario the only value that tests are presenting is *reassurance*. We all need a pat on the back from time to time, but the downside of it is that it never comes with a lesson. If you strive to improve (your software, your tests, your process, your CI pipeline, your testing environment) you need to have an opportunity to fail. Failed test, if treated properly, can give you information about all these subjects.

But how do you know that your tests do what you expect them to do if they never fail? Or how do you know if the bug, that your tests discovered some time ago, is still in your software if you disabled the test (because it kept failing and it was so frustrating)?

### Flaky tests. 80% of the time, it works every time
The main problem with failures is when they only happen sometimes. This means that there are certain conditions in which the same set of tests can yield different results. There are many ways that non-determinism can be introduced in automated tests (see [Fowler's article](https://martinfowler.com/articles/nonDeterminism.html) for details). Until the root cause is identified, such tests are useless. What's more, they make the whole test suite useless - it no longer provides value as a reliable bug detection mechanism. They may also be dangerous. And finally, they create a lot of frustration.

The problem is common for for all organizations that build and maintain software. Giant such as Google is [no exception](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html):
> Almost 16% of our tests have some level of flakiness associated with them! This is a staggering number; it means that more than 1 in 7 of the tests written by our world-class engineers occasionally fail in a way not caused by changes to the code or tests.

The numbers are too high to be ignored, so when confidence in value of tests is at stake, the only reasonable solution is to try and solve the burning problem.

### Have you tried turning it off and on again?
There are various ways that the situation described above can be dealt with:
1. Re-running until all tests pass. Normally any modern test runner is equipped with mechanism of marking test as flaky, and/or re-running any marked test a couple of times. Such logic can also be implemented additionally.
2. Quarantine mechanism - contains of two phases: flakiness monitoring system and a sandbox for tests that were detected as flaky. The crucial part is to remove flaky test from the main path of integration/delivery pipeline and move them to a quarantine - so that they are still executed, but their result is not taken into consideration by quality gates. This solution is described in greater detail on [Google blog](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html).

The examples above are just partial solutions. They won't scale as the number of tests increases. There will be just more flaky tests, and more re-runs, more time need to integrate a piece of code, and even more frustration. Both solutions lack a *mitigation* part.

Let's take a look at quarantine example - as long as there's no additional mechanism for detecting the root cause of flakiness (let alone fixing it automatically), there needs to be a person dedicated for that job. Nevertheless, with large amount of tests, this seems to be the most sustainable solution. Re-running alone, without any mitigation step, is only a temporary treatment, and should be treated as last resort. In both cases, without root cause analysis for each and every failure, tests value will be decreasing significantly with time.

### Many faces of flakiness
Authors from University of Illinois in [*An Empirical Analysis of Flaky Tests*](http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf) performed an experiment, in which they searched for the main root causes of intermittently failing tests. They analyzed a number of open source projects from m the Apache Software Foundation, looking for commit messages that indicated a flaky test being fixed. What's more, authors analyzed each occurrence and classified them into 10 different categories, depending on the root cause of flakiness.

Here are my favorite parts:
1. Majority (77%) of all flaky tests fall into 3 categories:
  * asynchronous wait (45%) - test execution does not properly wait for asynchronously called result before using it,
  * concurrency (20%) - different threads interacting in a non-desirable manner,
  * test order dependency (12%) - test outcome depends on the order in which the tests are run.
2. There are clear patterns to most of the failures in the top 3 categories:
  * Many (54%) of tests with asynchronous wait issue are fixed using `waitFor`.
  * Almost all (97%) of tests failures with concurrency issue are due to concurrent accesses only on memory objects.
  * Majority (74%) of tests with order dependency issue are fixed by
cleaning the shared state between test runs.
3. Finally - some fixes to flaky tests (24%) modify the code under test (not the test code), and most of these cases (94%) **fix a bug in the code**.

The first two points make up for an interesting conclusion: flaky tests can be very easily dealt with. Most of failures show a clear pattern and non-trivial issues are not common. But a true revelation struck me in the third conclusion - more than 1 in every 5 flaky tests catches and exposes a bug in software! These tests do their job, they do tell you about bug - but they need some special attention. Either because the nature of the bug is complex (maybe it occurs only under very special circumstances?), or maybe because the tests are.

### Testing is hard
The biggest turning point in my approach to any kind of automated tests was when I understood that it's just another product. Like any piece of software, it requires maintenance, otherwise it's going to break. It's as dynamic as the software under test, and as the environment that it runs on. To tell the truth, I never aim to build a bullet-proof tests when I first create them - I know that they are subject to dynamic so strong that some kind of breakage is inevitable.

On the other hand, when I'm maintaining them, I always try to analyze and drill down the exact issue that caused the error. As an example, when I spot an issue regarding asynchronous wait on some resource, the easiest fix is to increase the timeout. This is not a stable solution, because there's no guarantee the resource will be always on time. A more sustainable solution in this particular case is conditional wait for resource with polling (mentioned `waitFor`).

With this approach, I have room for failure, so I can learn about: code, test, test environment, infrastructure, some specific language quirks and more[^quirks].

---

[^quirks]: For example, I once learned the hard way the difference between `==` and `is` operators in Python, and how it caches small integer objects, when my tests (which used `is` for comparing two integers) were passing for small values and failing for bigger ones. See for yourself:
```python
>>> a = 256
>>> b = 256
>>> a == b
True
>>> a is b
True
>>> a = 257
>>> b = 257
>>> a == b
True
>>> a is b
False

```
 Of course, the value in my tests was calculated dynamically, depending on the state of database - so the test was flaky!
