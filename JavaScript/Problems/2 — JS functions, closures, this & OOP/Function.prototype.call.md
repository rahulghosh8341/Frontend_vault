---
title: Implement `Function.prototype.call` without calling the native `call` method
aliases:
  - Function.prototype.call
difficulty: Easy
time: 10 min
languages:
  - JavaScript
concepts:
  - "[[this]]"
  - "[[call, apply and bind]]"
  - "[[Function.prototype.apply]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-18
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Implement `Function.prototype.call` which calls a function with a given `this` value and individually provided arguments.

## Problem

Implement your own `Function.prototype.call` without calling the native `Function.prototype.call` method. To avoid overwriting the actual prototype method, implement it as `Function.prototype.myCall`.

```js
function multiplyAge(multiplier = 1) {
  return this.age * multiplier;
}

const mary = { age: 21 };
const john = { age: 42 };

multiplyAge.myCall(mary);       // 21
multiplyAge.myCall(john, 2);    // 84
```

---

## 🤔 Thought Process

- `call()` needs to invoke the original function with an explicit `this` context and individually passed arguments.
- `this` inside `myCall` refers to the function being called.
- `argArray` collects all subsequent parameters with rest syntax `...argArray`.
- Without using native `call()`, temporarily attach the function as a property on `thisArg`.
- Calling `thisArg.Tempfunc(...)` establishes implicit `this` binding where `this === thisArg`.
- Execute, store return value, delete the temporary property, and return result.

---

## 💻 Final Solution

### Approach 1 — User Solution (Attaching to `thisArg`)

```js
/**
 * @param {any} thisArg
 * @param {...*} argArray
 * @return {any}
 */
Function.prototype.myCall = function (thisArg, ...argArray) {

  if (thisArg == null) {
    thisArg = Object(thisArg);
  }
  
  thisArg.Tempfunc = this;
  const result = thisArg.Tempfunc(...argArray);
  delete thisArg.Tempfunc;
  return result;
};
```

### Approach 2 — Collision-Free Using `Symbol()`

```js
Function.prototype.myCall = function (thisArg, ...argArray) {
  // Handle null/undefined -> globalThis in non-strict mode; wrap primitives in Object
  thisArg = thisArg != null ? Object(thisArg) : globalThis;

  const sym = Symbol('fn');
  thisArg[sym] = this;
  const result = thisArg[sym](...argArray);
  delete thisArg[sym];
  return result;
};
```

### Approach 3 — Using `.bind()`

```js
Function.prototype.myCall = function (thisArg, ...argArray) {
  return this.bind(thisArg)(...argArray);
};
```

---

## 🤔 Why This Works

Calling a function as an object method:

```js
thisArg[sym](...argArray);
```

automatically makes:

```js
this === thisArg;
```

Temporarily attaching the target function to `thisArg` recreates explicit `this` binding behavior using JavaScript's native member access invocation rule.

### Edge Cases & Details

- **Property Collision**: Using a static property name like `Tempfunc` can overwrite an existing key on `thisArg`. `Symbol('fn')` guarantees a collision-free key.
- **Null / Undefined `thisArg`**: In non-strict mode, `call(null)` or `call(undefined)` binds to `globalThis` (or `window`).
- **Primitive `thisArg`**: If a number, string, or boolean is passed as `thisArg`, `Object(thisArg)` wraps it into its object wrapper (`Number`, `String`, `Boolean`) so properties can be attached.
- **Empty arguments**: Rest syntax `...argArray` captures an empty array `[]` when no extra arguments are passed; spreading `...[]` invokes with zero arguments cleanly.

---

## 🐞 Bugs I Made

None. Handled `thisArg == null` coercion and argument spreading properly.

---

## Production Considerations

- In modern JavaScript, use native `fn.call(thisArg, ...args)` or `Reflect.apply(fn, thisArg, args)`.
- Polyfilling `call` tests deep understanding of implicit vs explicit `this` binding, `Symbol` uniqueness, and boxing of primitives.

---

## ⭐ Revision Notes

### Key Facts

- `call(thisArg, ...args)` takes comma-separated arguments, while `apply(thisArg, [args])` takes an array.
- Method invocation (`obj.fn()`) establishes **implicit `this` binding**.
- `Symbol()` guarantees property key uniqueness, preventing mutation of existing object properties.
- `this.bind(thisArg)(...argArray)` binds context and immediately executes.

### Common Interview Questions

- What is the difference between `call` and `apply`? `call` takes parameters individually; `apply` takes parameters as an array.
- How can you invoke a function with a custom `this` without `call`, `apply`, or `bind`? Attach the function temporarily to the target object and call it as a method (`obj[sym](...)`).
- Why use `Symbol`? Prevents overwriting pre-existing keys on `thisArg`.
- What does non-strict mode do when `thisArg` is `null` or `undefined`? Binds to `globalThis` (window in browser, global in Node.js).

### Interview Takeaways

- Core polyfill recipe: wrap `Object(thisArg)` -> attach via `Symbol()` -> invoke with spread -> `delete` symbol -> return result.
- Alternative one-liner using `bind`: `return this.bind(thisArg)(...argArray);`.

### Related

- [[this]]
- [[call, apply and bind]]
- [[Function.prototype.apply]]
- [[Function.prototype.bind]]
