---
title: Remodeling an Ember App - Introduction
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Tue Jun 01 2021 20:31:47 GMT-0400 (Eastern Daylight Time)
tags:
  - new
---

In a series of blog posts, we'll go on a journey together to remodel an older Ember app, incrementally bringing it up to date with the
latest and best Ember and Ember Data patterns.

We will walk through updating a complex, real-world app,
[ember-api-docs](https://github.com/ember-learn/ember-api-docs),
which is the front end of [api.emberjs.com](https://api.emberjs.com). Along the way, you will learn how to approach this process
for your own apps - updating dependencies, debugging build errors, migrating to Octane syntax, writing better Ember Data serializers & adapters... there's a lot to do!

For this series, I'm pair programming with Chris Thoburn, aka [@runspired](https://github.com/runspired), who is known for
his work on Ember Data. He has over 500 commits and some great
debugging skills that you and I can learn from.

## Preparation

The focus of this first article will be about deciding where to begin,
what kind of mindset to have, and an overview of the debugging tools in our toolbox.
There are a lot of things we _could_ work on improving in this app, and there
are multiple paths we could take that lead to success.

### Areas of focus

Making a list of them and prioritizing is very important step for any app refactor or upgrade.
Some broad tasks to consider include:

- Updating to the latest package versions
- Resolving deprecations
- Running codemods
- Refactoring confusing stuff
- Documentation
- Testing and QA

### Making a detailed list

Thinking about these areas of focus helps me get started with making a detailed list.

Chris Thoburn and I did a brainstorm about `ember-api-docs` and came up with the following list:

- Upgrade Ember and Ember data versions
- Run some codemods for Octane
- rewrite computed properties to cached getters, simplifying where possible (reduce intermediary computed properties)
- Figure out how to use less of Ember Data
- Figure out what needs to be in API response meta
- audit models and remove async relationships that didn't need to be async and ensure that inverses were properly wired
- make sure the API returned the format the store expects, and remove the serializer entirely
- Remove the adapter and replace it with one that just does a simple fetch request
- drop both the adapter and the serializer package and the ember-data package, instead installing store and model and record-data directly

### Prioritizing

Next up is prioritizing. This can be hard to do when you don't know an app well, and that's ok. This list's ordering should shift over time as you learn new things! But you do need to decide what to try first.

For this app, the clear first step is to upgrade the Ember and Ember Data versions.

We start here so that we have all the latest and greatest features. This can also be a challenging step because shifting dependency versions sometimes reveal bugs, but finding them later in the middle of a refactor is no fun.

### Getting in the right mindset

Overhauling an app (or even a complex component) takes a different mindset than I use in my usual daily coding.
Most days, I have expectations about how the code works, because I wrote it or studied a section of it closely. But in an overhaul, big upgrade, or refactor, there are a lot of unknowns! If I intentionally adopt a different mindset at the beginning of a big uprade, I'm a happier developer. What's that mindset look like?

When I make a change that does not fix a problem, I try to be curious about why.

When I see a different error, I treat it as a step forward.

I keep a list of my successes as I go, including learning new things or getting one step deeper into the process.

When I get stuck, I conduct small experiments, and record the results.

I accept and expect that some of my experiments will be dead ends.

When I don't know where an error is coming from, I focus on trying to find the source before trying to fix the error.

I am skeptical of the things I changed, by default. This especially includes what's in the node_modules folder and shifting dependency versions.

### General debugging strategies

Over the course of this series, we will use lots of different debugging techniques.
You will see examples of all of these! But it's useful to look at them as a group, and think of this as tools in your toolbox.

- Selectively commenting things out from your app to hone in on where a problem is coming from. Chris calls this "bisecting," and I call it "Cat in the Hat" debugging. Identifying the areas that are _not_ a problem is progress!
- Cleaning out `node_modules` - deleting the app's `node_modules` folder and the lock file, either `package-lock.json` or `yarn.lock`, and reistalling dependencies
- Restarting the local server - important to do if you change dependency versions or build configuration
- Logging template values to the console. In your `.hbs` files, you can add `{{log this.myVariable}}` and see the output in the console
- If your app is using it, turning `fastboot` off while you debug. Add `?fastboot=false` to the end of your locally served URL, i.e. `http://localhost:4200/?fastboot=false`. 
- Using "Break on exceptions" or "Break on uncaught exceptions" in your browser's debugging tools, to help with finding out where errors are coming from, with some more informative context
- Using `debugger` in your `js` files to set breakpoints that you can see in your browser's debugging tools
- Using `yarn why package-name` or `npm ls package-name` to confirm which versions of dependencies
are actually in use, and which other dependencies use them.

## Up next

Next, in Part 2, we will start doing the app version upgrade!
