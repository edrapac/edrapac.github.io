---
title: "Tools"
permalink: /tools/
excerpt: "Small self-contained things I've built."
layout: single
author_profile: true
---

Small self-contained things I've built. Each one opens as its own full-width
page — they need more horizontal room than this column gives.

## [Hashcat Rules Reference](/tools/hashcat-rules/)

A browsable index of 294 hashcat rulesets — standalone files plus collapsed
near-duplicate families. Each entry describes what the ruleset actually does,
derived by parsing its hashcat operators and applying them to a sample word,
alongside rule counts and rough runtime cost.

Search by name, behavior, or use case; filter by speed band; sort by rule
count. Source data lives in
[edrapac/Hashcracking](https://github.com/edrapac/Hashcracking).

<p><a href="/tools/hashcat-rules/" class="btn btn--primary">Open the reference &rarr;</a></p>

<p style="font-size:.85em;opacity:.7">
Runtime figures assume a ~55M-word list against a fast unsalted hash at
~10&nbsp;GH/s. Slow hashes (bcrypt, WPA) scale to days or years — divide your
real hashrate into the candidate count.
</p>