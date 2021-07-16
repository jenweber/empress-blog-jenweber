---
title: Remodeling an Ember App - Codemods and jQuery
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Tue Jul 13 2021 16:23:53 GMT-0400 (Eastern Daylight Time)
tags:
  - new
---

When you need to upgrade an Ember app, codemods can help you update your app's syntax faster than you could make changes by hand. Today, we'll talk about codemods and cover what to do about jQuery usage in your apps.

This is Part 4 of a series of blog posts. We're on a journey together to remodel an older Ember app, [ember-api-docs](https://github.com/ember-learn/ember-api-docs), incrementally bringing it up to date with the latest and best Ember and Ember Data patterns.

For this series, I'm pair programming with Chris Thoburn, aka [@runspired](https://github.com/runspired), who is known for
his work on Ember Data. He has over 500 commits and some great
debugging skills that you and I can learn from.

## What you will learn in this segment

- How to find codemods
- How to run the codemods
- How to decide what to change, and what to leave alone
- Where jQuery fits into the story

## The road so far

So far, we have upgraded our dependencies and tests are all passing. This sets a strong foundation for running codemods and then making sure that tests keep passing.

## Octanify

Ember Octane is Ember's first "edition." You can think of an edition as a collection of features and syntax that together form a cohesive mental model. There are multiple ways to accomplish a feature in Ember, and Octaneified apps use the latest styles.

There is a CLI command that configures an app to use Octane:

```
npx @ember/octanify from   which set some dependencies and flipped flags in optional-features.json
```

