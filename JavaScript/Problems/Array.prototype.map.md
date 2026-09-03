---
title: Implement Array.prototype.myMap
aliases:
  - Array.prototype.map
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
  - "[[Apple]]"
  - "[[ByteDance]]"
  - "[[Pinterest]]"
  - "[[Canva]]"
pattern:
  - "[[Higher Order Mapping]]"
concepts:
  - "[[this]]"
  - "[[Function.prototype.call]]"
  - "[[Sparse Arrays]]"
  - "[[Higher Order Functions]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement `Array.prototype.myMap` that creates a new array populated with results of calling a provided function on every element.

## Problem

`Array.prototype.map` creates a new array populated with the results of calling a provided function on every element in the calling array. Implement as `Array.prototype.myMap`.

```js
[1, 2, 3, 4].myMap((i) => i);     // [1, 2, 3, 4]
[1, 2, 3, 4].myMap((i) => i * i); // [1, 4, 9, 16]
```

The callback receives `(element, index, array)`. `map` also accepts a `thisArg` second parameter. Preserve sparse array holes.

## Companies

- [[Amazon]]
- [[Apple]]
- [[ByteDance]]
- [[Pinterest]]
- [[Canva]]

## Pattern

- [[Higher Order Mapping]]

## 🤔 Thought Process

1. Create new array with same length as original.
2. Loop through every index.
3. Check whether index actually exists using `i in this`.
4. If it exists, call callback with `(element, index, original array)`.
5. Store result at same index.
6. Return new array.

## 💻 Final Solution

```js
Array.prototype.myMap = function (callbackFn, thisArg) {
  let result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this)
      result[i] = callbackFn.call(thisArg, this[i], i, this);
  }
  return result;
};
```

## 🤔 Why This Works

### `this` Binding

Inside `Array.prototype.myMap`, `this` is the calling array. `[1, 2, 3].myMap(...)` → `this === [1, 2, 3]`.

### Callback Arguments

Callback receives three arguments: `callbackFn(element, index, array)`. Invoke with `callbackFn.call(thisArg, this[i], i, this)`.

### `thisArg`

`thisArg` becomes `this` inside a regular function callback. Arrow functions ignore `thisArg`.

### Sparse Arrays

Given `const arr = [1, 2, , 4]`, index `2` is a hole, not `undefined`. Native `map()` preserves the hole. `if (i in this)` checks whether the index exists.

### Why `push()` Is Wrong

`push()` collapses holes. `[1, 2, , 4]` with `push` produces `[1, 2, 4]` — index `4` moves from index `3` to `2`. Use `result[i] = ...` to preserve original index.

### `i in this` vs `this[i] !== undefined`

An array can contain actual `undefined`: `[1, undefined, 3]` — `1 in arr` is `true`. A hole `[1, , 3]` — `1 in arr` is `false`. These are different.

## 🐞 Bugs I Made

None.

## Production Considerations

- Native `Array.prototype.map` available in all modern JS.
- Time: **O(n)**, Space: **O(n)**.
- Prefer native `map` over polyfill for performance.

## ⭐ Revision Notes

### Key Facts

- `map` preserves array length and holes — create `new Array(this.length)` and assign by index, don't use `push`.
- Callback signature: `(element, index, array)`.
- `thisArg` sets `this` for callback (regular functions only).
- `i in this` checks property existence, not value.

### Mental Model

```
Original array
     ↓
new Array(same length)
     ↓
for each index
     ↓
index exists?
   ↙       ↘
 yes       no
  ↓         ↓
callback   keep hole
  ↓
result[i] = ...
     ↓
return result
```

### Common Interview Questions

- What arguments does the map callback receive? → `(element, index, array)`.
- How do you handle sparse arrays in a map polyfill? → `i in this` check, assign by index.
- What's the difference between `push` and indexed assignment for polyfills? → `push` collapses holes.

### Interview Takeaways

- **Interview keywords:** prototype method · `this` · `thisArg` · `Function.prototype.call` · callback arguments · sparse arrays · holes · indexed assignment · `i in array`.

### Related

- [[Higher Order Mapping]]
- [[Array.prototype.filter]]
- [[Array.prototype.reduce]]
