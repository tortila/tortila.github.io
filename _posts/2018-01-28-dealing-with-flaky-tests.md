---
layout: single
title:  "Dealing with flaky tests"
date:   2018-01-28 20:00:00 +0100
permalink: dealing-with-flaky-tests
tags: testing
slug: dealing-with-flaky-tests
comments: true
header:
  image: assets/images/tim-gouw-128115.jpg
---
## Embrace the failure
I love failed tests, I really do. The sight of a red light on continuous integration system on Monday morning is my favorite way to start a week. Whenever I spot a failed test, I'm thrilled - mostly because it means there's something I'm going to learn. There's something that went wrong, something that got broken, or flaw that was hidden until now just resurfaced. And there's no best way to learn about it than from tests!

The core value of tests is all about information. Successful results give you a very simple piece of information: *all is fine*. But as far as all tests are passing, the only value that tests are presenting is *reassurance*. We all need a pat on the back from time to time, but the downside of it is that it never comes with a lesson. If you strive to improve (your software, your tests, your process, your CI pipeline, your testing environment) you need to have an opportunity to fail. Failed test, if treated properly, can give you information about all these subjects.

Have you ever disabled any tests? If so, how do you know that the bug that caused it to fail is still in your software? On the other hand, have some of your tests ever fail? If not, how do you know that they check what they're supposed to?

## Flaky tests - 80% of the time, it works every time
The main problem with failures is when they only happen sometimes. This means that there are certain conditions in which the same set of tests can yield different results. There are many ways that non-determinism can be introduced in automated tests (see [Fowler's article](https://martinfowler.com/articles/nonDeterminism.html) for details). Until the root cause is identified, such tests are dangerous. They make the whole test suite unreliable - it no longer provides value as a bug detection mechanism. And what's more important, they create a lot of frustration, which may lead to even more dangerous events.

The problem is common for for all organizations that build and maintain software. Giant such as Google is [no exception](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html):
> Almost 16% of our tests have some level of flakiness associated with them! This is a staggering number; it means that more than 1 in 7 of the tests written by our world-class engineers occasionally fail in a way not caused by changes to the code or tests.

The numbers are too high to be ignored, so when confidence in value of tests is at stake, the only reasonable solution is to try and solve the burning problem.

## Have you tried turning it off and on again?
There are various ways that the situation described above can be dealt with:
1. Re-running until all tests pass. Normally any modern test runner is equipped with mechanism of marking test as flaky, and/or re-running any marked test a couple of times. Such logic can also be implemented additionally.
2. Quarantine mechanism - contains of two phases: flakiness monitoring system and a sandbox for tests that were detected as flaky. The crucial part is to remove flaky test from the main path of integration/delivery pipeline and move them to a quarantine. Such tests are still executed, but their result is not taken into consideration by quality gates. This solution is described in greater detail on [Google blog](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html).

The examples above are just partial solutions. They won't scale as the number of tests increases. There will be just more flaky tests, and more re-runs, more time need to integrate a piece of code, and even more frustration. Both solutions lack a *mitigation* part.

Let's take a look at quarantine example - as long as there's no additional mechanism for detecting the root cause of flakiness (let alone fixing it automatically), there needs to be a person dedicated for that job. Nevertheless, with large amount of tests, this seems to be the most sustainable solution. Re-running alone, without any mitigation step, is only a temporary treatment, and should be treated as last resort. In both cases, without root cause analysis for each and every failure, tests value will be decreasing significantly with time.

## Many faces of flakiness
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

The first two points make up for an interesting conclusion: flaky tests can be very easily dealt with. Most of failures show a clear pattern and non-trivial issues are not common. But a true revelation struck me in the third conclusion - more than 1 in every 5 flaky tests catches and exposes a bug in software! These tests do their job, they do tell you about bug - but they need some special attention. Either because the nature of the bug is complex (maybe it occurs only under very special circumstances?), or maybe because the tests are. Whichever it is, by investigating it, you'll learn a whole lot about both your application and your tests.

## Testing is hard
The biggest turning point in my approach to any kind of automated tests was when I understood that it's just another product. Like any piece of software, it requires maintenance, otherwise it's going to break. It's as fragile as the software under test, and as the environment that it runs on. To tell the truth, I never aim to build a bullet-proof tests when I first create them - I know that they are subject to dynamic so strong that some kind of breakage is inevitable. Especially during early stages of product development.

But when I'm maintaining the tests, I do my best to analyze and drill down the exact issue that caused the failure, and fix for good (hopefully). As a trivial example, when I spot an issue regarding asynchronous wait on some resource, the easiest fix is to increase the timeout. This is not a stable solution, because there's no guarantee the resource will be always on time. A better solution in this particular case is conditional wait for resource with polling (mentioned `waitFor`).

With this approach, I have room for failure, so I can learn about: code, test, test environment, infrastructure, some specific language quirks and more[^quirks]. It's sustainable as long as the cause of failures is diagnosed quickly. It doesn't mean that it has to be fixed immediately - it's enough to have a plan for fixing along with priority. Regardless of the root cause, it's great to have a quarantine mechanism in place. An isolated environment where both flaky tests and those that discovered a known failure can keep giving you information, without decreasing the speed or reliability of your software delivery.

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
