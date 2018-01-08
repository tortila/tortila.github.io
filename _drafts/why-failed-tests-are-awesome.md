---
layout: single
title:  "DRAFT: Why failed tests are awesome"
permalink: draft-why-failed-tests-are-awesome
slug: draft-why-failed-tests-are-awesome
comments: true
---
## It's all about information


## Tests that fail randomly
The main problem with failures is when they happen only *sometimes*. This means that there are certain conditions in which the same set of tests can yield different results. There are many ways that non-determinism can be introduced in automated tests (see Fowler's article [^fowler] for details). Until the root cause is identified, such tests are useless. What's more, they make the whole test suite useless - it no longer provides value as a reliable bug detection mechanism.

The problem is common for for all organizations that build and maintain software. Giant such as Google is no exception[^google]:
> Almost 16% of our tests have some level of flakiness associated with them! This is a staggering number; it means that more than 1 in 7 of the tests written by our world-class engineers occasionally fail in a way not caused by changes to the code or tests.

~asda~

There are various ways that this situation can be dealt with:
1. Re-running until all tests pass. Normally a test runner is equipped with mechanism of marking test as flaky, and/or re-running a marked test a couple of times.
2. Quarantine - see Google
3.


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
3. Finally - some fixes to flaky tests (24%) modify the code under test (not the test code), and most of these cases (94%) **fix a bug in the code**.

The first two points make up for an interesting conclusion, that flaky tests can be very easily dealt with. Most of failures show a clear pattern and non-trivial issues are not common.

## Test failures treatment
There are couple of ways failed tests can be dealt with. In the perfect world failed test is treated as a highest priority. Developers (or person responsible) stops doing anything and jumps quickly to investigate failed test and fix its root cause. Unfortunately, for various reasons, this is not always the case. So what are common real-world solutions?
### The good
### The bad
### The ugly
Much like in *The Wire*, my beloved TV series, failed tests are quite often a subject of *juking the stats*. Why bother with test that is useless, when you can simply ignore it, skip it, disable it or remove it from the suite? The outcome is tempting - a green light from CI system, 100% of tests pass, a round of applause for a hero that made it happen.

{% include figure image_path="https://artist.api.lv3.cdn.hbo.com/images/GVU2-Eg7Vl47DwvwIAWY0/detail?size=640x360&format=jpeg&partner=hbocom" alt="source: HBO" caption="A promise of crime rate decrease can be a powerful advantage for candidate running for mayor's office" %}

When the crime rate stats were being manipulated in *The Wire*, a few things could have happened in Baltimore - one of them is, surprisingly, a real decrease in crime (at least in theory)[^thinkprogress]:
> Confidence matters a lot for a city. When people have the sense that conditions in a given city are good and improving, businesses will invest and that creates jobs. And people become more inclined to move in, raising property values. Higher property values mean more tax revenues for social services and for public safety. And more jobs, by all accounts, leads to less crime.

In my experience, cooking up the results of tests just in a pursuit of much desired green light leads to a completely opposite situation. The results are being manipulated by disabling the failing tests. As a result the whole suite becomes a meaningless gate in software integration pipeline. The only reason for their existence is to pass. They no longer provide a valuable information about software under test.

---

[^fowler]: Fowler. *Eradicating non-determinism in tests*.  [https://martinfowler.com/articles/nonDeterminism.html](https://martinfowler.com/articles/nonDeterminism.html)

[^luo]: Luo, Hariri, Eloussi, Marinov (2014). *An Empirical Analysis of Flaky Tests*.  [http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf](http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf)

[^google]: Micco. *Flaky Tests at Google and How We Mitigate Them*.  [https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)

[^thinkprogress]: Yglesias. *Juking the Stats*. [https://thinkprogress.org/juking-the-stats-e9ba45af18e0/](https://thinkprogress.org/juking-the-stats-e9ba45af18e0/)
