---
title: Building an Octane-style input in Ember
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Wed Apr 08 2020 23:17:09 GMT-0400 (EDT)
tags:
  - new
---

Building a text input from scratch in Ember.js
Ember.js provides the `<Input />` component out of the box, but sometimes it falls short of a developer's needs for functionality, data flow, and accessibility. In this tutorial, you'll learn how to use Ember's APIs and patterns to interact with native input elements. Even if you never need to make your own input, you'll leave with a better understanding of how to use the DOM within an Ember app.

## Why make your own input?

In Ember Octane, there's a heavy emphasis on data down, actions up for interacting with the DOM. However, the built-in `<Input />` component follows the old two-way-bindings pattern. This is a fine and ok thing to do; I still use Input in all my apps! I really like two-way-binding for inputs! However, it can be a bit counterintuitive to do things like form validation in the Octane style, when you are still using the built-in Input.

Another reason to make your own input is for accessibility reasons. Unfortunately, every popular front-end framework has its share of accessibility problems. Although Ember is doing better than other frameworks in some regards, there are still some things to work through. If you make your own input component, you can programatically enforce that whenever a developer on your team uses it, they pass in some label text.

Lastly, Input has some confusing and inconsistent behavior, since it has to be backwards-compatible. It's in need of a rewrite, but that can't happen without introducing breaking changes into the framework. So, for now, we're stuck with it, at least until someone writes an RFC, gets it accepted, and does the work.

## Native input basics

To understand Ember's `<Input />`, it helps to understand what a normal, native input can do, and what it needs in order to work well for all users. This information will be important to know when we build our own input.

### 1. inputs and labels are inseparable pals.

Whenever you create an input, it ought to have an associated label. There are two common styles for this:

```html
<!-- put the input inside the label -->
<label>
  First Name
  <input type="text" />
</label>

<!-- reference the label by its id-->
<label for="input-first-name">
  First Name
</label>
<input id="input-first-name" type="text" />
```

In the second example, you have to be very careful to not re-use the ID assigned to the inputs and labels.

### 2. inputs have many different possible attributes

A native input can have many different attributes, such as required,aria-labelled-by, id, class, type, disabled, and more. See the input docs on MDN for the full list.

### 3. inputs have automatic accessibility behavior

Whenever you use the label and input elements, you get some excellent features automatically. They include keyboard navigation, screen reader support, and more. For this reason, you should always use the input element for inputs, not write a weird div that kinda acts like an input.

### 4. Interacting with the input fires events

Every input has events that fire whenever someone types in it. For example, there is an oninput event as well as onchange. Whenever you tie directly into events, it's important to check your browser compatibility needs. For example, oninput isn't supported in IE9.

### 5. Follow semantic HTML to avoid UX traps

If you find it difficult to write valid markup and styling for an input, that's a red flag that your designs might have accessibility problems. This pain can save you from making big mistakes. For example, many developers work from designs where the label is hidden and placeholder text communicates the "label." This was popularized by Material Design, and it had some serious UX problems for all users. The MDN docs go as far as to say that you should not use placeholders at all, if you can avoid them.

## Writing vanilla JavaScript inputs

In Ember, you can't use things like script tags in the templates, but it's still good to see how one would do this in a vanilla HTML and HTML + JavaScript settings. MDN has a good tutorial on this topic.
HTML only

A regular input can send data without any JavaScript at all if it is part of a form:

```html
<form action="/api/names" method="post"> 
  <label for="name">Name:</label>
  <input type="text" id="name" name="user_name" />
  <button type="submit">Send your message</button>
</form>
```

Clicking "submit" in the example above would send the name to the `/api/names` endpoint. So, if your app was hosted on `https://my-forms.example.com`, it would send a request to `https://my-forms.example.com/api/names`.

## HTML + JavaScript

You can write event listeners in JavaScript that respond to user interactions, and attach them to specific inputs.

For example, this HTML and JavaScript code will print the current value of the input to the console:

```html
<label for="name">Name:</label>
<input type="text" id="name" name="user_name" />
<script>
  let myInput = document.querySelector('#name')
  myInput.addEventListener('input', function(e) {
    console.log(e.target.value)
  })
</script>
```

You can even call an anonymous or specific function from the HTML, although this is considered bad practice for maintainability, testing, linting, etc in many codebases:

```html
<input type="text" oninput="(function () {console.log('ew, gross') })()" />
<input type="text" oninput="someOtherFunction()" />
```

## Using Ember's built-in Input and two-way binding

Now let's take a look at what Ember's own built-in `<Input />` does. The Input component, with a capital I, gives you some additional features and shortcuts compared to using a regular old input with a lowercase i. Most importantly, the input's value (what someone has typed into the box) can be "bound" to a property of the component. When the value changes as a user interacts, so does a property available to the JavaScript side. This is called "two way binding" and it's a very common pattern in pre-Octane Ember apps.

In our template, we can use the Input component and set its attributes. Don't forget to include a label! This example is for Ember 3.15+. Note that if you copy-paste this, you'll get errors until you also copy-paste the JavaScript.

```handlebars
<label for="name">
  Name
</label>
<Input type="text" id="name" value={{this.username}} />
<button {{on "click" this.submitName}}>
  Submit
</button>
```

Now, in the JavaScript file for this component, we can indicate that username should be tracked for changes, and write an action for the Submit button. Just like in the vanilla HTML + JavaScript example, we'll print the input value to the console when someone clicks submit:

```js
import Component from '@glimmer/component';
import { action } from '@ember/object';
import { tracked } from '@glimmer/tracking';
export default class MyInputComponent extends Component {
  @tracked username = '';
  @action
  submitName() {
    console.log(this.username)
  }
}
```

Components are often meant to be reusable throughout an app, and it would be bad if we reused a component with hard-coded IDs. You might end up with multiple reused, non-unique ids on the page, which breaks your app. So, we can add in guidFor to generate a unique ID for our component to use. We'll save the results of guidFor to a new property called uniqueId, and use it in the markup:

```handlebars
<label for="name-{{this.uniqueId}}">
  Name
</label>
<Input type="text" id="name-{{this.uniqueId}}" value={{this.username}} />
<button {{on "click" this.submitName}}>
  Submit
</button>
```

Component JavaScript that generates the unique ID:

```js
import Component from '@glimmer/component';
import { action } from '@ember/object';
import { tracked } from '@glimmer/tracking';
import { guidFor } from '@ember/object/internals';
export default class MyInputComponent extends Component {
 uniqueId = guidFor(this);
 @tracked username = '';
 @action
  submitName() {
    console.log(this.username)
  }
}
```

## Making our own text input with a lowercase i, level one

Ok, so now you know the basics of how to interact with an input with a lowercase i using HTML and JavaScript, and also how to use Ember's Input with an uppercase I with two-way binding. Now it's time to make our own implementation that does not have two-way binding. This is a toy example, and in the following section, we'll add more features that help ensure that whenever it is used, it is accessible.
Let's start with the markup we had before, but make it a lowercase i and remove the value binding:

```handlebars
<label for="name-{{this.uniqueId}}">
  Name
</label>
<input type="text" id="name-{{this.uniqueId}}" />
<button {{on "click" this.submitName}}>
  Submit
</button>
```

We have a few different paths that we could take, but let's start with the simplest. Ember provides the {{on}} helper for attaching JavaScript behavior to DOM events. Let's watch for the input event to fire, and write an action handleInput to call when it happens. Like before, you will need to copy-paste both the template and the JavaScript before it will work:

```handlebars
<label for="name-{{this.uniqueId}}">
  Name
</label>
<input type="text" id="name-{{this.uniqueId}}" {{on "input" this.handleInput}} />
<button {{on "click" this.submitName}}>
  Submit
</button>
```

The component JavaScript, which will print the input's value to the console:

```js
import Component from '@glimmer/component';
import { action } from '@ember/object';
import { tracked } from '@glimmer/tracking';
import { guidFor } from '@ember/object/internals';
export default class MyInputComponent extends Component {
 uniqueId = guidFor(this);
 @tracked username = '';
 
 @action
 handleInput(e) {
   console.log(e.data)
 }
 @action
  submitName() {
    console.log(this.username)
  }
}
```

Notice that our handleInput receives an argument, which I named el in the example above.  e is short for "event." I could have called it "bananas" if I wanted to instead. Anyway, whenever you use `{{on}}` to call a component's action, it always passes the event itself as the first argument. Our event is input. If we console.log el, we'd see a huge object for InputEvent. The InputEvent has a property called data that contains the value someone typed into the input!

