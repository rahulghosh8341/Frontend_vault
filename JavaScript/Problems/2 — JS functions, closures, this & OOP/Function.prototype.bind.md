---
title: Implement `Function.prototype.bind` without calling the native `bind` method
aliases:
  - Function.prototype.bind
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
  - "[[Rippling]]"
  - "[[Atlassian]]"
  - "[[ByteDance]]"
  - "[[Adobe]]"
pattern:
  - "[[Closure]]"
concepts:
  - "[[this]]"
  - "[[call, apply and bind]]"
  - "[[Closure]]"
  - "[[Function.prototype.apply]]"
  - "[[Function.prototype.call]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-28
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement `Function.prototype.bind` which creates a new function that, when called, has its `this` keyword set to a provided value, with preceding arguments.

## Problem

The `Function.prototype.bind()` method creates a new function that, when called, has its `this` keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called.

Implement your own `Function.prototype.bind` without calling the native `bind` method. Implement the function as `Function.prototype.myBind`.

```js
const john = {
  age: 42,
  getAge: function () {
    return this.age;
  },
};

const unboundGetAge = john.getAge;
console.log(unboundGetAge()); // undefined

const boundGetAge = john.getAge.myBind(john);
console.log(boundGetAge()); // 42
```

## Companies

- [[Amazon]]
- [[Rippling]]
- [[Atlassian]]
- [[ByteDance]]
- [[Adobe]]

## Pattern

- [[Closure]]

## 🤔 Thought Process

The key to `bind` is storing a **binding record** in a closure:
1. When `myBind(thisArg, ...argArray)` is called:
   - Capture reference to the original function (`const originalFn = this`).
   - Capture `thisArg` (the receiver to bind).
   - Capture initial bound arguments (`argArray`).
2. Return a new wrapper function:
   - When the bound function is later invoked with new call-time arguments (`...callArgs`), combine them: `[...argArray, ...callArgs]`.
   - Invoke `originalFn` with `thisArg` and combined arguments using `apply` or `Reflect.apply`.

| Saved when `myBind` is called | Used when bound function is called |
|---|---|
| Original function (`this`) | Call it via `apply` / `Reflect.apply` |
| `thisArg` | Pass as the receiver |
| Bound arguments (`argArray`) | Prepend before new call-time arguments |

## 💻 Final Solution

### Approach 1 — Standard Polyfill (Using `apply` / `Reflect.apply`)

```js
/**
 * @param {any} thisArg
 * @param {...*} argArray
 * @return {Function}
 */
Function.prototype.myBind = function (thisArg, ...argArray) {
  const originalMethod = this;

  return function (...args) {
    return originalMethod.apply(thisArg, [...argArray, ...args]);
  };
};
```

### Approach 2 — Using `Reflect.apply` (Official GreatFrontend Solution)

```js
Function.prototype.myBind = function (thisArg, ...argArray) {
  const originalMethod = this;

  return function (...args) {
    return Reflect.apply(originalMethod, thisArg, [...argArray, ...args]);
  };
};
```

### Approach 3 — Without Native `apply`/`call` (Symbol Property Attachment)

```js
Function.prototype.myBind = function (thisArg, ...argArray) {
  const originalMethod = this;

  return function (...args) {
    const target = thisArg != null ? Object(thisArg) : globalThis;
    const sym = Symbol('fn');
    target[sym] = originalMethod;
    const result = target[sym](...argArray, ...args);
    delete target[sym];
    return result;
  };
};
```

### Approach 4 — Supporting Constructor Invocations (`new boundFn()`)

In standard ECMAScript specification, if a bound function is constructed with `new`, the bound `thisArg` is ignored and a new instance of the original constructor is created instead:

```js
Function.prototype.myBind = function (thisArg, ...argArray) {
  const originalMethod = this;

  function boundFunction(...args) {
    // If invoked with `new`, `this` is an instance of boundFunction
    const isNew = new.target !== undefined || this instanceof boundFunction;
    const receiver = isNew ? this : thisArg;

    return originalMethod.apply(receiver, [...argArray, ...args]);
  }

  // Preserve prototype chain for `instanceof` checks
  if (originalMethod.prototype) {
    boundFunction.prototype = Object.create(originalMethod.prototype);
  }

  return boundFunction;
};
```

## 🤔 Why This Works

- **Closure Retention**: The returned wrapper function closes over `originalMethod`, `thisArg`, and `argArray`. These variables remain in memory between `bind()` time and eventual execution time.
- **Partial Application**: Prepending `...argArray` before `...args` means arguments provided at bind-time take precedence and appear first in the parameter list.
- **`Reflect.apply(target, thisArgument, argumentsList)`**: Clean ES6 API to invoke a function with an explicit `this` and arguments array without directly touching `Function.prototype.apply`.

## 🐞 Bugs / Pitfalls

- **Using arrow function for `myBind`**: Arrow functions capture lexical `this`, so `this` inside `myBind` would not point to the caller function. Must use regular `function`.
- **Losing original function reference**: If you reference `this` inside the returned inner function without saving `const originalMethod = this;`, `this` will refer to the caller of the returned function, not the original target method.
- **Overwriting arguments**: Appending instead of prepending (`[...args, ...argArray]`) breaks parameter ordering. Bind arguments must precede call-time arguments.
- **Ignoring constructor calls (`new`)**: Advanced interview follow-up: in the real spec, `new boundFn()` overrides the bound `thisArg`.

## Production Considerations

- Use native `Function.prototype.bind()` in production code; it is engine-optimized and handles internal slots (`[[BoundTargetFunction]]`, `[[BoundThis]]`, `[[BoundArguments]]`).
- Bound functions do not have their own `prototype` property; their `.prototype` is `undefined`.
- Once bound, a function's `this` cannot be overridden by subsequent `.bind()`, `.call()`, or `.apply()`.

## ⭐ Revision Notes

### Key Facts

- `bind` returns a **new function**; it does not invoke the function immediately (unlike `call`/`apply`).
- Supports **currying / partial application**: arguments passed to `bind` precede arguments passed to the returned function.
- A function can only have its `this` bound once. Repeated `.bind()` calls only curry arguments; the first bound `this` permanently sticks.
- Native `bind` creates a exotic bound function object.

### Common Interview Questions

- What is the difference between `bind`, `call`, and `apply`?
  - `call`: invokes immediately with comma-separated arguments.
  - `apply`: invokes immediately with an array of arguments.
  - `bind`: returns a new function with fixed `this` and pre-filled arguments for later execution.
- Can you rebind a bound function with another `.bind()`?
  - No. The inner function will execute with its originally bound receiver; subsequent `bind` calls can only append arguments.
- How does `new` interact with bound functions?
  - When called with `new`, the bound `thisArg` is ignored. The new instance created by `new` is used as `this`.

### Interview Takeaways

- Simple 4-line implementation (`Reflect.apply(originalMethod, thisArg, [...argArray, ...args])`) satisfies 95% of interview questions.
- Mentioning `new.target` or `instanceof` handling demonstrates senior-level knowledge of the ECMAScript specification.

### Related

- [[this]]
- [[call, apply and bind]]
- [[Function.prototype.call]]
- [[Function.prototype.apply]]
- [[Currying]]
- [[Closure]]
