---
layout: single
title:  "Dealing with flaky tests"
date:   2018-01-28 20:30:00 +0100
permalink: dealing-with-flaky-tests
categories: ["Software Engineering"]
slug: dealing-with-flaky-tests
header:
  overlay_image: assets/images/tim-gouw-128115.jpg
  og_image: assets/images/tim-gouw-128115.jpg
  overlay_filter: 0.5
  show_overlay_excerpt: false
---
## Embrace the failure
If there is one thing I love about looking at the results of any kind of tests, it's definitely seeing them fail. The sight of a huge red light on the CI dashboard on Monday morning is my favorite way to start the week. Whenever I spot a failed test, I'm thrilled - mostly because it means there's something I'm going to learn. There's something that went wrong, something that got broken, or a flaw that was hidden until now has just resurfaced. And there's no better way to learn about it than from failed tests!

The core value of any kind of tests is all about feedback loop. Successful results give you a very simple piece of information: *all is fine*. But as far as all tests are passing, the only value that the tests are presenting is the *reassurance*. Everyone needs a pat on the back from time to time, but the downside of it is: it never comes with a lesson. If you strive to improve (your software, your tests, your process, your CI pipeline, your testing environment) you need to have an opportunity to fail. Failed test, if treated properly, can give you information about all these subjects.

I'm pretty sure I'm not the only one guilty of disabling the tests when they fail. I understand it as a last resort, but still feel like I'm committing a sin by `@Ignore`ing a test that I can't deal with at the moment. Living by the rules is one thing, but delivering software is more complex than that. By disabling the test, I intentionally shut my eyes to the information that they've been providing so far - information about something broken. And if I already know what's broken, and decide not to fix it *right now*, all I'm losing is the same error log, repeated on the console, and information that it hasn't been fixed yet. Well, as long as the test fails always and only because of the same root cause. But how about tests that sometimes fail, and sometimes pass? Should we have more or less mercy for them? How useful are they anyway?

{% include figure image_path="https://calvinandhobbesagain.files.wordpress.com/2012/11/ch861125.jpg" caption="Calvin, my favourite character from a comic book, surely knows how to embrace the failure" %}{: .align-center}

