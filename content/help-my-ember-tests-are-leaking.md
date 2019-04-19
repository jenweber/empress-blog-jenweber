---
title: Help, my Ember acceptance tests are leaking!
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Thu Apr 18 2019 20:08:12 GMT-0400 (EDT)
tags:
  - new
---

Are you stuck debugging async in your tests?
I'll cover one use case, including what the symptoms looked like, the root problem, and the solutions.
The code samples are all Ember 3.x.

## The symptoms

Today, I was working on fixing the following Ember Data deprecation:

> DEPRECATION: Attempted to call store.serializerFor(), but the store instance has already been destroyed. [deprecation id: ember-data:method-calls-on-destroyed-store]

Everywhere I could, I put checks like `this.store.isDestroyed` so that I could find and skip the offending code, but that didn't help. It always turned out to be `false`, and therefore useless to my debugging.

I thought, maybe this is some weird Ember Data thing, and I should look into that more.

_Spoilers - it wasn't a weird Ember Data thing._

I asked for help on the [Ember Community chat](https://emberjs.com/community/), and Chris Thoburn aka [Runspired](https://twitter.com/Runspired) helped me out.
I almost always leave conversations with him by thinking, I need to write a blog post about that.

## The problem

Here's what was going on. 

My acceptance test was leaking state, meaning that some asychronous code was resolving after the test had already moved on and reset some resources for the next test.
This process of bringin the test back to a "clean slate" is referred to as "test teardown" in Ember and other JavaScript environments.

In my case, an `afterEach` hook that destroyed the store was happening before the code was actually done.
However, you might see the same sort of thing if you use `this.anything` after the testing teardown steps.

## Finding the cause

The test itself was fairly normal, and used Ember's async test helpers just like it was supposed to:

```js
test('track field dirtiness in owned, related records', async function(assert) {
  await visit('/hub/posts/1');

  let reviewStatusActionTrigger = findTriggerElementWithLabel.call(this, /Comment #1: Karma/);
  await click(reviewStatusActionTrigger);

  let karmaInput = findInputWithValue.call(this, '10');
  await fillIn(karmaInput, '9');
  assert.dom('[data-test-cs-version-control-button-save="false"]').exists('Save button is enabled');
  assert.dom('[data-test-cs-version-control-button-cancel="false"]').exists('Cancel button is enabled');
});

```

The test code was doing what it should. Chris confirmed that it was my app code causing the problem, not an Ember Data issue, nor this test. But where could it be hiding?

My first strategy was that I commented out parts of the test until I figured out exactly which interaction caused the problem. Something happened after doing the `fillIn`.

I followed the code to see what actions were active.
It turns out that the component was using a Service to do `fetch` data, and then using the Ember Data serializer to parse the response:

```js
async validate(model) {
  let responses = this.toValidate.map(async (record) => {
    let { url, verb } = this._validationRequestParams(record);
    let response = await fetch(url, {
      method: verb,
      headers: this._headers(),
      body: JSON.stringify(record.serialize())
    });
  });

  let json = await Promise.all(responses);
  // do more work
}
```

The offending line was `record.serialize()`, which needs Ember Data, and my tests weren't waiting for the result.

## Fixing it

I could think of three possible paths forward:

- Stub the service
- Make the tests wait (how???)
- Check the Ember environment in my app code (ew)

Stubbing the service was easiest in this case.
When you "stub" a service, you make a fake service that the test uses.

The actual code for the service used by the component was over 200 lines, but the component was only using three functions from it!

There are just a few steps to stubbing a service:

### 1. Move this troublesome test into its own file.

My acceptance test had many smaller tests within it, but only one had async problems. I didn't want to have to stub every service function used by every one of those tests, so I moved the deprecation-triggering test into its own file.

I can run this test alone with `ember test --server --module "Acceptance | your test name"`. The module is whatever is specified in the test: `module('Acceptance | your test name', function(hooks) {...})`. 

Alternately, to run my test alone, I can use `ember test --server --filter "some term"` where some term is found in the line `test('here is some term', async function(assert) {...}`

The `--server` means that my tests will automatically re-run when I make changes, and it runs much much faster than doing `ember test` repeatedly by hand.

### 2. Create the stub in the acceptance test. 

Next I made my fake service.
I looked at the component to see which functions it used.
I studied those functions to see what kind of data they should return, and console logged the things they returned when the test was runnning.
The asynchronous methods return a promise, and the other methods return data that the test needs to render.

_The example below is Ember 3, but if you're using an older version of Ember, [this other article](https://medium.com/ember-ish/how-to-use-a-service-from-an-acceptance-test-in-ember-js-6781fee2411b) might help you out with service stubbing. Ignore the pop-ups. There's no paywall._

```js
// my stubbed service

let StubCardstackDataService = Service.extend({
  validate() {
    return Promise.resolve({})
  },
  getCardMeta() {
    return 'Comment #1'
  },
  branches() {
    return []
  },
  fetchPermissionsFor() {
    return Promise.resolve({mayUpdateResource: false, writableFields: ['karmaValue', 'karmaType']})
  }
})
```

### 4. Inject the test in the `beforeEach` hook.

This special function runs before every test. Instead of using my real `cardstack-data` service, the component will use my fake service when it runs the test. In order to make that happen, I have to register my service.

```js
hooks.beforeEach(function() {
  this.owner.register('service:cardstack-data', StubCardstackDataService);
});
```
Make sure the information that you give to `register()` matches the name of the service used in the Component.

### The final result

If you want to see how the stub is used in the context of the whole test, check out resulting PR [here!](https://github.com/cardstack/cardstack/pull/749/files)

## Takeaways

One thing I learned from this experience was that if I am writing components that have async data fetching behavior, there's a huge benefit to putting that code into a Service. That way, I can stub it easily in my tests!

This isn't the only way to handle this problem - remember, I outlined 3 options earlier, and there are probably more.

My other takeaways are that whenever I can, I should lean on Ember Data and the model hook to do data requests. Sticking closely to the "happy path" in Ember means that you get some async data handling for free. I was working on a project that does not have model hooks because it has no route files (weird, right?), but a normal Ember app has more async-handling features available!

## For more information

Here are some resources that I found helpful while learning and debugging. Thanks to these authors for their great work!

- The offical Ember tutorial's example of [stubbing a service](https://guides.emberjs.com/release/tutorial/service/#toc_integration-testing-the-map-component)
- [Ember Timer Leaks](https://engineering.linkedin.com/blog/2018/01/ember-timer-leaks) by Evan Farina
- [Testing your Ember application in 2018](https://dockyard.com/blog/2018/03/29/testing-your-ember-application-in-2018) by Scott Newcomer. Ok so it's not 2018 but if the async/await in this article is news to you, this is a really great primer
- Similar to the item above, [The future of Ember Testing](https://www.youtube.com/watch?v=8D-O4cSteRk) presented by Tobias Bieniek 

## Thanks!

Thanks for reading and especially thanks to Chris Thoburn for the help!

## About the author

_[Jen Weber](https://twitter.com/jenweber) is web developer and writer based in Boston, USA (she/her or they/them). As a member of the [Ember Core Team](https://emberjs.com/team), she works on code, docs, and contributor engagement for the [Ember.js](https://emberjs.com) front end framework. Jen works at [Cardstack](https://cardstack.com/), where she helps build tools to power Web 3.0. She is a fan of open source and making tech a more welcoming, inclusive industry._
