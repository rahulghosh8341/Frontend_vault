---
title: Implement `Function.prototype.apply` without calling the native `apply` method
aliases:
  - Function.prototype.apply
difficulty: Medium
time: 15 min
languages:
  - JavaScript
concepts:
  - "[[this]]"
  - "[[call, apply and bind]]"
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Implement `Function.prototype.apply` which calls a function with a given `this` value and an array/array-like object of arguments.

## Problem

Implement your own `Function.prototype.apply` without calling the native `Function.prototype.apply` method. To avoid overwriting the actual prototype method, implement it as `Function.prototype.myApply`.

```js
function multiplyAge(multiplier = 1) {
  return this.age * multiplier;
}

const mary = { age: 21 };
const john = { age: 42 };

multiplyAge.myApply(mary);        // 21
multiplyAge.myApply(john, [2]);   // 84
```

---

## 🤔 Thought Process

- Understand what `apply()` needs to control: the function's `this` value and its arguments.
- `this` inside `myApply` refers to the original function being called.
- `argArray` contains the arguments, so spread it with `...argArray`.
- Without using `call()`/`apply()`, temporarily attach the original function to `thisArg`.
- Invoke it as `thisArg.Tempfunc(...)` so JavaScript automatically sets `this` to `thisArg`.
- Store the return value, remove the temporary property, and return the result.

---

## 💻 Final Solution

```js
// Approach 1 — Using .call()
Function.prototype.myApply = function (thisArg, argArray) {
  return this.call(thisArg, ...argArray);
};

// Approach 2 — Without call/bind/apply (Attaching to thisArg)
Function.prototype.myApply = function (thisArg, argArray) {
  thisArg.Tempfunc = this;
  const result = thisArg.Tempfunc(...argArray);
  delete thisArg.Tempfunc;
  return result;
};

// Approach 3 — Robust / Collision-free (Handling Symbols & primitive thisArg)
Function.prototype.myApply = function (thisArg, argArray = []) {
  thisArg = thisArg != null ? Object(thisArg) : globalThis;
  const sym = Symbol('fn');
  thisArg[sym] = this;
  const result = thisArg[sym](...argArray);
  delete thisArg[sym];
  return result;
};
```

---

## 🤔 Why This Works

Calling a function as an object method:

```js
thisArg.Tempfunc(...argArray);
```

automatically makes:

```js
this === thisArg;
```

So temporarily attaching the original function to `thisArg` recreates the important behavior of `apply()` without using the native `call()` or `apply()` methods.

### Edge Cases & Details

- **Property Collision**: Using a hardcoded property name like `Tempfunc` could accidentally overwrite an existing property on `thisArg`. Using `Symbol('fn')` guarantees uniqueness.
- **Null / Undefined `thisArg`**: In non-strict mode, passing `null` or `undefined` defaults `this` to `globalThis` (or `window`).
- **Primitive `thisArg`**: Passing a primitive (e.g. `number` or `string`) requires wrapping via `Object(thisArg)` so properties can be assigned.
- **Missing `argArray`**: If `argArray` is omitted or `undefined`, default to `[]` or spread empty args.

---

## 🐞 Bugs I Made

Initially considered using `.call()`, but then understood how to invoke the function with the desired `this` without `call()`/`apply()`.

---

## Production Considerations

- In real-world code, use native `Function.prototype.apply()` or modern spread syntax `fn(...args)`.
- Polyfilling native function methods is useful for understanding prototype chaining, implicit vs explicit `this` binding, and `Symbol` usage in JS internals.

---

## ⭐ Revision Notes

### Key Facts

- `apply(thisArg, [argsArray])` accepts arguments as an **array/array-like object**, whereas `call(thisArg, arg1, arg2, ...)` accepts arguments individually.
- Method invocation (`obj.fn()`) establishes **implicit `this` binding**.
- `Symbol()` provides a collision-free property key for temporary object attachments.

### Common Interview Questions

- What is the main difference between `call` and `apply`? → `call` accepts comma-separated arguments; `apply` accepts an array of arguments.
- How can you set `this` context without `call`, `apply`, or `bind`? → Attach the function as a property on the target object and call it as a method (`obj.fn()`).
- Why use `Symbol()` when polyfilling `call`/`apply`? → To avoid mutating or overwriting pre-existing keys on `thisArg`.

### Interview Takeaways

- Remember to handle edge cases: optional `argArray`, `null`/`undefined` `thisArg`, and cleanup (`delete thisArg[sym]`).

### Related

- [[this]]
- [[call, apply and bind]]
