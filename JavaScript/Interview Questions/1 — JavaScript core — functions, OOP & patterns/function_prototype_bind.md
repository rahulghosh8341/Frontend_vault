---
id: function-prototype-bind
title: Explain `Function.prototype.bind` in JavaScript
type: quiz
section: "1 — JavaScript core — functions, OOP & patterns"
solved: true
tags:
  - javascript
  - interview
  - functions
  - this
  - bind
solvedDate: 2026-09-09
---

> [!info] Source
> GreatFrontEnd — **Explain `Function.prototype.bind` in JavaScript**

## TL;DR

`Function.prototype.bind` creates a **new function** with a specific `this` value and optionally pre-filled arguments.

Unlike `call()` and `apply()`, `bind()` **does not invoke the function immediately**.

```js
const boundFn = fn.bind(thisArg, arg1, arg2);

boundFn();
```

Main uses:

- Preserve a function's `this` context.
- Pass methods safely as callbacks.
- Partially apply arguments.
- Borrow methods from another object.

## Interview Answer (30–60 sec)

`Function.prototype.bind` creates a new function with a permanently bound `this` value. It can also preset some arguments, which is useful for partial application.

Unlike `call` and `apply`, `bind` doesn't execute the function immediately. Instead, it returns a new function that can be called later.

A common use case is passing an object method as a callback while preserving its `this` context. For example, `john.greet.bind(john)` ensures that `this` inside `greet` still refers to `john`.

## Key Takeaways

- `bind()` **returns a new function**.
- It does **not** call the original function immediately.
- The first argument specifies the bound `this`.
- Arguments after `thisArg` are pre-filled.
- Later arguments are appended when the bound function is called.
- Useful for callbacks and event handlers.
- Useful for partial application.
- Can be used for method borrowing.
- Arrow functions have lexical `this`, so binding cannot change their `this`.

## 🧠 Mental Model

```text
fn.bind(thisArg, presetArgs...)
              |
              ↓
       NEW FUNCTION
              |
              ↓
       called later
              |
              +--> this = thisArg
              +--> preset args + new args
```

Compare:

```text
call()  → invoke now
apply() → invoke now
bind()  → create function for later
```

## Common Interview Traps

### 1. Thinking `bind()` invokes the function

```js
const bound = fn.bind(obj);
```

Nothing has executed yet.

```js
bound();
```

This is where the function executes.

### 2. Confusing `bind()` with `call()`

```js
fn.call(obj);      // executes now

const bound = fn.bind(obj); // returns new function
bound();           // executes later
```

### 3. Forgetting partial application

```js
function multiply(a, b) {
  return a * b;
}

const multiplyBy5 = multiply.bind(null, 5);

multiplyBy5(3); // 15
```

The `5` is pre-filled.

### 4. Thinking `bind()` can change arrow-function `this`

Arrow functions don't have their own dynamic `this`, so `bind()` cannot replace their lexical `this`.

## Common Follow-ups

### What is the difference between `call`, `apply`, and `bind`?

- `call()` invokes immediately with individually supplied arguments.
- `apply()` invokes immediately with arguments supplied as an array/array-like value.
- `bind()` returns a new function with bound `this` and optional preset arguments.

### What is partial application?

Partial application creates a new function by pre-filling some arguments of an existing function.

### Why is `bind()` useful with callbacks?

Passing a method as a callback can detach it from its original object. `bind()` creates a function whose `this` remains associated with the intended object.

### Can `bind()` be used for method borrowing?

Yes. You can bind an object's method to another object and invoke the returned function with the second object as `this`.

## My Notes

- This directly follows the previous `this` and `call/apply` questions.
- **Most important distinction:** `call/apply` execute now; `bind` returns a function.
- This directly prepares for the GFE coding problem:
  - `Function.prototype.bind`
- Also useful for:
  - callbacks
  - constructors/classes
  - partial application
  - method borrowing
- Remember: **bind = fix `this` + optionally preset arguments + return a new function.**

# Detailed Reference

## TL;DR

