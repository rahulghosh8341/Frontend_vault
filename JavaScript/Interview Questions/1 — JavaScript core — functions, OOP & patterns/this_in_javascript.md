---
id: this-in-javascript
title: Explain how `this` works in JavaScript
type: quiz
section: "1 — JavaScript core — functions, OOP & patterns"
solved: true
tags:
  - javascript
  - interview
  - functions
  - this
  - oop
solvedDate: 2026-09-09
---

> [!info] Source
> GreatFrontEnd — **Explain how `this` works in JavaScript**

## TL;DR

`this` is a dynamic reference to the context in which a function is executed.

The value of `this` depends mainly on **how the function is called**, not where the function was defined.

The main rules are:

1. With `new`, `this` is the newly created instance.
2. In a class constructor, `this` is the newly created instance.
3. With `call()`, `apply()`, or `bind()`, `this` is explicitly set to the supplied object.
4. In a method call such as `obj.method()`, `this` is the object used to call the method.
5. In a standalone function call, `this` is the global object in non-strict mode and `undefined` in strict mode.
6. Arrow functions do not have their own `this`; they inherit `this` lexically from the surrounding scope.
7. When multiple rules could apply, the higher-priority rule wins.

## Interview Answer (30–60 sec)

`this` in JavaScript refers to the execution context of a function, and its value is determined mainly by how the function is called.

For example, when a function is called as `obj.method()`, `this` refers to `obj`. When a function is called with `new`, `this` refers to the newly created instance. `call`, `apply`, and `bind` can explicitly control `this` for regular functions.

Arrow functions are different because they don't have their own `this`; they inherit it from the surrounding lexical scope.

So the key thing to remember is: **for regular functions, determine `this` from the call site; for arrow functions, look at the surrounding scope.**

## Key Takeaways

- `this` is determined by the **invocation context** for regular functions.
- `obj.method()` → `this` is `obj`.
- `new Foo()` → `this` is the new instance.
- `call`, `apply`, and `bind` can explicitly set `this` for regular functions.
- Standalone regular function:
  - non-strict mode → global object
  - strict mode → `undefined`
- Class methods are strict mode by default.
- Arrow functions use **lexical `this`**.
- `call`, `apply`, and `bind` cannot change an arrow function's `this`.
- If a method is detached from its object, it loses the original method-call context.

## 🧠 Mental Model

Think:

```text
How was this function called?
        |
        +-- new Foo() ----------> new instance
        |
        +-- call/apply/bind -----> explicitly supplied object
        |
        +-- obj.method() --------> obj
        |
        +-- standalone ----------> global / undefined in strict mode
        |
        +-- arrow function -------> surrounding lexical this
```

The most useful interview rule:

> **Regular function → look at the call site.**
>
> **Arrow function → look at the surrounding scope.**

## Common Interview Traps

### 1. Assuming `this` means the object containing the function

```js
const obj = {
  name: 'John',

  showThis() {
    console.log(this);
  },
};

obj.showThis(); // obj
```

But if the method is extracted:

```js
const showThisStandalone = obj.showThis;

showThisStandalone();
```

It is now a standalone function call, so it no longer gets `obj` as `this`.

---

### 2. Thinking arrow functions get `this` from the object

```js
const person = {
  name: 'John',

  sayHello: () => {
    console.log(this.name);
  },
};

person.sayHello();
```

The arrow function does **not** get `this` from `person`. It inherits `this` from its surrounding scope.

---

### 3. Thinking `call()` / `apply()` / `bind()` can change arrow-function `this`

They can change `this` for regular functions:

```js
function showName() {
  console.log(this.name);
}

const person = { name: 'John' };

showName.call(person); // John
```

But not for arrow functions:

```js
const showName = () => {
  console.log(this.name);
};

showName.call(person); // does not make this === person
```

---

### 4. Forgetting strict mode

```js
'use strict';

function showThis() {
  console.log(this);
}

showThis(); // undefined
```

Without strict mode, a standalone regular function call uses the global object.

---

### 5. Confusing a method call with a detached function call

These are different:

```js
obj.method();       // this === obj

const fn = obj.method;
fn();               // standalone call
```

The call expression determines the `this` value.