You can find more instructions about this command in the [Ember upgrade guide](https://guides.emberjs.com/release/upgrading/current-edition/). In short, it sets some dependencies in `package.json` and feature flags in `optional-features.json`. Optional features are one way that new features are rolled out in Ember apps without making breaking changes. When you are ready, you can opt into them, but if you are not ready, you aren't blocked from upgrading. Regular upgrading is critical for companies and teams that value getting security, bugfixes, and feature updates over long periods of time.

Octanify sets the following feature flags in `optional-features.json`:

```
{
  "default-async-observers": true,
  "jquery-integration": false,
  "template-only-glimmer-components": true,
  "application-template-wrapper": false,
}
```

You can learn all about these individual features in the [Optional Features Guide](https://guides.emberjs.com/release/configuring-ember/optional-features/).

## Handling jQuery

For our app, all the optional features set by Octanify were fine. Setting `jquery-integration: false` meant that instead of using `this.$()` in our apps, we had to install `@ember/jquery` and import jQuery individually - no problem. However, the `jquery-integration` flag made us wonder, could we remove `jQuery` from our app altogether? We were only using it in one place in our app, and doing so would cut some kb from our app.

However, we didn't just need to check our app. We also had to check if any addons used jQuery. I used to search my `node_modules` for usages, but that would return a lot of false positives compared to a strategy that Chris Thoburn showed me.

Chris first ran the build for the app:

```sh
npx ember build
```

By default, the command puts built files in the `dist/` directory. Then, Chris was able to search for `this.$` and `Ember.$` and confirm that our app's addons did not need jQuery. This is better than searching `node_modules`, since we are only checking for the use of jQuery in addon features we are _actually using_.

Another common use of jQuery in apps is via Ember Data. By default, Ember Data uses jQuery for its HTTP requests. However, if an app has `ember-fetch` installed, it will use `fetch` instead. It's important to install `ember-fetch` in order to provide broad browser support for your fetch requests.

Finally, we took another look at our direct use of jQuery in this app, in our Table of Contents component:

```js
import { action } from '@ember/object';
import Component from '@ember/component';
import jQuery from 'jquery';

export default class TableOfContents extends Component {
  @action
  toggle(type) {
    jQuery(this.element)
      .find('ol.toc-level-1.' + type)
      .slideToggle(200);
  }
}
```

jQuery is providing a nice open/close animation for one of our menus. Whenever I see jQuery in use, I always check [youmightnotneedjquery](http://youmightnotneedjquery.com/) to see if there's an easy alternative. In this case, there was not, so in the interest of moving forward, we leave jQuery in our app for now, and open an issue asking for help making a new CSS/plain JavaScript animation.

It's not a huge deal to leave jQuery in your app unless you have a strategic, benchmarked focus on app performance. jQuery is an incredibly successful project - so successful that its best features are now provided by native browser JavaScript. Ember's API documentation app is used by developers around the world with varying internet quality levels, and so we do need to save some kb where we can, however this one task should not block our progress towards improving other areas of the app.

## Ember CLI Update Codemods

Now on to some more codemods. [Ember CLI Update](https://github.com/ember-cli/ember-cli-update) can update your dependencies, and also provides a subset of the most common codemods that make deeper changes.

After running dependency updates, you can run the codemods like this:

```sh
# Start your app
npx ember serve
# Run the codemods
npx ember-cli-update --run-codemods
```

Some codemods start with the letters `fpe`. This stands for "function prototype extension."

## Additional codemods

There are many more codemods than those provided by Ember CLI update. One good place to look for them is [ember-codemods](https://github.com/ember-codemods) on GitHub.

Some codemods require that you have your app running. That's why we start the app with `ember serve`. Such codemods look at the built files in order to infer the correct changes to make. This is possible through a strategy called telemetry, via [`ember-codemods-telemetry-helpers`](https://github.com/ember-codemods/ember-codemods-telemetry-helpers).

### A problem with one codemod

When we ran `ember-modules-codemod`, we encountered an error. The codemod helpfully told us which line it failed on:

```js
titleToken(model) {
  return model?.fn?.name;
}
```

This line of code uses optional chaining, with `?.` syntax. Optional chaining is a feature of JavaScript that was added _after_ these codemods were initially written.

We can see for ourselves one of the challenges of codemods - when they are written, they are a snapshot in time. It takes work to keep them functioning as JavaScript and Ember apps change. If you are working on a large app, it may be less work to fix a codemod bug than to make all the changes by hand. Additionally, codemods are written to work for individuals' apps, and so it takes community effort for codemods to work across many edge cases that authors could not forsee.

Another example of maintainability issues with codemods is the ES5 getter codemod - you have to run it before you run native classes codemods, because it doesn't know how to parse native classes. They didn't exist when the getter codemod was written. These aren't unsolvable problems, but they mean that developers whose apps are super out of date may have a harder time upgrading. There are many benefits to having a regular ugrade and maintenance schedule for your work, and this is one example.

## Configuring VSCode to play nice with decorators

Some of the codemods we ran introduced decorators. VSCode was our code editor of choice, and its default linters didn't like the use of decorators.

We removed `jsconfig.json` from our app's `.gitignore`, and then configured VSCode to stop yelling about decorators in `jsconfig.json`:

```
{
    "compilerOptions": {
      "experimentalDecorators": true
    },
}
```

## Dealing with codemod mistakes

Not all codemods are flawless. There's a lot of variation in an app! You should think of them has helpful suggestions, rather than a complete solution to your upgrade process. The best way to catch codemod mistakes is to carefully review the diff, run your tests, and make a commit after each codemod. Here's an example issue we had with a codemod that rewrote `link-to`:

```handlebars
// before codemod

{{#link-to
  data-test-uses-link
  (concat parentName section.routeSuffix)
    model.project.id
    model.projectVersion.compactVersion
    model.name
    item.name
    (query-params anchor=item.name)}}
  {{item.name}}
{{/link-to}}

// after codemod
<LinkTo @route={{concat this.parentName section.routeSuffix }} @models={{array this.model.project.id this.model.projectVersion.compactVersion}} @query={{hash anchor=item.name}}>
```

The codemod cut out some of our route segments, leading to incorrect links in one part of the app:

```text
// correct
/ember-data/3.26/classes/Ember.Inflector/methods/singular?anchor=singular

// incorrect
/ember-data/3.26/classes/ember-data/methods/3.26?anchor=singular
```

How did this happen? We took a look at the source code for the codemod, and saw that it has special handling for dynamic routes, query params, and data-test. When we used all three of these things together in one link-to, it was the perfect storm.

## Codemod strategies and takeaways

Our upgrade would have been easier if we ran a single codemod at a time, and make it its own commit, and ran it in CI, and shipped it. If you have the ability to roll your app back quickly in production, you don't need to work as carefully.

That said, you really need to go through the diff if you try to do a big bang upgrade.

If your test suite coverage is bad, and you can't improve it, you can run codemods on specific chunks of code and QA them - do one set of related components at a time, then look at the diff and click test. Many (or maybe all) codemods accept file paths in the CLI commands.

## Conclusion

That's it for codemods! Next up, we will take a look at the code that couldn't be modded, and do some refactors to simplify the app now that we have Octane's awesome features available, such as `tracked`. Thanks for reading!
