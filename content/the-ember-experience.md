---
title: The Ember Experience - 2019 Roadmap post
image: ./images/repair.jpg
imageMeta:
  attribution: Milan Degraeve
  attributionLink: https://unsplash.com/@milandegraeve?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
featured: true
authors: 
  - jen-weber
date: Sat Jun 01 2019 21:38:15 GMT-0400 (EDT)
tags:
  - new
---

If I imagine life after [Octane](https://emberjs.com/editions/octane) lands, I can see a few areas that need work: the learning story for new Ember devs and adopting accessibility as a framework feature.

This post is in response to Ember's [Call for Roadmap  Blog posts](https://blog.emberjs.com/2019/05/20/ember-2019-roadmap-call-for-posts.html), and I invite you to write one of your own! What do you think Ember should focus on over the next year? I wrote [one of these last year too](https://medium.com/r/?url=https%3A%2F%2Fgist.github.com%2Fjenweber%2Fa9fbea98478fc3841fb8b24f7dc961c8), and you can read [here](../revisiting-my-2018-ember.js-roadmap-blog-post) about how Ember hit nearly all the goals I was hoping for!

## What is the Ember Experience?

The ideal Ember Experience is that things work as they should out-of-the-box, following modern web development patterns. It's easy to get ramped up using real-world code examples that teach the recommended practices used by experienced Ember developers, from day one.

What do we need to fix or build so we can get there?

## Let's teach people to use the things we're proud of

Take a look at [Vue Vixen's library](https://vuevixens.github.io/docs/workshop/) of open-source workshops. It's inspiring to see what it looks like when framework learning and teaching is done exceptionally well! While we may not have the same resources as other frameworks, whatever we have set as priorities, we have consistently achieved. I've heard other people say that the [Ember Guides](https://guides.emberjs.com) are great and some of the best docs out there, but we need to reach higher.

Why should teaching and documentation be a priority of the entire Ember organization? As many as 50% of visitors to the Ember Guides and Tutorials are visiting for the first time. (Either that, or a ton of you are very good about using Incognito mode on your browsers, or other tools that block Google Analytics.)

### A problem to fix

There's a problem though. The majority of Ember's Core Team members have never done any writing for the Guides.

Thankfully, there are a zillion community members who help keep the guides up to date and improve them over time, at a pace of [30–40 PRs per month](https://github.com/ember-learn/guides-source/pulse). Although it takes up most of my open source time, whatever time I spend on reviewing PRs is always time well spent, and I am so grateful for the participation.

**However, the voice of Ember's leadership is largely missing from the materials that are most often used.**
 
Everyone has their area of specialization, and many core team members do their own kind of docs writing for the APIs, deprecations, etc, but I think we need to do better. Even inside the learning core team, we're not focused on long-lived, most-used content. The group focus is on website infra and the [Times newsletter](https://the-emberjs-times.ongoodbits.com/). Only a couple people do most of the Guides work. We're off-balance, and we need more core team members to be engaged.

### How to move the needle

These are the main areas where we could make some changes:

1. We could make documentation and learning be a focus for 2019–2020, meaning that the core teams would commit to doing more work to help teach the tools that we are all guiding and building. 
2. We add more writing-focused people to the core teams (all four teams)
3. We support the creation of "less official" Ember resources, which could serve as a training ground for new writers. It's hard writing within the constraints of the Guides, so cultivating other resources is good practice and enriches the knowledge base. The Guides are the "happy path" but we need other places that document the weird stuff.
4. We do a better job of collaborating with community members who could help with big changes. There's too much risk currently for most non-core people to do something like devote a full week to helping revise docs- would their work ever get merged? Or will it get lost in back & forth about the right way to teach Ember? I'm willing to be a bit reckless with the Guides merge button, but the rest of the teams need to be on board.
5. We set a specific, time-based goal of writing another Ember Tutorial to go with Super Rentals and the Quickstart. If there are no community volunteers, we do it ourselves.
6. We consider granting PR review & merge abilities to non-core team members, for the Guides only. This distributes review responsibility so that seasoned writers/reviewers can spend more time planning the next big steps, getting new writers involved, and writing itself.
7. We get some kind of code block tests in place. We burn a lot of time fixing formatting and mistakes that could be caught by a JavaScript parser. It's more "infra" work, but in direct service of learning quality, and recaptures some time burned by the current writers & maintainers.

There are some downsides to these points. If core team members were to help more with the guides, it would mean they were doing less of something else. What can we cut? What can we do more efficiently? Which things can become the responsibility of someone else, through mentorship and leadership as opposed to individual contributions? Who is bored of what they are doing, or has been uninvolved lately, and wants to try something new? This will take discussion.

## Embrace the land of copy-paste

Another achievable, realistic way to level up the guides is to give better code examples. Copying and pasting code is an important part of how people learn and work, so our resources should make that easy and safe to do.

Today, if you copy and paste from Ember's official Guides, there are problems. For example, the Guides show input text boxes over and over, but only a small percentage of those samples have a `<label>`. Sure, the article focus is on teaching how to use the component, not how to write HTML. However, if we assume that most people will copy and paste, and we want Ember users to follow best practices for the web, we should step up our game. There are a ton of little mistakes like that which impact accessibility, for example, plus browser compatibility.

### More real-world examples
To that end, we should create more real-world examples for Ember and make them available to our users. For example, we teach how to use form inputs, but not how to make a fully-functional, accessible form. A long time ago, there was a section of the Guides called the Cookbook which tried to do this, but it was unmaintained and eventually removed.
The only way we can deliver is if we have more people engaged in writing and maintaining Ember's documentation and tutorials, at a deep level.

## Accessibility as a framework feature
Accessibility is a big theme in Ember lately, and I'm a huge fan of any progress in that area. Since Ember covers so much of the concerns of front end development, it's not a giant leap to roll accessibility into Ember's out-of-the-box experience.

A focus on accessibility gives Ember a competitive edge that can attract companies who must follow accessibility standards, and developers who care about making their work usable by everyone. Our outside image is important, and how people feel while using Ember apps is important.

As a long-lived library and community, we get the chance to solve the really hard problems, the kinds of things that can change the web. We should make that happen.

To give some concrete examples, I think the [accessibility linting rules](https://github.com/ember-template-lint/ember-template-lint/blob/master/docs/rule/no-invalid-interactive.md) were a huge success, and we should make the next moves to incorporate ember-a11y tools as the standard. We could become the leader among SPA frameworks for solving things like changing routes in a way that screen readers understand. 

For more on this, read Y[ehuda Katz' 2019 Roadmap Blog Post](https://yehudakatz.com/2019/05/20/ember-2019/) as well as [Melanie Sumner's post](http://www.melsumner.com/blog/ember/wishing-on-a-star/) from last year.

## Business use case

These aspects of Ember have a real impact on my work at [Cardstack](https://cardstack.com). Ember.js is a key dependency of the Card SDK. Any improvements we make to the Ember Guides will help me teach Cardstack to new devs. Any improvements we make to accessibility help our products gain traction with enterprise customers.

Other companies benefit in the same way on the accessibility front. When it comes to learning materials, there's a real cost to _not_ investing in this area, since most companies using Ember train at least some of their developers in-house.
It's totally reasonable for a dev with experience in React, Vue, or Angular to learn Ember quickly.
We have an opportunity to make onboarding easier for these companies, and if we do it well, new teams who want to adopt Ember can confidently do so.

## How to participate in the Roadmap

Please write your own blog post for the [Ember 2019 Roadmap](https://blog.emberjs.com/2019/05/20/ember-2019-roadmap-call-for-posts.html)! Even if it's just to say you agree/disagree with someone else, we want to hear your voice. It has a bearing on the outcome. Thanks for reading!
