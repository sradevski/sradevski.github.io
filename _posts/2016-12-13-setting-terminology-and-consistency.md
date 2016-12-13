---
title: "Consistency and Clear Terminology in Software Development"
layout: post
date: 2016-12-13 16:33
tag:
- software engineering
- personal experience
- startup
- mobile applications
- terminology
- naming consistency
- variable naming
blog: true
star: false
author: stevcheradevski
description: Discussing how setting up terminology early in a project can improve consistency and decrease cognitive load.
---

## Summary:

Problems I faced when starting a new project, and some easy ways I could have overcame them (but I didn't) by setting up terminology and being consistent with its usage.

## The Wall I Hit

I have been working on this React Native project for several months now, and it has been going quite well. I set up the architecture, testing environment, app design, and the goals of my project. Everything has been nice, except for one thing: **consistency**. I did so many refactoring because of poorly chosen names. I introduced bugs. I wrote confusing code. All thanks to not being consistent in my coding style. Of course, I use ESLint to resolve some of these problems, but it doesn't catch everything. And the main thing that is not caught by ESLint is **terminology**.

Depending on the industry your project belongs to, there will be industry-specific terminology that will be used throughout the application. Some of the terms may be confusing, some might have several meanings, or there might be several words describing the same thing. Not being clear what to use when and how will increase the complexity of the code and drive developers mad. Not having a well-established terminology means lack of consistency, which in turn is what makes code confusing and more difficult to reason about on a project-level.

## The Simple Solution For It

There is one word that will mitigate most of the problems: **cheatsheets**. Although the project requirements will probably change quite a lot, terminology wont't (in principle). Get few people who are the most knowledgeable in the industry you are working in, and try to define all the industry-specific terms you can think of. Afterwards, write a very short (5-6 words) description of the term, along with some synonyms. I guess it will look a lot like a thesaurus. Have the cheatsheet easily and quickly accessible to all developers, and make sure you can easily extend it (digital). You can either deliver it as an online document, or have a big screen in the middle of the room showing cheatsheets (if you can afford it), as it will not clutter the desktop of developers. Whenever some confusion appears, extend the list. You should do this even if you work alone on a project.

Once everyone is on the same page regarding terminology, talking about things becomes simpler. Renaming/refactoring code becomes simpler. It is easier to reason about the code and to get the "big" picture of the project. It is definitely a worthwhile effort, yet it is just one aspect of consistency.  

There are several more aspects of consistency that tools like ESLint can't entirely enforce, and that is proper, unambiguous naming. As naming has been really well covered in books such as [The Pragmatic Programmer](https://en.wikipedia.org/wiki/The_Pragmatic_Programmer) and [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), there is no need to repeat things. All I have to say is, have someone establish naming standards, create a cheatsheet, and have it available the same way as the terminology cheatsheet. It is not as important to do exactly the same thing as everyone else does it (although it can help), but to be consistent, no matter what kind of naming style you use.

You can also take it a bit further. Companies like Spotify have done exactly that, using the term tribe for a team, tribe leads for project managers, and so on. Not necessarily better, but interesting, and sometimes that also counts. Nevertheless, I am sure everyone in the company knows exactly what exactly a tribe is, and this is what is important.

## Conclusion

The impact of consistency in code can be more than what many would think, especially as the size of a project grows. I have talked about the problems I faced as a consequence of not having properly defined terminology, but it is not limited to it. Consistency can be affected by naming, coding style, even tools used in a project. ESLint or any other linting tool is a definite must, and for everything else not currently supported by a tool create cheatsheets. Having everyone on the same page will result in cleaner code and less cognitive load.
