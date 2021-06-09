---
title: Remodeling an Ember App - Package Updates
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Wed Jun 09 2021 09:10:08 GMT-0400 (Eastern Daylight Time)
tags:
  - new
---

Today's topic is upgrading dependencies on an older Ember app.
This is a real-world app and the issues we face will be different from your apps, but you can learn the overall strategy and debugging approaches,

This is Part 2 of a series of blog posts. We're on a journey together to remodel an older Ember app, incrementally bringing it up to date with the
latest and best Ember and Ember Data patterns.


## About the series

We will walk through updating a complex, real-world app,
[ember-api-docs](https://github.com/ember-learn/ember-api-docs),
which is the front end of [api.emberjs.com](https://api.emberjs.com). Along the way, you will learn how to approach this process
for your own apps - updating dependencies, debugging build errors, migrating to Octane syntax, writing better Ember Data serializers & adapters... there's a lot to do!

For this series, I'm pair programming with Chris Thoburn, aka [@runspired](https://github.com/runspired), who is known for
his work on Ember Data. He has over 500 commits and some great
debugging skills that you and I can learn from.

## Deciding how to approach upgrading dependencies

There are two main approaches Ember developers take, either:

1. Upgrade addons
2. Upgrade core Ember and Ember Data dependencies

Or, they approach this in the reverse order:

1. Upgrade core Ember and Ember Data dependencies
2. Upgrade addons

The path you choose depends on your app - how many addons does it have? How outdated are they?

In our case, we decided on the latter. We didn't want to introduce too many changes at once if we didn't have to, and we were pretty confident in our ability to sort through addon-specific errors.

## Running the Ember and Ember Data upgrade

This app was on 3.16. The latest Ember and Ember Data version is 3.26.

We can do the upgrade with the help of `ember-cli-update`, which will apply the blueprint used for fresh apps.

```
npx ember-cli-update
```

You can read a whole bunch about upgrading in the [Ember Guides](https://guides.emberjs.com/release/upgrading/) and [Ember CLI Guides](https://cli.emberjs.com/release/basic-use/upgrading/),
so we won't go into too much detail here.

We reviewed each file that was modified first, to confirm that we wanted those changes.

Then, we walked through all merge conflicts and resolved them.

Next, we installed our new dependencies with `yarn install`.

Finally, we started the Ember server with `npx ember serve` and hit our first bug.

_(Why `npx ember serve`? `npx` ensures that the Ember CLI version used by 
your app is the one specified in package.json, and not your globally
installed version. It usually doesn't matter, but when it does, it's often
when working on upgrade tasks like this!)_

## Debugging a build failure

Here's the build failure we saw in the console after we started up our server:

```
Build Error (broccoli-persistent-filter:Babel > [Babel: @ember/ordered-set]) in @ember/ordered-set/index.js

Duplicate plugin/preset detected.
If you'd like to use two separate instances of a plugin,
they need separate names, e.g.

  plugins: [
    ['some-plugin', {}],
    ['some-plugin', {}, 'some unique name'],
  ]

Duplicates detected are:
[
  {
    "alias": "/Users/jweber/projects/ember-api-docs/node_modules/ember-compatibility-helpers/comparision-plugin.js",
    "options": {
      "emberVersion": "3.26.1",
      "root": "/Users/jweber/projects/ember-api-docs",
      "name": "@ember/ordered-set"
    },
    "dirname": "/Users/jweber/projects/ember-api-docs",
    "ownPass": false,
    "file": {
      "request": "/Users/jweber/projects/ember-api-docs/node_modules/ember-compatibility-helpers/comparision-plugin.js",
      "resolved": "/Users/jweber/projects/ember-api-docs/node_modules/ember-compatibility-helpers/comparision-plugin.js"
    }
  },
  {
    "alias": "/Users/jweber/projects/ember-api-docs/node_modules/ember-compatibility-helpers/comparision-plugin.js",
    "options": {
      "emberVersion": "3.26.1",
      "root": "/Users/jweber/projects/ember-api-docs",
      "name": "@ember/ordered-set"
    },
    "dirname": "/Users/jweber/projects/ember-api-docs",
    "ownPass": false,
    "file": {
      "request": "/Users/jweber/projects/ember-api-docs/node_modules/ember-compatibility-helpers/comparision-plugin.js",
      "resolved": "/Users/jweber/projects/ember-api-docs/node_modules/ember-compatibility-helpers/comparision-plugin.js"
    }
  }
]
```

Not very nice.

We did `yarn why @ember/ordered-set` to confirm that only one version of
that dependency was in use:

```
yarn why @ember/ordered-set

yarn why v1.22.10
[1/4] 🤔  Why do we have the module "@ember/ordered-set"...?
[2/4] 🚚  Initialising dependency graph...
[3/4] 🔍  Finding dependency...
[4/4] 🚡  Calculating file sizes...
=> Found "@ember/ordered-set@4.0.0"
info Reasons this module exists
   - "ember-data" depends on it
   - Hoisted from "ember-data#@ember#ordered-set"
   - Hoisted from "ember-data#@ember-data#record-data#@ember#ordered-set"
info Disk size without dependencies: "44KB"
info Disk size with unique dependencies: "1.98MB"
info Disk size with transitive dependencies: "47.57MB"
info Number of shared dependencies: 151
✨  Done in 1.83s.
```

Chris hypothesized that this was due to floating dependencies.
Yarn and NPM let developers specify a range of acceptable dependency versions.
Sometimes there are mismatches. It's often a good initial step to try
deleting node modules and the lock file, as a small experiment, and then try again:

```sh
rm -rf node_modules
rm yarn.lock
yarn install
npx ember s
```

That resolved the error about duplicate plugins!

## On to the next error - dependencies of dependencies

Next, we saw some errors about `ember-popper`.

We looked to see what was using `ember-popper` using `yarn why`.
One of our dependencies, `ember-styleguide` used it. We looked at
a more recent version of `ember-styleguide`, in the `package.json`.
The latest ember-styleguide didn't even use popper,
so upgrading ember-styleguide got us over this hurdle.

We also saw an error about `ember-basic-dropdown`.
`yarn why` told us it was used by `ember-power-select`.
Upgrading `ember-power-select` and following instructions in the
CHANGELOG fixed that issue.

Now, when we did `yarn install` and `npx ember s`, the app built successfully.

## Debugging runtime errors

Now when we visted `localhost:4200`, we saw something strange.
The app would render for a moment (thanks to Fastboot),
but then when the app's JavaScript loaded in fully, it crashed.

Apps that use Fastboot show their errors in the terminal running the server,
_not_ the browser console. So, to get a closer look at what was going on,
we temporarily turned off fastboot by visiting `http://localhost:4200?fastboot=false`.

There were two errors in the browser console:

One:

```
Error occurred:

- While rendering:
  -top-level
    application
      es-header
        es-navbar
          bs-navbar
            bs-navbar/content
              bs-collapse
                bs-navbar/nav
                  search-input
                    ember-tether
                      search-input/dropdown
```

Two:

```
Uncaught (in promise) ReferenceError: process is not defined
    <anonymous> Ember
    js vendor.js:173080
    __webpack_require__ vendor.js:172838
    <anonymous> Ember
    js vendor.js:173069
    __webpack_require__ vendor.js:172838
    <anonymous> Ember
    js vendor.js:173126
    __webpack_require__ vendor.js:172838

...and a whole lot more
```

We wanted to see where the problem was coming from, as step one.
We commented out the `es-header` component and saw the app render.
Then we commented out pieces of the app until we got to
identifying `search-input` as the culprit.

The search feature of this app comes from a third-party library.
Is something in there using `process`?

I enabled "Break on exceptions" in my browser's debugging tools.
This showed us that indeed, `process.env` was being used in a third party script.
But why did this work before? What else had changed?

## Debugging webpack

The webpack errors are a clue about what changed.
When we updated a bunch of things in `package.json` using `ember-cli-update`,
one of them was `ember-auto-import`. This package uses webpack to
bring npm dependencies into an Ember app with zero config,
and we changed its versions.

One thing that changed was that certain polyfills were no longer included.
This was outlined [in the project README](https://github.com/ef4/ember-auto-import#i-upgraded-my-ember-auto-import-version-and-now-things-dont-import-what-changed)
along with steps to fix it.

We added the following to our `ember-cli-build.js`:

```
autoImport: {
  webpack: { 
    node: { 
      process: 'mock'
    }
  }
},
```

Now, the app rendered successfully! We removed `?fastboot=false` from our
localhost URL, and saw that it was working fine in that mode too.

## Up next

So, the app rendered, but the tests are failing.
Coming soon, we'll tackle those test failures.
