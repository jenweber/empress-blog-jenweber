---
title: Remodeling an Ember App - Testing
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Wed Jun 16 2021 17:45:45 GMT-0400 (Eastern Daylight Time)
tags:
  - new
---

Today's topic is debugging an Ember app's test suite after upgrading some dependencies. This is a real-world app and the issues we face will be different from your apps, but you can learn the overall strategy and debugging approaches,

This is Part 3 of a series of blog posts. We're on a journey together to remodel an older Ember app, [ember-api-docs](https://github.com/ember-learn/ember-api-docs), incrementally bringing it up to date with the latest and best Ember and Ember Data patterns.

For this series, I'm pair programming with Chris Thoburn, aka [@runspired](https://github.com/runspired), who is known for
his work on Ember Data. He has over 500 commits and some great
debugging skills that you and I can learn from.

## What you will learn in this segment

- How to run tests on-demand instead of on every change
- How to run the tests for different git branches, side by side
- What to expect in your test suite after upgrading the linters
- How to deal strategically and efficiently with hundreds of linting errors

## Our progress so far

In the previous article, we had updated a bunch of dependencies and solved issues that prevented the app from building (build time errors) or prevented it from working correctly in the browser (runtime errors). We got to the end successfully, but there were some test failures.

Why were there failures? Well, we made a _lot_ of changes! No matter what framework I'm using, any time I make a bunch of changes before running my test suite, I expect some tests to fail. Another path we could have chosen was to make changes one at a time, while running the test suite at every step. However, sometimes, seasoned Ember.js developers can take a big leap forward and then reconcile the errors. Sometimes, this latter approach saves you time, but if you get stuck, it's a good idea to take a step back and do one change at a time.

## New tools for your toolkit - test running modes

Sometimes, the default behavior of `ember test --server` gets in the way. By default, the app rebuilds and test run every time you make a change. That wipes out the test failure information that you may need to reference while you make changes in multiple files.

Chris Thoburn showed me one way to improve on this experience and fix my test failures more quickly! He often runs a build continuously, and then in a separate process, run the tests. Although the app rebuilds every time you make a change, the test only re-run when you tell them to. So, the test failure output is always there when we need it.

```sh
# In one tab of your terminal, start the build and watch for changes
ember build --watch --output-path="./dist"

# In a new tab or window of your terminal, run the tests
ember test --serve --path="./dist" --no-launch
```

In these examples, we pointed the tests to the `dist` folder for the built app. We can apply this same strategy in order to run tests from different git branches side by side. For example, you could check out one branch of your app, run a single build, check out another branch, run a build, and then run the tests for both. Then you can inspect the results and compare them! You can even add debuggers to both branches and stop the test suites in the same place to inspect the state. Here's how!

```sh
git checkout main
yarn install
ember build --output-path="./dist-main"
git checkout upgrade-branch
ember build --output-path="./dist-my-upgrade-branch"
# Run the tests for main
ember test --serve --path="./dist-main" --no-launch
# Run the tests for the upgrade branch
ember test --serve --path="./dist-my-upgrade-branch" --no-launch
```

## Resolving linting errors

One change we made was that we increased the version number of `eslint-plugin-ember` and `ember-template-lint`. When we did that, we brought in new linting rules for our `.hbs` templates and JavaScript files. We got automatic guidance on how we could improve our app's syntax, coding style, and most importantly, its accessibility. However, it means our linting tests were all failing with over 100 errors! How do we deal with that?

For every linting error or warning, we must choose between the following options:

- Update the codebase itself to resolve the warning
- Turn the rule off in `.eslintric` or `.template-lintrc.js`
- Add a code comment that tells the linter to ignore that file or line of code

Most of the time, when you are doing a dependencies upgrade, you should only fix very very tiny things and for everything else, ignore the rule and do the fixes themselves in later PRs. Why? We don't want to inadverdantly change the app's behavior! When upgrading dependencies, ideally the app looks the same when we're done.

### Tools to help you deal with linting errors

There are some helpful tools in the Ember ecosystem for tracking and resolving linting issues over time, and they are especially important for large apps or teams.

