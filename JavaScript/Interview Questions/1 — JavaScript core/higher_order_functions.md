---
title: Higher-Order Functions
aliases:
  - Higher-Order Functions
  - HOF
tags:
  - javascript
  - interview
  - functions
  - higher-order-functions
section: "1 — JavaScript core"
solved: true
solvedDate: 2026-09-09
type: quiz
---

> [!info] Question
> What is the definition of a higher-order function in JavaScript?

## TL;DR

A **higher-order function (HOF)** is a function that **takes another function as an argument, returns a function, or both**.

```js
function execute(fn) {
  return fn();
}

function multiplier(factor) {
  return function (number) {
    return number * factor;
  };
}
```

Common examples include `map`, `filter`, `reduce`, `memoize`, `curry`, and `bind`.

## Interview Answer (30–60 sec)

> A higher-order function is a function that either accepts another function as an argument or returns a function as its result. For example, `map`, `filter`, and `reduce` are higher-order functions because they accept callback functions. Functions such as `memoize`, `curry`, and `bind` are also higher-order functions because they take or return functions. Higher-order functions are useful for abstraction, composition, and creating reusable behavior.

## Key Takeaways

- JavaScript functions are **first-class values**.
- A HOF can accept a function, return a function, or do both.
- A **callback** is a function passed to another function.
- A callback and a HOF are related, but they are **not the same thing**.
- `map`, `filter`, `reduce`, and `forEach` are common HOFs.
- `memoize`, `curry`, `once`, and `bind` are also HOF patterns.
- HOFs are often combined with **closures**.

## 🧠 Mental Model

```text
                 FUNCTION
                    │
         ┌──────────┴──────────┐
         │                     │
   receives function       returns function
         │                     │
         ↓                     ↓
       map()                 curry()
       filter()              memoize()
       reduce()              once()
       forEach()             bind()
         │                     │
         └──────────┬──────────┘
                    ↓
            HIGHER-ORDER
             FUNCTION
```

Ask:

> **Does this function receive another function or return a function?**

If yes, it is a higher-order function.

## Common Interview Traps

### ❌ A callback is the same as a higher-order function

Not exactly.

```js
numbers.map(x => x * 2);
```

Here:

```text
map → higher-order function
x => x * 2 → callback
```

### ❌ Only `map`, `filter`, and `reduce` are HOFs

No. Any function that accepts or returns another function qualifies.

```js
function createGreeter(greeting) {
  return function (name) {
    return `${greeting}, ${name}`;
  };
}
```

`createGreeter` is a HOF even though it has nothing to do with arrays.

### ❌ A function must both take and return a function

No. Either one is enough.

## Common Follow-ups

- What is a callback function?
- What is the difference between a callback and a higher-order function?
- Why are JavaScript functions called first-class values?
- What are common examples of higher-order functions?
- How are closures related to higher-order functions?
- What is function composition?
- How are `map`, `filter`, and `reduce` higher-order functions?
- Why are `curry`, `memoize`, and `bind` higher-order functions?

## My Notes

- 

# Detailed Reference

## Definition

A higher-order function is a function that **takes another function as an argument OR returns a function as its result**.

There are two main patterns.

### Pattern 1 — Takes a function as an argument

```js
function greet(name) {
  return `Hello ${name}`;
}

function execute(greeter, name) {
  return greeter(name);
}

execute(greet, "Rahul");
```

Here:

```text
execute = higher-order function
greet   = callback function
```

`execute` is higher-order because it receives a function.

## `map()` is a classic example

```js
const numbers = [1, 2, 3];

const result = numbers.map(number => number * 2);
```

Conceptually:

```text
numbers.map(...)
       ↓
map receives a function
       ↓
map calls that function for each item
```

Therefore:

```text
map = higher-order function
number => number * 2 = callback
```

The same concept applies to:

```js
filter()
forEach()
reduce()
```

## Pattern 2 — Returns a function

```js
function multiplier(factor) {
  return function (number) {
    return number * factor;
  };
}

const double = multiplier(2);

double(5); // 10
```

The returned function demonstrates a **closure** because it remembers `factor`.

```text
Higher-order function
        ↓
returns function
        ↓
closure
        ↓
remembers outer variables
```

## Higher-order functions are not only about arrays

Returning a function also qualifies:

```js
function createGreeter(greeting) {
  return function (name) {
    return `${greeting}, ${name}`;
  };
}
```

`createGreeter` is a HOF because it returns a function.

## Callback vs Higher-Order Function

```js
function process(numbers, callback) {
  return numbers.map(callback);
}
```

Here:

```text
process  → higher-order function
callback → function passed to process
```

A useful distinction:

> A callback is a function passed to another function.

> A higher-order function is a function that takes functions and/or returns functions.

## Decorator / Wrapper Pattern

A common HOF pattern is wrapping a function to add behavior.

```js
function withLogging(fn) {
  return function (...args) {
    console.log("Calling function");
    return fn(...args);
  };
}
```

Then:

```js
const loggedAdd = withLogging(add);
```

Calling `loggedAdd(2, 3)` conceptually:

```text
loggedAdd()
    ↓
log
    ↓
add(2, 3)
    ↓
return result
```

This pattern is useful for logging, caching, authentication wrappers, performance measurement, retry logic, middleware, and React higher-order components.

When a wrapper needs to preserve the original function's `this`, `apply` can be useful:

```js
function withLogging(fn) {
  return function (...args) {
    console.log("Calling function");
    return fn.apply(this, args);
  };
}
```

## Connection to the Next Coding Section

Higher-order functions are a foundation for:

```text
Higher-order functions
        │
        ├── Compose
        ├── Memoize
        ├── Curry
        ├── Once
        ├── Make Counter
        └── Function.prototype.bind
```

For example:

```js
function memoize(fn) {
  return function (...args) {
    // cache logic
  };
}
```

`memoize` takes a function and returns a function, so it is a HOF.

Likewise:

```js
function curry(fn) {
  return function (...args) {
    // collect arguments
  };
}
```

Again, it takes a function and returns a function.

## Quick Recognition Test

### Not a HOF

```js
function add(a, b) {
  return a + b;
}
```

It doesn't receive or return a function.

### HOF — receives a function

```js
function execute(fn) {
  return fn(10);
}
```

### HOF — returns a function

```js
function createAdder(x) {
  return function (y) {
    return x + y;
  };
}
```

### HOF + callback

```js
[1, 2, 3].map(x => x * 2);
```

```text
map → HOF
x => x * 2 → callback
```
