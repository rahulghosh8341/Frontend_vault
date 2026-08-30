---
title: Array.prototype.square()
aliases:
  - Array Square
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Higher Order Mapping]]"
concepts:
  - "[[Prototypes]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Implement `Array.prototype.square()` to return a new array with squared elements.

## Problem

```javascript
[-2].square(); // [4]
[1, 2, 3, 4].square(); // [1, 4, 9, 16]
```

## Companies

- None specified.

## Pattern

- [[Higher Order Mapping]]

## 🤔 Thought Process

User used `this.map` to iterate and return new array. Direct, clean, preserves immutability.

## 💻 Final Solution

```javascript
/**
 * @return {Array<number>}
 */
Array.prototype.square = function () {
  return this.map(i => i * i);
};
```

## 🤔 Why This Works

- `this` refers to array instance.
- `map` handles creation of new array and element-wise transformation.
- Squaring `i * i` works for all numbers.

## 🐞 Bugs I Made

None.

## Production Considerations

- `Array.prototype.map` is standard and performs well.
- Using `** 2` or `Math.pow(i, 2)` also valid but `i * i` usually fastest.

## ⭐ Revision Notes

### Key Facts

- `this` context inside prototype methods points to calling object.
- Arrow functions should not be used for prototype methods if `this` access needed (they inherit lexical `this`).

### Common Interview Questions

- Why not use arrow function? → Arrow functions don't have own `this`.
- Is original array modified? → No, `map` returns new array.

### Interview Takeaways

- Prototype extension is common polyfill/utility pattern task.

### Related

- [[Higher Order Mapping]]
- [[Prototypes]]