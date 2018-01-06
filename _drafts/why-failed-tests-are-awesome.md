---
layout: single
title:  "DRAFT: Why failed tests are awesome"
permalink: draft-why-failed-tests-are-awesome
slug: draft-why-failed-tests-are-awesome
comments: true
---

## Tests that fail randomly
The main problem with failures is when they happen only *sometimes*. This means that there are certain conditions in which the same set of tests can yield different results. There are many ways that non-determinism can be introduced in automated tests (see Fowler's article [^fowler] for details). Until the root cause is identified, such tests are useless. What's more, they make the whole test suite useless - it no longer provides value as a reliable bug detection mechanism.

The problem is common for for all organizations that build and maintain software. Giant such as Google is no exception[^google]:
> Almost 16% of our tests have some level of flakiness associated with them! This is a staggering number; it means that more than 1 in 7 of the tests written by our world-class engineers occasionally fail in a way not caused by changes to the code or tests.

~asda~

There are various ways that this situation can be dealt with:
1.


Authors from University of Illinois in *An Empirical Analysis of Flaky Tests* [^luo] performed an experiment, in which they searched for the main root causes of intermittently failing tests. They analyzed a number of open source projects from m the Apache Software Foundation, looking for commit messages that indicated a flaky test being fixed. What's more, authors analyzed each occurrence and classified them into 10 different categories, depending on the root cause of flakiness.

Here are my favorite parts:
1. Majority (77%) of all flaky tests fall into 3 categories:
  * asynchronous wait (45%) - test execution does not properly wait for asynchronously called result before using it,
  * concurrency (20%) - different threads interacting
in a non-desirable manner,
  * test order dependency (12%) - test outcome depends on the order in which the tests are run.
2. There are clear patterns to most of the failures in the top 3 categories:
  * Many (54%) of tests with asynchronous wait issue are fixed using `waitFor`.
  * Almost all (97%) of tests failures with concurrency issue are due to concurrent accesses only on memory objects.
  * Majority (74%) of tests with order dependency issue are fixed by
cleaning the shared state between test runs.
5. Finally - some fixes to flaky tests (24%) modify the code under test (not the test code), and most of these cases (94%) **fix a bug in the code**.

---

[^fowler]: Fowler. *Eradicating non-determinism in tests*.  [https://martinfowler.com/articles/nonDeterminism.html](https://martinfowler.com/articles/nonDeterminism.html)

[^luo]: Luo, Hariri, Eloussi, Marinov (2014). *An Empirical Analysis of Flaky Tests*.  [http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf](http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf)

[^google]: Micco. *Flaky Tests at Google and How We Mitigate Them*.  [https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
