---
title: "Static Typing in JavaScript Reduces Bugs"
layout: post
date: 2017-07-13 21:09
tag:
- software engineering
- research summary
- bugs
- types in javascript
- typescript
- flow
blog: true
star: false
author: stevcheradevski
description: Using static types in JavaScript, either by using TypeScript or Flow, reduces bugs and eliminates unnecessary type checks in the code.
---

## Summary:

This article does a very simple comparison of the benefits and drawbacks of using static types in JavaScript. Aside from the more intuitive reasons why you might benefit from types, now there is also scientific evidence that static types do in fact reduce bug count, which is summarized as well. At the pace the JS community is moving, building large-scale systems in JavaScript will become commonplace, and static types are one tool in our arsenal that can increase robustness and quality.

## The Long-lasting Dilemma

Numerous developers have been faced with the same dilemma - shall we use a static typing system for JavaScript, in other words, is it worth it? I have read and heard different opinions by different people, some saying [you don't need static types](https://medium.com/javascript-scene/you-might-not-need-typescript-or-static-types-aa7cb670a77b) as they don't really reduce bugs, and it was difficult to decide if static types are the way to go. Not having to write types statically is very convenient at times, but what is the tradeoff for this flexibility?

Roughly speaking, there are three main drawbacks:
1. Increased verbosity
2. Reduced flexibility, eliminating some of the benefits of dynamically-typed languages.
3. Added complexity in tooling and increased cost for training/learning.

Equally, there are three main benefits of static types:
1. Better tooling support (better autocomplete, visible types)
2. Reducing the number of redundant tests checking for type errors.
3. Reducing the number of bugs in the codebase.

To be honest, the first benefit is not very convincing, as all decent editors offer decent autocomplete. What attracted me more are the other two potential benefits.

I think to a lot of people who have written a bit of production JavaScript have faced themselves testing a function merely for invalid types passed to the function. This can be mitigated by using static types, thus reducing the amount of code (and no code is always good code). The third benefit, however, has had some controversy surrounding it, so I dug a bit deeper to find any evidence on whether static types increase or decrease bugs.

## Does Static Typing Reduce Bugs

Proving or disproving the impact of static types on JavaScript code robustness can be quite a challenging undertaking. Fortunately, [a study has been published earlier this year](http://dl.acm.org/citation.cfm?id=3097459) that tries to give a quantitative measure on how types affect quality and bugs in JavaScript. The question they try to answer is:

> "How many public bugs could Flow and TypeScript have prevented if they had been in use when the bug committed?".

Followed is a very short summary of how the study was conducted. The researchers took the code from a prior commit of commits that represent a bug fix from public repositories. They used static type annotations (with both Flux and TypeScript), and checked whether that would have fixed the bug fixed in the commit to follow (thus the bug not ending up in the repo in the first place). Taking a representative sample of 400 bugs from a number of different projects of varying sizes, the above-mentioned procedure was repeated with both "type checkers", and the results summarized.

**On the 400 bugs, both Flow TypeScript detected 60 bugs.** This represents 15% of the bugs investigated, which is quite a significant number in my opinion. We can also see that both Flow and TypeScript perform more or less the same (there were 3 bugs that only Flow detected, and 3 other bugs that only TypeScript detected). This means that **no matter the annotation tool, having static types in JavaScript reduces the bug count.**

As this is a very short summary of the paper and presents the main findings, so many details were omitted. As with any research paper, there are certain limitations and threats to validity, so I urge you to take the results with a grain of salt. Nevertheless, it is a good pointer towards the benefits of typing. I urge you to read the paper if you want to know the details on how the study was conducted.


## Conclusion

If you still ask yourself whether you should be using static types or not, my answer is, as in almost all cases, it depends. In my opinion, anything bigger than a hobby project will benefit from types. I am not saying that it is impossible to build something of high quality without them, as there are quite a lot of projects suggesting the contrary, but I do think in the long term types will increase the robustness and confidence you have in your code.

A number of developers might state all sorts of arguments disagreeing with the of the study, like how the people who introduced the bugs might have been inexperienced, the projects were not as widely used, and so on. The fact is, this is the reality of programming. Not everyone is closely familiar with a language, and not everyone has decades of experience. Finding ways to mitigate such risks is one part of building robust software.
