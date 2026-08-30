---
title: Implement a function mean(array) that returns the mean (also known as the average) of the values in array, an array of numbers.
aliases:
  - Mean
difficulty: Easy
time: 5 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Reduce]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 5 min
> Implement a function `mean(array)` that returns the arithmetic mean of an array of numbers.

## Problem

Implement a function `mean(array)` that returns the mean (also known as the average) of the values in `array`, an array of numbers.

```js
mean([4, 2, 8, 6]); // => 5
mean([1, 2, 3, 4]); // => 2.5
mean([1, 2, 2]); // => 1.6666666666666667
mean([]); // => NaN
```

## Companies

- [[Lodash]]

## Pattern

- [[Array Traversal]]

## 🤔 Thought Process

- Check if the array is empty (`array.length === 0`). If so, return `NaN` to match Lodash behavior and avoid division by zero.
- Sum all elements in the array and divide by `array.length`.
- Can be implemented via a `for...of` loop or `Array.prototype.reduce()`.

## 💻 Final Solution

```js
export default function mean(array) {
  let sum = 0;
  if (array.length === 0)
    return NaN;
  for (let num of array) {
    sum += num;
  }
  return sum / array.length;
}
```

Alternative concise version using `reduce`:

```js
export default function mean(array) {
  if (array.length === 0) return NaN;
  return array.reduce((a, b) => a + b, 0) / array.length;
}
```

## 🤔 Why This Works

- Checking `array.length === 0` handles the empty array case explicitly, returning `NaN` instead of `0 / 0` (`NaN`) or throwing.
- Summing values and dividing by length calculates the standard arithmetic mean.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Handle non-array inputs or edge cases if strict type checking is required.
- Native equivalent: Lodash `_.mean`.

## ⭐ Revision Notes

### Key Facts

- Empty array returns `NaN`.
- Mean equals sum of elements divided by array length.

### Common Interview Questions

- What does `mean([])` return? → `NaN`.
- How to handle empty arrays without `if` checks? → Using conditional checks or guards.

### Interview Takeaways

- Always check edge cases like empty inputs.

### Related

- [[Array Traversal]]
- [[Sum]]