## 80% of the time, it works every time
As [Fowler](https://martinfowler.com/articles/nonDeterminism.html) puts it:
 > A test is non-deterministic when it passes sometimes and fails sometimes, without any noticeable change in the code, tests, or environment. Such tests fail, then you re-run them and they pass.

Sometimes these tests are being referred to as *flaky*[^randomly]. There are many ways that non-determinism can be introduced in automated tests - the more complex setup, the more factors can play the role. Until the root cause is identified, such tests are quite dangerous. They make the whole test suite unreliable - it no longer provides value as a bug detection mechanism. And what's more important, they create a lot of frustration, which may lead to even more dangerous events.

The problem is common for all organizations that build and maintain software. Giant such as Google is [no exception](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html):
> Almost 16% of our tests have some level of flakiness associated with them! This is a staggering number; it means that more than 1 in 7 of the tests written by our world-class engineers occasionally fail in a way not caused by changes to the code or tests.

The numbers are too high to be ignored, so when confidence in value of tests is at stake, the only reasonable solution is to try and solve the burning problem.

## Have you tried turning it off and on again?
There are various ways that the situation described above can be dealt with:
1. Re-running until all tests pass. Normally any modern test runner is equipped with mechanism of marking test as flaky, and/or re-running any marked test a couple of times. Such logic can also be implemented additionally.
2. Quarantine mechanism - contains of two phases: flakiness monitoring system and a sandbox for tests that were detected as flaky. The crucial part is to remove flaky test from the main path of integration/delivery pipeline and move them to a quarantine. Such tests are still executed, but their result is not taken into consideration by quality gates. This solution is described in greater detail on [Google blog](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html).

The examples above are just partial solutions - they won't scale as the number of tests increases. There will be just more flaky tests, and more re-runs, more time needed to integrate any piece of code, and even more frustration. Both solutions lack a *mitigation* part.

Let's take a look at the quarantine example - as long as there's no additional mechanism for detecting the root cause of flakiness (let alone fixing it automatically), there needs to be a person dedicated for that job. Nevertheless, with large amount of tests, this seems to be the most sustainable solution. Re-running alone, without any mitigation step, is only a temporary treatment, and should be treated as the last resort. In both cases, without the root cause analysis for each and every failure, the value of the tests will be decreasing significantly with time.

## Many faces of flakiness
The authors from University of Illinois in [*An Empirical Analysis of Flaky Tests*](http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf) performed an experiment, in which they searched for the main root causes of intermittently failing tests. They analyzed a number of open source projects from the Apache Software Foundation, looking for commit messages that indicated a flaky test being fixed. They analyzed each occurrence and classified them into 10 different categories, depending on the root cause of flakiness. The findings were very interesting, but in case you're not convinced, here are my favorite parts:
1. Majority (77%) of all flaky tests fall into 3 categories:
  * asynchronous wait (45%) - test execution does not properly wait for asynchronously called result before using it,
  * concurrency (20%) - different threads interacting in a non-desirable manner,
  * test order dependency (12%) - test outcome depends on the order in which the tests are run.
2. There are clear patterns to most of the failures in the top 3 categories:
  * Many (54%) of tests with asynchronous wait issue are fixed using `waitFor` - a polling mechanism which allows to wait for desired event.
  * Almost all (97%) of tests failures with concurrency issue are due to concurrent accesses only on memory objects.
  * Majority (74%) of tests with order dependency issue are fixed by
cleaning the shared state between test runs.
3. Finally - some fixes to flaky tests (24%) modify the code under test (rather than the tests' code), and almost all of these cases (94%) **fix a bug in the code**.

The first two points make up for an interesting conclusion: **flaky tests can be very easily dealt with**. Most of failures show a clear pattern. What's more, these concepts are quite known in the general area of software development, and are not limited only to tests. That leads me to a provoking thought - does it mean that writing tests is held by lesser standards than writing software? Unfortunately, I can't answer that based on this research without having data about total number of bugs fixed in the code base and their categorization.

Finally, let's focus on the third point. I must say, it's quite a revelation - **more than 1 in every 5 flaky tests catches and exposes a bug in software**! These tests do their job, they do tell you about bug - but they need some special attention. Either because the nature of the bug is complex (maybe it occurs only under very special circumstances?), or maybe because the tests are. Whichever it is, by investigating it, you'll learn a whole lot about both your application and your tests. By disabling such test without root cause analysis you deliberately throw this information on the floor, depriving yourself of learning about the failure in your software.

## Maintenance is not easy-peasy
The biggest turning point in my approach to any kind of automated tests was when I understood that **the tests are just another product**. Like any piece of software, it requires maintenance, otherwise it's going to break. It's as fragile as the software under test, and as the environment that it runs on. Tests are subject to dynamics so strong that some kind of breakage is inevitable, especially during early stages of product development.

With this approach, I have room for failure, so I can learn about: code, test, test environment, infrastructure, some specific language quirks and more. It's sustainable as long as the cause of failures is diagnosed quickly. It doesn't mean that it has to be fixed immediately - it's enough to have a plan for fixing along with its priority. Regardless of the root cause, it's great to have a quarantine mechanism in place. An isolated environment where both flaky tests and those that discovered a known failure can keep giving you information, without decreasing the speed or reliability of your software delivery, definitely won't contribute to the frustration factor.

## TL;DR
It's OK to disable the failing test as long as its root cause is known, the fix is planned and prioritized. Failures in tests are awesome, because they have something to teach you. Flaky tests are even more awesome, because they teach black magic, and are more fun in general. Just start dealing with them. And fret not, everyone has them.

---

#### Appendix

Analyzing flaky tests can bring some interesting lessons. For example, I once learned the hard way the difference between `==` and `is` operators in Python, and how it caches small integer objects, when my tests (which used `is` for comparing two integers) were passing for small values and failing for bigger ones. See for yourself:
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
Of course, the value in my tests was calculated dynamically, depending on the state of the database - so the test was flaky!

 ---

#### Footnotes

[^randomly]: Sometimes these tests are also being referred to as *failing randomly*. I hate this wording, because it suggest there's a random factor to it. And unless the test or application explicitly relies on pseudo-random number generation, it's not really random. Instead, we can say that the test is failing intermittently, irregularly or discontinuously. Or we can stick to calling the test *flaky*.
