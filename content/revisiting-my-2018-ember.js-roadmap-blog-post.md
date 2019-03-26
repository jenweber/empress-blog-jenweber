---
title: Revisiting My 2018 Ember.Js Roadmap Blog Post
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Tue Mar 26 2019 17:07:29 GMT-0400 (EDT)
tags:
  - new
---

What has changed since last year? In May of 2018, I wrote a blog post for the [Ember.js Roadmap process](https://blog.emberjs.com/2018/05/02/ember-2018-roadmap-call-for-posts.html).
It was called [Be loud and be ready - my hopes for Ember.js in 2018](https://gist.github.com/jenweber/a9fbea98478fc3841fb8b24f7dc961c8).

I tried to imagine a future where Ember was wildly successful, not just surviving but _growing_, then work backwards to think through what those imaginary, wildly successful people must have done.

So, how did we do? Are we on-track to being the wildly successful people?

## Being loud

Here's what I said a year ago about "Being Loud":

> The people of Future-Ember worked to increase public awareness so that more developers knew about it and considered it for their projects. The Core Team led by example, writing and speaking, and the rest of the community was empowered to do the same.

I would wager that it's been a record year for this. 

I know that a lot of core team members and cornerstone community members have given talks at non-Ember events and have done a _ton_ of writing. 7/10 of my talks were at general venues like BostonJS and Boston Code Camp. 
Community members have also revived the [emberjs subreddit](https://www.reddit.com/r/emberjs/). The [official blog](https://blog.emberjs.com/), once kind of a dead zone except for version releases, is now packed with evidence that Ember is on the move.
The [Ember Times](https://the-emberjs-times.ongoodbits.com/) team has their writing down to a science.
The newest Ember.js Core Team member, Chris Garrett aka [pzuraq](https://github.com/pzuraq) is both an accomplished writer _and_ someone who has helped ship a whole bunch of Ember's latest & greatest features. See posts [here](https://medium.com/@pzuraq) and [here](https://www.pzuraq.com/).
In another example, core team member [Melanie Sumner](https://github.com/MelSumner) wrote an [Accessibility Guide](https://guides.emberjs.com/release/reference/accessibility-guide/) for the official Ember Guides, gave talks at accessibility-focused events throughout the year, and is working on framework code to make Ember better out-of-the-box for people who use assistive technology like screen readers.
[Jessica Jordan](https://github.com/jessica-jordan) is speaking at this year's [JSConf EU](https://2019.jsconf.eu/) - a huge achievement - on web comics and animation.
The list goes on and on.

> Future-Ember provided approachable, current, convincing materials for new visitors.

A lot of this work has been completed, and the next 3 months will be the true testing grounds for its effectiveness.

Most notably, the outcome of the 2018 Roadmap process was [this RFC](https://github.com/emberjs/rfcs/pull/364), which introduced the first-ever ["edition"](https://emberjs.com/editions/) of Ember, called Octane. Octane is currently in a [preview period](https://emberjs.com/editions/octane) before its official release.
The purpose of Octane is to pull together all the latest features of Ember into one cohesive (non-breaking) experience.
It looks a whole lot more like regular JavaScript, with more explicit signals about which parts of an app are "special Ember things."
Hopefully, as a result, it will be easier for new devs to pick up.

## Being ready

Here's what I said was needed last year. Team... we did all of these. If you helped out with them, thank you so much.

> A refactor of (at least) the first page of every topic within the Guides, to make them friendlier and more readable... Ember has a large group of community members who I believe are ready and willing to help out.

> "Breaking" some of the Guides urls that have been held as canon.

While some things still need work, there were close about 400 Guides PRs over the past year. Four. Hundred.
Along the way, I ended up [writing an RFC](https://github.com/emberjs/rfcs/pull/431) to help jumpstart an even deeper set of revisions, and it was ultimately accomplished by over 40 contributors as part of work to prepare for the Octane preview.

> Current addon authors could help lead a refactor of the CLI docs. Newer people can help with writing and making sure that the results are as helpful as possible.

This also had an [RFC](https://github.com/ember-cli/rfcs/pull/120) that described the plan. Again, over 40 people helped to migrate content from the old site, remove outdated/incorrect info, and write some new things.
For example, we now have an official tutorial for anyone who wants to write an addon.
It's all live at [cli.emberjs.com](https://cli.emberjs.com).
There's still work to do to clean up, like adding redirects from the old site, but we can do it!

> Freshen up the look and feel of our main "marketing" resources like the home page.

The [proposed website redesign](https://github.com/emberjs/rfcs/pull/425) looks phenomenal.
It's inviting, modern, and really signals that Ember is something worth paying attention to.
[Sam Selikoff](https://github.com/samselikoff), [Leah Silber](https://github.com/wifelette), and others were instrumental to bringing this design to the forefront. People like [Melanie Sumner](https://github.com/MelSumner), [Chris Manson](https://github.com/mansona), and the Ember Learning Team laid the infra groundwork that makes it possible to do a full rebranding, thanks to some public website overhauls and the [ember-styleguide](https://github.com/ember-learn/ember-styleguide).

## Conclusion

I wouldn't have said this in previous years, but based on everything that's happened latey, I think we are actually on track to grow Ember's popularity and use.
I'm constantly in awe of the trailblazing going on in this community, in terms of diversity, inclusion, accessibility, removing barriers to entry, maintainability, cooperation across national borders, and more.
If people take notice of our tech, maybe they will take notice of these other things too, and spread them throughout the industry.
Even if people don't use our code, maybe our work will still make things just a little bit better for everytone.

It's easier than ever to be optimistic.