Now that we have that input entry, we could do things like run validation and display helper text, set a property on this component, or call an action passed into the component from a parent. "Validation" in this context means something like checking to see if a user entry has the right format, and displaying a message to help them edit their information. For example, maybe the input has a character limit, and you want to let the user know that they are over it, and they need to do some editing before submitting.

## Making our own text input with a lowercase i, level two

In the previous section, we made a toy example. Now it's time to get to the real challenge - writing an input that requires you to have a label, and handles its ids gracefully and automatically.

I don't have a one-size-fits-all implementation for you (yet). However, my apps really only had three different flavors of inputs anyway - a basic text input, a text input with labels hidden because I had no choice in the design, and then non-text inputs whose attributes varied wildly. It's up to me to figure out which kind of input is used the most, and design an easily reusable component for that use case first. This would have the greatest impact in the shortest amount of time. Then, I should work through all the other cases throughout my app.

First, let's make our text input more generic, and have it rely on properties that are passed in. I'll explain the rationale for each one in a moment:

```handlebars
<label for="input-{{this.uniqueId}}" class={{@labelClass}}>
  {{@labelText}}
</label>
<input type="text" id="input-{{this.uniqueId}}" class={{@inputClass}} {{on "input" @handleInput}} value={{@initialValue}} />
```

Next, let's define the JavaScript that provides the uniqueId and enforces that @labelText is passed in:

```js
import Component from '@glimmer/component';
import { action } from '@ember/object';
import { tracked } from '@glimmer/tracking';
import { guidFor } from '@ember/object/internals';
export default class MyInputComponent extends Component {
 uniqueId = guidFor(this);
 
 constructor() {
   super(...arguments);
   if (!this.args.labelText) {
    assert('All inputs must have labels! MyInputComponent requires passing in labelText, i.e. <MyInputComponent @labelText="First Name" />')
  }
 }
}
```

Here's my rationale for each attribute passed in, or not passed in:

`labelText`: I will want to customize the label displayed every time I show this input
labelClass and inputClass: My input and its label need to look different depending on their context in my app.

`initialValue`: I commonly need to pre-fill the input with the data that's available. For example, I might already know the username from when someone logged in. I call it initialValue to help prevent a developer from thinking that it is two-way-bound.

`handleInput`: following data down, actions up, we should call a method passed in from the parent in order to modify data.

`id`: I did not make id a property to pass in, because it's best practice to not use ids in CSS.
type: I also did not pass in type, since I prefer having components that are easy to understand and use, instead of a component that is complicated but ultra-reusable. I try to ask myself whether the component I'm making would need to be explained to a new hire, or if it would be self-explanatory.

_Sidenote: I could have also used an Input with a capital I in my template example above, and written it to use two-way binding. I might go that route if I am refactoring an app that already makes use of two-way-bound inputs._

## Making this happen for everyone

Do you think that this sort of thing should be part of Ember's out-of-the-box toolset? The best solutions are those where you don't need to understand a lot in order to do the correct thing!
There is a group of developers working to improve Ember's accessibility right now, and they are working through this issue to problem solve and fix outstanding problems. If you would like to participate, speak up in the #st-a11y channel in the Ember Community Discord.

## Another approach to handling DOM events

Ember Octane popularized the use of modifiers, which provide a way to attach events or behavior as an element renders. There are some examples of modifiers in the ember-modifier library's README.
One could write a modifier that attaches and tears down event listeners for an input. This could be helpful if you need something that is both really fancy and reusable.

## Yet another way to handle events
Ember contributor and core team member Chris Garret has written an addon that provides a helper called box. This addon isn't part of Ember itself, and you would have to install it yourself in order to use it.

The `{{box}}` helper provides a way to add two-way-binding-like behavior to templates in a way that is concise and element-agnostic.Even if you don't plan to use box, it may be helpful to read it over as a way to boost your own understanding of the types of abstractions that people could create for handling DOM interactions.

## In conclusion…

It's worth stating here that there's nothing wrong with using Input in your app, but there is something wrong with not using labels and aria attributes appropriately. It is up to you and your team whether you choose to enforce correct HTML through programmatic means, or just try to catch it in peer reviews.

I strongly recommend going the programmatic route whenever you can. This allows your fellow developers to learn on your own, and prevents the case where one developer is exhausted by enforcing standards while their colleagues grow annoyed. I have been on both sides of that table. I really prefer when the code helps me fix my own mistakes.

Good luck!