`Function.prototype.bind` is a method in JavaScript that allows you to create a new function with a specific `this` value and optional initial arguments. Its primary purpose is to:
  * Binding `this` value to preserve context: The primary purpose of `bind` is to bind the `this` value of a function to a specific object. When you call `func.bind(thisArg)`, it creates a new function with the same body as `func`, but with `this` permanently bound to `thisArg`.
  * Partial application of arguments: `bind` also allows you to pre-specify arguments for the new function. Any arguments passed to `bind` after `thisArg` will be prepended to the arguments list when the new function is called.
  * Method borrowing: `bind` allows you to borrow methods from one object and apply them to another object, even if they were not originally designed to work with that object.
The `bind` method is particularly useful in scenarios where you need to ensure that a function is called with a specific `this` context, such as in event handlers, callbacks, or method borrowing.

* * *

## `Function.prototype.bind`

`Function.prototype.bind` allows you to create a new function with a specific `this` context and, optionally, preset arguments. `bind()` is most useful for preserving the value of `this` in methods of classes that you want to pass into other functions.

`bind` was frequently used on legacy React class component methods which were not defined using arrow functions.

```js
const john = {
  age: 42,
  getAge: function () {
    return this.age;
  },
};
console.log(john.getAge()); // 42

const unboundGetAge = john.getAge;
console.log(unboundGetAge()); // undefined

const boundGetAge = john.getAge.bind(john);
console.log(boundGetAge()); // 42

const mary = { age: 21 };
const boundGetAgeMary = john.getAge.bind(mary);
console.log(boundGetAgeMary()); // 21
```

In the example above, when the `getAge` method is called without a calling object (as `unboundGetAge`), the value is `undefined` because the value of `this` within `getAge()` becomes the global object. `boundGetAge()` has its `this` bound to `john`, hence it is able to obtain the `age` of `john`.

We can even use `getAge` on another object which is not `john`! `boundGetAgeMary` returns the `age` of `mary`.

## Use cases

Here are some common scenarios where `bind` is frequently used:

### Preserving context and fixing the `this` value in callbacks

When you pass a function as a callback, the `this` value inside the function can be unpredictable because it is determined by the execution context. Using `bind()` helps ensure that the correct `this` value is maintained.

```js
class Person {
  constructor(firstName) {
    this.firstName = firstName;
  }
  greet() {
    console.log(`Hello, my name is ${this.firstName}`);
  }
}

const john = new Person('John');
// Without bind(), `this` inside the callback will be the global object
setTimeout(john.greet, 1000); // Output: "Hello, my name is undefined"

// Using bind() to fix the `this` value
setTimeout(john.greet.bind(john), 2000); // Output: "Hello, my name is John"
```

You can also use arrow functions to define class methods for this purpose instead of using `bind`. Arrow functions have their `this` value bound to the lexical context.

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  greet = () => {
    console.log(`Hello, my name is ${this.name}`);
  };
}

const john = new Person('John Doe');
setTimeout(john.greet, 1000); // Output: "Hello, my name is John Doe"
```

### Partial application of functions (currying)

`bind` can be used to create a new function with some arguments pre-set. This is known as partial application or currying.

```js
function multiply(a, b) {
  return a * b;
}

// Using bind() to create a new function with some arguments pre-set
const multiplyBy5 = multiply.bind(null, 5);
console.log(multiplyBy5(3)); // Output: 15
```

### Method borrowing

`bind` allows you to borrow methods from one object and apply them to another object, even if they were not originally designed to work with that object. This can be handy when you need to reuse functionality across different objects.

```js
const person = {
  name: 'John',
  greet: function () {
    console.log(`Hello, ${this.name}!`);
  },
};

const greetPerson = person.greet.bind({ name: 'Alice' });
greetPerson(); // Output: Hello, Alice!
```

## Practice

Try implementing your own `Function.prototype.bind()` method on GreatFrontEnd.

## Further Reading

- Function.prototype.bind() - JavaScript | MDN
- Function Binding | javascript.info