## Common Follow-ups

### What is the difference between `call`, `apply`, and `bind`?

- `call()` invokes the function immediately and accepts arguments individually.
- `apply()` invokes the function immediately and accepts arguments as an array-like value.
- `bind()` returns a new function with `this` bound; it does not invoke the function immediately.

### Can arrow functions have their `this` changed?

No. Arrow functions inherit `this` lexically and ignore `call`, `apply`, and `bind` attempts to change it.

### What happens to `this` when a method is passed as a callback?

A regular method can lose its original `this` when passed as a callback because it may later be invoked as a standalone function. `bind()` or an arrow wrapper can preserve the desired context.

### What is `this` inside a constructor?

When called with `new`, `this` refers to the newly created object instance.

## My Notes

- `this` is one of the most important prerequisites for the upcoming coding section.
- This directly prepares for:
  - `Function.prototype.apply`
  - `Function.prototype.call`
  - `Function.prototype.bind`
  - constructor/OOP problems
  - method context
  - arrow functions
- **Do not memorize `this` as “the object.”** Memorize it as a value determined by the invocation rules for regular functions.

# Detailed Reference

## `this` keyword

In JavaScript, `this` is a keyword that refers to the current execution context of a function or script.

### Used globally

In the global scope, `this` refers to the global object, such as `window` in a browser.

```js
console.log(this);
```

### Within a regular function call

When a regular function is called standalone:

```js
function showThis() {
  console.log(this);
}

showThis();
```

In non-strict mode, `this` refers to the global object. In strict mode, `this` is `undefined`.

### Within a method call

When a function is called as a method:

```js
const obj = {
  name: 'John',

  showThis: function () {
    console.log(this);
  },
};

obj.showThis();
```

`this` refers to `obj`.

However:

```js
const showThisStandalone = obj.showThis;
showThisStandalone();
```

This is a standalone function call, so `this` is no longer `obj`.

### Within a function constructor

When a function is called with `new`, `this` refers to the newly created instance:

```js
function Person(name) {
  this.name = name;
}

const person = new Person('John');

console.log(person.name); // John
```

### Within class constructors and methods

In a class, `this` refers to the instance:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  showThis() {
    console.log(this);
  }
}

const person = new Person('John');

person.showThis();// Person {name: 'John'}

const showThisStandalone = person.showThis;
showThisStandalone(); // `undefined` because in JavaScript class bodies, all methods are strict mode by default, even if you don't add 'use strict'

```

Class methods are strict mode by default, so a detached class method has `this` as `undefined`.

### Explicitly binding `this`

`call()`, `apply()`, and `bind()` can explicitly control `this` for regular functions.

```js
function showThis() {
  console.log(this);
}

const obj = { name: 'John' };

showThis.call(obj);
showThis.apply(obj);

const boundFunc = showThis.bind(obj);
boundFunc();
```

`call()` and `apply()` invoke the function immediately. `bind()` creates a new function with the specified `this`.

### Within arrow functions

Arrow functions do not have their own `this`. They inherit `this` from their surrounding lexical scope.

```js
const obj = {
  name: 'John',

  showThis: function () {
    const arrowFunc = () => {
      console.log(this);
    };

    arrowFunc();
  },
};

obj.showThis();
```

Here, the arrow function inherits the `this` value from `showThis()`.

Because arrow functions have lexical `this`, `call()`, `apply()`, and `bind()` cannot change their `this`.

### Within event handlers

For a regular DOM event handler, `this` refers to the element receiving the event:

```js
document.getElementById('my-button').addEventListener(
  'click',
  function () {
    console.log(this);
  }
);
```

With an arrow callback, `this` is inherited from the surrounding scope instead:

```js
document.getElementById('my-button').addEventListener(
  'click',
  () => {
    console.log(this);
  }
);
```

This is why choosing between a regular function and an arrow function matters when event-handler `this` is required.

## ⭐ Interview Takeaway

If the interviewer gives you a `this` question, don't start by asking “what object does this belong to?”

Ask:

> **“How is this function being invoked?”**

Then apply the appropriate rule.

This concept is a direct foundation for the next coding problems involving `call`, `apply`, `bind`, constructors, and OOP.
