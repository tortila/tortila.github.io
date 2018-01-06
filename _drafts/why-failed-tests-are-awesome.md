---
layout: single
title:  "DRAFT: Why failed tests are awesome"
permalink: draft-why-failed-tests-are-awesome
slug: draft-why-failed-tests-are-awesome
comments: true
---

Authors from University of Illinois in *An Empirical Analysis of Flaky Tests* [^1] performed an experiment, in which they searched for the main root causes of intermittently failing tests. They analyzed a number of open source projects from m the Apache Software Foundation, looking for commit messages that indicated a flaky test being fixed. What's more, authors analyzed each occurrence and classified them into 10 different categories, depending on the root cause of flakiness.

Here are my favorite parts:
1. The top 3 root causes of flaky tests are:
  * asynchronous wait (45%) - test execution does not properly wait for asynchronously called result before using it,
  * concurrency (20%) - different threads interacting
in a non-desirable manner,
  * test order dependency (12%) - test outcome depends on the order in which the tests are run.
2. Many (54%) of tests with asynchronous wait issue are fixed using `waitFor`.
3. Almost all (97%) of tests failures with concurrency issue are due to concurrent accesses only on memory objects.
4. Majority (74%) of tests with order dependency issue are fixed by
cleaning the shared state between test runs.
5. Finally - some fixes to flaky tests (24%) modify the code under test (not the test code), and most of these cases (94%) fix a bug in the code.

---
[^1]: Luo, Hariri, Eloussi, Marinov (2014). *An Empirical Analysis of Flaky Tests*  [http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf](http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf)
