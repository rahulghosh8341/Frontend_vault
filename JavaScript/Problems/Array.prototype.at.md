---
title: Implement Array.prototype.myAt
aliases:
  - Array.prototype.at
difficulty: Easy
time: 15 min
languages:
  - JavaScript
pattern:
  - "[[Index Normalization]]"
concepts:
  - "[[this]]"
  - "[[Negative Indices]]"
  - "[[Bounds Checking]]"
solved: true
solvedDate: 2026-09-01
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement `Array.prototype.myAt` to return item at given index, supporting negative integers that count from end of array.

## Problem

`Array.prototype.at` takes an integer value and returns the item at that index, allowing for positive and negative integers. Negative integers count back from the last item in the array. Implement as `Array.prototype.myAt`.

```js
const arr = [42, 79];
arr.myAt(0); // 42
arr.myAt(1); // 79
arr.myAt(2); // undefined

arr.myAt(-1); // 79
arr.myAt(-2); // 42
arr.myAt(-3); // undefined
```

## Pattern

- [[Index Normalization]]

## 🤔 Thought Process

1. `this` refers to the array on which `myAt()` is called.
2. Get the array length.
3. If the index is negative, convert it to a normal positive index.
4. Check whether the resulting index is valid.
5. Return the element; otherwise return `undefined`.

## 💻 Final Solution

```js
Array.prototype.myAt = function (index) {
  let result;
  let arrLength = this.length;
  if (index < 0) {
    index = arrLength + index;
  }
  if (index >= 0 && index < arrLength) {
    return this[index];
  }
};
```

## 🤔 Why This Works

### `this` Binding

When `arr.myAt(1)` is called, inside `Array.prototype.myAt = function (index) { ... }`, `this` is `[42, 79]`. So `this.length` and `this[index]` operate on the calling array.

### Negative Index Conversion

Negative indices count backward from the end. For `[42, 79]`:

```
normal index:   0    1
negative:      -2   -1
```

Convert using `index = this.length + index`:

```
length = 2
-1 → 2 + (-1) = 1
-2 → 2 + (-2) = 0
-3 → 2 + (-3) = -1 → invalid
```

### Bounds Check

A valid array index must satisfy `index >= 0 && index < this.length`. Both conditions must be true, so use `&&`, not `||`.

## 🐞 Bugs I Made

### 1. Wrong negative-index formula

Had `index = arrLength - index`. For `-1`: `2 - (-1) = 3` — incorrect. Correct: `index = arrLength + index`.

### 2. Used `||` for bounds

Had `index > 0 || index < arrLength`. This allows invalid indices because only one condition needs to be true. Correct: `index >= 0 && index < arrLength`.

### 3. `index > 0` excludes index `0`

Array index `0` is valid (`arr.myAt(0)`). Use `>=` not `>`.

## Production Considerations

- Native `Array.prototype.at` is available in modern JS (ES2022). Use it directly in production.
- Time: **O(1)**, Space: **O(1)**.

## ⭐ Revision Notes

### Key Facts

- Negative array indices are converted by adding the negative index to the array length.
- Resulting index must satisfy `0 <= index < length`.
- `this` inside prototype method = the calling array.

### Mental Model

```
myAt(-1)
   ↓
length + (-1)
   ↓
last index
   ↓
return this[index]
```

### Common Interview Questions

- What does `this` refer to inside a prototype method? → The calling array.
- How do you convert a negative index to a positive one? → `length + index`.
- What happens for out-of-bounds negative indices? → `undefined`.

### Interview Takeaways

- **Interview keywords:** `this` · prototype method · negative indexing · zero-based indexing · bounds checking · `&&` vs `||`.

### Related

- [[Index Normalization]]
- [[Fill]]
