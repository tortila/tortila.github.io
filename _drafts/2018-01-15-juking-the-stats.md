---
layout: single
title:  "DRAFT: Juking the stats"
permalink: juking-the-stats
slug: juking-the-stats
comments: true
---
## Test failures treatment
There are couple of ways failed tests can be dealt with. In the perfect world failed test is treated as a highest priority. Developers (or person responsible) stops doing anything and jumps quickly to investigate failed test and fix its root cause. Unfortunately, for various reasons, this is not always the case. So what are common real-world solutions?
### The good
### The bad
### The ugly
Much like in *The Wire*, my beloved TV series, failed tests are quite often a subject of *juking the stats*. Why bother with test that is useless, when you can simply ignore it, skip it, disable it or remove it from the suite? The outcome is tempting - a green light from CI system, 100% of tests pass, a round of applause for a hero that made it happen.

{% include figure image_path="https://artist.api.lv3.cdn.hbo.com/images/GVU2-Eg7Vl47DwvwIAWY0/detail?size=640x360&format=jpeg&partner=hbocom" alt="source: HBO" caption="A promise of crime rate decrease can be a powerful advantage for candidate running for mayor's office" %}

When the crime rate stats were being manipulated in *The Wire*, a few things could have happened in Baltimore - one of them is, surprisingly, a real decrease in crime (at least in theory)[^thinkprogress]:
> Confidence matters a lot for a city. When people have the sense that conditions in a given city are good and improving, businesses will invest and that creates jobs. And people become more inclined to move in, raising property values. Higher property values mean more tax revenues for social services and for public safety. And more jobs, by all accounts, leads to less crime.

In my experience, cooking up the results of tests just in a pursuit of much desired green light leads to a completely opposite situation. The results are being manipulated in a very simple way - by disabling the failing tests. As a result the whole suite becomes a meaningless gate in software integration pipeline. The only reason for their existence is to pass. They no longer provide a valuable information about software under test.

---
[^thinkprogress]: Yglesias. *Juking the Stats*. [https://thinkprogress.org/juking-the-stats-e9ba45af18e0/](https://thinkprogress.org/juking-the-stats-e9ba45af18e0/)
