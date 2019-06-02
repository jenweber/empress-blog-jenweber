---
title: Ember.js Octane in Five Minutes
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Sat Jun 01 2019 23:17:09 GMT-0400 (EDT)
tags:
  - new
---

Octane is an upcoming Edition of Ember.js. Once it's out, `ember new` is going to show you an app like you haven't seen before. Here's a quick rundown of everything you need to know if you are an existing Ember dev.

It's a lot to take in, especially when you're used to Ember always looking the same year after year, but I think you'll enjoy the results.
We don't have enough time to get into why these things exist, but if you have questions, [ask away](https://emberjs.com/community/). 

If you are new to Ember, check out [this article instead](https://medium.com/ember-ish/faqs-about-ember-js-in-2019-64efabbf84e6).

## Impact on existing apps

Nobody panic. Octane's features are done through minor releases.

Existing apps can keep doing their regular upgrade cycle, as they did before. Although Octane looks like lots of changes to Ember, they are provided as opt-in, non-breaking features. You would only see differences if you generated a new app, which has the Octane features all enabled.

## Upgrading existing apps

If you want an existing app to be "Octanified," you will need to change some configuration settings in your app, to do things like disable JQuery and enable Native Class Components as the default. From there, you'll be able to gradually change your app's syntax, wherever you feel like doing so. For the most part, you can use a blend of features. Classic and Native Class Components can coexist.

## What will be different in freshly generated apps

While playing around with the Octane preview/pre-release, here are the five areas where I needed to sit down and really learn them, not just copy/paste:

1. Native Classes
2. Decorators
3. Using `@tracked` instead of Computed Properties
4. Element modifiers instead of component lifecycle hooks
5. Enforced data down, actions up

An app created with `ember new` after Octane is released will be using all these features above, and more.

### Native classes

Whenever you import Component from `@glimmer/component` instead of `@ember/component`, you'll be in Native Class land. Classes are now a [feature of JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/class). If you find Class syntax confusing, it may be helpful to experiment with them some outside of Ember. This way, you'll have a clear picture of what is Ember and what is regular JavaScript.

Please note, you don't have to go learn Glimmer, even though the import comes from there.

Here's what a Component looks like using the new form:

```js
import Component from '@glimmer/component';
import { action } from '@ember/object';

export default class HelloButton extends Component {
  text = 'Hello, world!';

  @action
  sayHello() {
    console.log('Hello, world!');
  }
}
```

Pretty weird, huh? No commas. No nesting inside an `action` object. Now actions are done using Decorators.

Components, Routes, Controllers, Services, Ember Data Models, etc. will all be available as Native Class imports.

### Decorators

Decorators start with an `@`. They are already popular in tools like Angular and TypeScript, and are a Stage 2 JavaScript language feature. You can read about the general concept [here](https://github.com/tc39/proposal-decorators). If you create a Native Class version of a Component, Service, Controller, etc., then you can use decorators inside it.

The most important decorators to know about and use are `@action` and `@tracked`. Actions work more or less the same as they did before. `@tracked` is something new, and you'll use it instead of Computed Properties.
Keep reading for more information on that. You can also make your own Decorators if you're feeling fancy.

### Using `@tracked` instead of Computed Properties

Just to be clear, Computed Properties aren't _gone_ from Ember. You can keep using them in "Classic" (aka non-Native Class) Components, Controllers, Routes etc. in your app. But if you are switching to Native Classes, you have two options for refactoring your computed properties.

As a first pass, you could use the `@computed` decorator inside your Native:

```js
import { computed } from '@ember/object';
// ...
  @computed('firstName', 'lastName')
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
// ...
```

But really what you want to use is `@tracked`. You decorate (or label) the properties that you want to be watched for changes:

```js
import { tracked } from '@glimmer/tracking';
//...
  @tracked firstName;
  @tracked lastName;

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
// ...
```

I can use `{{this.fullName}}` in my template, just like before.

### Element Modifiers instead of lifecycle hooks

Ember Components have lifecycle hooks like `didInsertElement` that are popular places for shenanigans like inserting charts, registering features from non-Ember UI libraries, etc.

In Native Class Components, instead of declaring a `didInsertElement` method in your JavaScript file, you tell the template which action to run when the element renders:

```
<div {{did-insert this.someAction}}>
  ...
</div>
```

When that div renders, the action `someAction` will be called automatically. Pretty neat! As someone who used that hook a lot when using data vis tools like D3, I love this.

### Enforced Data Down, Actions Up

In "classic" components, if you pass a property from a parent component to a child component, the child component could make changes to it, and those changes would affect every other place that same property was used. The change would show up in the parent, in other components, etc. This was referred to as "two way binding."

Now, for Native Class components, two way binding is no more.

If a child component tries to set a value of a property, it can only set the value on itself. The property from the parent is unchanged.

To give an example, let's say we have a property `penguins` passed from a parent component into a child component, and its value is zero.

```hbs
<ChildComponent @penguins={{this.penguins}} />
```

In the child component, we try to set the value of `penguins` to 5, by calling this action:

```
@action
morePenguins() {
  this.penguins = 5
}
```

In the child component, here's what we get displayed:

```hbs
{{@penguins}} <!-- 0 - this is the property from the parent -->
{{this.penguins}} <!-- 5 - this is the component's own property -->
```

In the parent component, the value of `penguins` is still zero too.

To change the value of `penguins` on the parent, we'd have to pass down an action that the child component can use:

```hbs
<ChildComponent @penguins={{this.penguins}} @someAction ={{this.evenMorePenguins}} />
```

## Learn more

The goal of this article was to cover what to expect, in very brief terms. Ready for more?

- Learn what editions are [here](https://emberjs.com/editions/)
- Octane-specific official resources are [here](https://emberjs.com/editions/octane), including how to test out the preview/pre-release
- There are excellent, in-depth resources written mainly by Chris Garret, both for the Guides and a [personal blog](https://www.pzuraq.com/).
- Ask questions in the [Community Chat](https://emberjs.com/community/)

Good luck learning! 
