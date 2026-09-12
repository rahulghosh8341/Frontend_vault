---
id: call-vs-apply
title: What’s the difference between `.call` and `.apply` in JavaScript?
type: quiz
section: "1 — JavaScript core — functions, OOP & patterns"
solved: true
tags:
  - javascript
  - interview
  - functions
  - this
solvedDate: 2026-09-09
---

> [!info] Source
> GreatFrontEnd — **What’s the difference between `.call` and `.apply` in JavaScript?**

## TL;DR

`.call` and `.apply` are both used to invoke functions with a specific `this` context and arguments. The primary difference is how they accept arguments:

- `.call(thisArg, arg1, arg2, ...)` → arguments individually
- `.apply(thisArg, [argsArray])` → arguments as an array

```js
function add(a, b) {
  return a + b;
}

add.call(null, 1, 2);      // 3
add.apply(null, [1, 2]);   // 3
```

## Interview Answer (30–60 sec)

`.call` and `.apply` both invoke a function immediately and allow you to explicitly set its `this` value.

The main difference is how arguments are passed. `.call` accepts arguments individually, while `.apply` accepts them as an array or array-like value.

For example, `add.call(null, 1, 2)` and `add.apply(null, [1, 2])` produce the same result.

A common use case is function borrowing, where we invoke a function with another object as its `this` value. With modern JavaScript, `.call(...args)` is often used with spread syntax when the arguments are already in an array.

## Key Takeaways

- Both invoke the function **immediately**.
- Both can explicitly set `this`.
- `.call()` → comma-separated arguments.
- `.apply()` → array/array-like arguments.
- Easy memory trick:
  - **C**all → **C**omma-separated
  - **A**pply → **A**rray
- `.call(null, ...args)` can provide an array of arguments using spread syntax.
- `.apply()` can be used for function borrowing.
- `Array.prototype.push.apply(arr1, arr2)` is equivalent to `arr1.push(...arr2)`.
- `.call()` and `.apply()` differ from `.bind()`: `bind()` returns a new function instead of invoking it immediately.

## 🧠 Mental Model

```text
call / apply
     |
     +---- first argument ----> this
     |
     +---- remaining arguments
              |
              +-- call  → arg1, arg2, arg3
              |
              +-- apply → [arg1, arg2, arg3]
```

Think:

> **Same job, different argument format.**

## Common Interview Traps

### 1. Confusing `call` with `bind`

```js
fn.call(obj, 1, 2);
```

invokes `fn` immediately.

```js
const newFn = fn.bind(obj, 1, 2);
```

creates a new function that can be invoked later.

### 2. Forgetting that the first argument is `this`

```js
greet.call(person);
```

The `person` object becomes the function's `this`.

### 3. Mixing up argument syntax

```js
fn.call(obj, 1, 2, 3);

fn.apply(obj, [1, 2, 3]);
```

### 4. Thinking `.apply()` is always preferable for arrays

Modern JavaScript can use spread syntax:

```js
fn.call(obj, ...args);
```

So `.apply()` is less necessary when you already have an array of arguments.

## Common Follow-ups

### How do `.call` and `.apply` differ from `Function.prototype.bind`?

`.call` and `.apply` invoke the function immediately. `bind` returns a new function with a bound `this` value and optionally bound arguments.

### When would you use `.apply()`?

One use case is when the arguments are already stored in an array or array-like value.

### What is function borrowing?

Function borrowing means using a function defined for one object with another object by explicitly supplying the desired `this` value using `.call()` or `.apply()`.

### What is the modern alternative to `.apply()` when arguments are in an array?

Use `.call()` with the spread operator:

```js
fn.call(obj, ...args);
```

## My Notes

- This is directly connected to the previous **`this`** question.
- The key difference is **argument passing**, not `this`.
- Both `.call()` and `.apply()` explicitly set `this`.
- This question directly prepares for the coding problems:
  - `Function.prototype.apply`
  - `Function.prototype.call`
- Next important concept after this is **`bind()`**.

# Detailed Reference

## TL;DR

`.call` and `.apply` are both used to invoke functions with a specific `this` context and arguments. The primary difference lies in how they accept arguments:

- `.call(thisArg, arg1, arg2, ...)`: Takes arguments individually.
- `.apply(thisArg, [argsArray])`: Takes arguments as an array.

Assuming we have a function `add`, the function can be invoked using `.call` and `.apply` in the following manner:

```js
function add(a, b) {
  return a + b;
}

console.log(add.call(null, 1, 2)); // 3
console.log(add.apply(null, [1, 2])); // 3
```

---

## Call vs Apply

Both `.call` and `.apply` are used to invoke functions, and the first parameter will be used as the value of `this` within the function. However, `.call` takes in comma-separated arguments as the next arguments, while `.apply` takes in an array of arguments as the next argument.

An easy way to remember this is C for `call` and comma-separated and A for `apply` and an array of arguments.

```js
function add(a, b) {
  return a + b;
}

console.log(add.call(null, 1, 2)); // 3
console.log(add.apply(null, [1, 2])); // 3
```

With ES6 syntax, we can invoke `call` using an array along with the spread operator for the arguments.

```js
function add(a, b) {
  return a + b;
}

console.log(add.call(null, ...[1, 2])); // 3
```

## Use cases

### Context management

`.call` and `.apply` can set the `this` context explicitly when invoking methods on different objects.

```js
const person = {
  name: 'John',
  greet() {
    console.log(`Hello, my name is ${this.name}`);
  },
};

const anotherPerson = { name: 'Alice' };

person.greet.call(anotherPerson); // Hello, my name is Alice
person.greet.apply(anotherPerson); // Hello, my name is Alice
```

### Function borrowing

Both `.call` and `.apply` allow borrowing methods from one object and using them in the context of another. This is useful when passing functions as arguments (callbacks) and the original `this` context is lost. `.call` and `.apply` allow the function to be invoked with the intended `this` value.

```js
function greet() {
  console.log(`Hello, my name is ${this.name}`);
}

const person1 = { name: 'John' };
const person2 = { name: 'Alice' };

greet.call(person1); // Hello, my name is John
greet.apply(person2); // Hello, my name is Alice
```

### Alternative syntax to call methods on objects

`.apply` can be used with object methods by passing the object as the first argument followed by the usual parameters.

```js
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

Array.prototype.push.apply(arr1, arr2); // Same as arr1.push(4, 5, 6)

console.log(arr1); // [1, 2, 3, 4, 5, 6]
```

Deconstructing the above:

1. The first object, `arr1` will be used as the `this` value.
2. `.push()` is called on `arr1` using `arr2` as an array of arguments because it's using `.apply()`.
3. `Array.prototype.push.apply(arr1, arr2)` is equivalent to `arr1.push(...arr2)`.

It may not be obvious, but `Array.prototype.push.apply(arr1, arr2)` mutates `arr1`. It's clearer to call methods using the OOP-centric way instead where possible.

## Follow-up questions

- How do `.call` and `.apply` differ from `Function.prototype.bind`?

## Practice

Practice implementing your own `Function.prototype.call` method and `Function.prototype.apply` method on GreatFrontEnd.

## Further reading

- Function.prototype.call | MDN
- Function.prototype.apply | MDN
