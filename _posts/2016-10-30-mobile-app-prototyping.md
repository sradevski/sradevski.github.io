---
title: "Designing and Prototyping a Mobile Application"
layout: post
date: 2016-10-30 10:25
tag:
- software engineering
- personal experience
- startup
- mobile applications
- prototyping
- design
blog: true
star: false
author: stevcheradevski
description: Discussion on tools for designing and prototyping mobile apps
---

## Summary:

A short discussion on what I found to be a fast way to design and prototype mobile applications, and the steps that can follow it.


## Why Designing and Prototyping?

Starting a new project is always an exciting thing. As in my case, you might be thinking of starting a business with a friend, a mobile app. You have thought about what to do, what functionalities you want to support, your business' target group, and so on. In order to start developing the app there are many design decisions that need to be made, and this is where designing and prototyping comes into play.

Designing and prototyping is a great way to test design concepts about your app. You can visualize your ideas better, see if there are gaps that you didn't consider, and fail fast. It is also difficult to start developing an app without having an idea of who it would look visually, so it is also a good guideline while developing the application. I would like to share my insights and how I approached designing and prototyping.

## Taking a Step Back

Before starting to prototype, I spent a considerable amount of time looking for a good tool, with no conclusion. I must say, there are more tools out there than are really necessary. I started comparing, checking for pricing, trying demos... and I got overwhelmed and tired. Many of them are great, no doubt about it, but I felt like that much effort is too much for a concept it will surely change all the time. If you are a multi-million company with many stakeholders and entities into play, then using some of those may be important, but for a startup in its baby phase with almost no resources, it was an overkill.

Then I decided to take a step back, and take a pencil in my hand. I found some [awesome templates](https://www.interfacesketch.com/) of mobile screens, printed them out, and started drawing. No need to install, license, drag-and-drop, just some simple sketching. With good UX in mind, I tried to keep away from any design rules and leave some space for creativity. I must say, it felt great to do some sketching by hand, and while doing so I got tons of new ideas about improving the interface. This of course doesn't mean that you won't get new ideas when designing on the computer, but doing it by hand definitely offers more flexibility. Even though things will change a lot from now on, I think this is definitely a good first step towards making a well-designed mobile app.

After making some initial sketches, we talked with my partner about what is good, what is bad, what can be improved, and did some modifications. From the finished design, I extracted all logical components that can be reused, and currently I am starting to implement a prototype in React Native. Once the prototype is done, we will test it out on several users and see how they interact with the application. You need only 5-6 people to find most of the UX problems in the app (there was a research paper on it that I can't find, so just take my word for it). Then we reiterate and reiterate, until we get to a product that is good enough to fully implement.


## Conclusion

I think limiting the number of tools you use will reduce the complexity of your development process, which is something very important in the beginning stages of a project. Doing a design by hand might not be the most sophisticated approach, but it definitely has a lot of benefits. Also, making a prototype the same way as you are implementing the future app can save you some time by reusing the components, if done correctly.