First, `ember-template-lint` helps you automatically convert template linting errors into future to-do tasks. You can learn more about this feature in the [Official Ember Blog](https://blog.emberjs.com/how-to-todo-in-ember-template-lint/).

Second, Chris Manson created [`lint-to-the-future`](https://github.com/mansona/lint-to-the-future), which turns all your errors into code comments that ignore the rule. When you fix a warning, remove the code comment. You can also visualize and measure your progress with some graphs!

### Our approach

Although 100+ linting errors sounds like a lot, most of them were the same error over and over again. We could see them by running `yarn lint`.

We wanted to avoid making too many unnecessary changes in this app until after our upgrade step was over, so we turned a whole bunch of rules off in our `.eslintrc`. It makes sense that we had to turn off these rules, since many of them were telling us about how to turn our app into an Octane-style app. They will be useful after we finish the dependencies upgrade! We can learn more about each rule by searching for the rule name in the [`ember-cli-eslint`](https://github.com/ember-cli/ember-cli-eslint) source code.

```js
// excerpt from .eslintrc
  rules: {
    'ember/no-jquery': 'off',
    'ember/no-jquery': 'off',
    'no-console': 'off',
    'ember/no-new-mixins': 'off',
    'ember/no-mixins': 'off',
    'ember/native-classes': 'off',
    'ember/require-tagless-components': 'off',
    'ember/no-test-this-render': 'off',
    'ember/no-classic-classes': 'off',
    'ember/no-get': 'off',
    'ember/no-actions-hash': 'off',
    'ember/no-classic-components': 'off',
    'ember/no-private-routing-service': 'off',
  },
```

We only needed to ignore two rules in our `.template-lintrc.js`, we only needed to add two rule ignores. We can learn all about why these rules exist by searching for the rule name in [`eslint-plugin-ember` source code](https://github.com/ember-cli/eslint-plugin-ember). The file name matches the rule that shows up in the warnings. You can find it quickly on GitHub by pressing the letter `T` from the repository's main page and typing in the name of the rule.

```js
// excerpt from .template-lintrc.js
rules: {
  'no-link-to-positional-params': false,
  'require-input-label': false,
  // and some more rules that were already there
}
```

[`require-input-label`](https://github.com/ember-template-lint/ember-template-lint/blob/master/docs/rule/require-input-label.md) is an example of a linting rule that helps you discover accessibility issues in your app! It warns you if you have an input element that lacks an associated label element. In our case, this warning was a false hit - we found a bug! We [reported the bug](https://github.com/ember-template-lint/ember-template-lint/issues/1835#issuecomment-857223622) and linked to the public example of it.

[`no-link-to-positional-params`](https://github.com/ember-template-lint/ember-template-lint/blob/master/docs/rule/no-link-to-positional-params.md) was telling us not to do links in this style: `{{link-to "About Us" "about"}}`, Instead, we should do `<LinkTo @route="about">About Us</LinkTo>`. We can definitely handle this later.

Once our linting tests were passing, we moved on to the next step!

## Understanding application test failures

When you do an upgrade like this, there are some common sources of failures. It's helpful to ponder thus line of questioning below for a little when you run into a test that is tricky to fix:

- Did I make a mistake during the upgrade?
- Did the upgrade uncover a bug that was hidden previously?
- Were there any breaking changes in my dependencies?
- Was my app relying on a bug in Ember that was fixed?
- Did my app rely on private API methods?

After pondering this list, now I can move to the next level:

- What evidence do I have that my hypothesis is correct?
- Do I need to update a test, update something in my app, or both?
- If I reread the test failure again, do I get any new insights?

### Our test failures

Here's an example test failure we worked through:

```text
Error: Element not found when calling `fillIn('#ember-basic-dropdown-content-ember
1246 .ember-power-select-search-input[type=search]')`.
  at http://localhost:7357/assets/test-support.js:35450:15
  at async selectSearch (http://localhost:7357/assets/test-support.js:39374:9)
  at async Object.<anonymous> (http://localhost:7357/assets/tests.js:66:7)
```

In this case, the question that jumps out at us is "Were there any breaking changes in my dependencies?" We upgraded from `ember-power-select` version `2.3.5` to `4.1.6`. Following the rules of semantic versioning, we can see that there have been two releases with breaking changes. Time to check the release notes of the library, in [CHANGELOG.md](https://github.com/cibernox/ember-power-select/blob/master/CHANGELOG.md). If you don't see a changelog for the library you are using, look at the Releases section on GitHub instead.

By reading the changelog, we could see that there was a new required attribute to add to the component. Adding `@searchEnabled={{true}}` to our power select dropdowns was all that was needed.

Another test failure we saw was this:

```text
not ok 66 Chrome 91.0 - [196 ms] - Acceptance | document title: is of format className - version - Ember API Docs
  ---
    actual: >
        Ember Api Docs
    expected: >
        Container - 1.0 - Ember API Documentation
```

This was an example of a mistake we made while upgrading. The test that was failing was checking the page title of a certain URL in the app. A page's title is important for SEO, accessibility, and overall user experience. It's part of the text that shows up in a search result. It is the text that shows up at the top of your browser tab. And for people who use assistive tech, it helps them know what page they are on when they flip through tabs. You can learn more about page titles in [the Ember Guides](https://guides.emberjs.com/release/accessibility/page-template-considerations/#toc_page-title).

What happened was that when the upgrade diff was applied, it added something like `{{page-title "EmberApiDocs"}}` to the `application.hbs` template. Community member [prakashchoudhary07](https://github.com/prakashchoudhary07) discovered that the page title was being set in another, more sophisticated way in this app. Therefore, we could delete the page title helper, and the test passed.

Lastly, there was an error due to reliance on private API. One test was using private APIs on the router inside the test only. When I had tried to upgrade the test to use the router service instead, the method was no longer available. So, I undid the change I made during linting fixes and instead added an ignore to the `eslint.rc`. We can deal with that issue in another PR!

## Up next

Now, all our tests are all passing! Next in this series, we will run some Octane codemods and refactor some components that are tough to work with. Thanks for reading!
