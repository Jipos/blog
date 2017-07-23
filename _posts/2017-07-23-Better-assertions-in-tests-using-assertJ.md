---
layout: post
title: Better assertions in tests using AssertJ
---

"AssertJ is a library providing easy to use rich typed assertions" is what the github page for the project says
> https://github.com/joel-costigliola/assertj-core

The Homepage for the project shows some examples of what the library can do:
> http://joel-costigliola.github.io/assertj/index.html

The reason I got interested in AssertJ is SoftAssertions.
> http://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html#soft-assertions

"Using soft assertions, AssertJ collects all assertion errors instead of stopping at the first one."

This way, when your test contains 4 assertions, and 3 fail, all 3 will be logged, instead of just the first one that failed.
