---
title: Implement a function uniqueArray that takes in an array and returns a duplicate-free version where only the first occurrence of each element is kept
aliases:
  - Unique Array
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[Set Lookup]]"
concepts:
  - "[[Set]]"
  - "[[Array Methods]]"
solved: true
solvedDate: 2026-09-01
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Remove duplicates from an array while preserving the first‑occurrence order, using SameValueZero equality.

## Problem

Implement a function `uniqueArray(array)` that returns a duplicate‑free version of the input array. Only the first occurrence of each element is kept, and the order of the result follows the order in which elements first appear in the input.

```js
uniqueArray([1, 2, 3]); // => [1, 2, 3]
uniqueArray([1, 1, 2]); // => [1, 2]
uniqueArray([2, 1, 2]); // => [2, 1]
```

**Constraints:** Uniqueness follows SameValueZero equality: `NaN` values are considered equal, and object equality is reference‑based (two distinct objects with the same contents are not equal). Do not mutate the original array.

## Companies

- None

## Pattern

- [[Set Lookup]]

## 🤔 Thought Process

1. Convert the array into a `Set` — this automatically removes duplicates because a `Set` stores only unique values.
2. Spread the `Set` back into an array (`[...new Set(array)]`) or use `Array.from`. Spreading a `Set` yields values in insertion order, which matches the order of first appearance in the original array.
3. Return the new array.

## 💻 Final Solution

```js
export default function uniqueArray(array) {
  return Array.from(new Set(array));
}
```

## 🤔 Why This Works

- `new Set(array)` iterates over `array` and adds each value to the set; duplicate values are ignored because a set cannot contain the same value twice.
- The set preserves insertion order, so when we convert it back to an array we get the elements in the order of their first occurrence.
- `Array.from` (or the spread operator) creates a fresh array, leaving the input untouched.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Lodash offers `_.uniq` with identical semantics.
- Complexity: **O(n)** time and **O(n)** space.
- Set uses SameValueZero, so `NaN` matches `NaN` but two different objects with identical contents are not considered equal.

## ⭐ Revision Notes

### Key Facts

- `[...new Set(arr)]` is the canonical one‑liner for ordered deduplication in modern JavaScript.
- `Set.has()` uses SameValueZero, which treats `+0` and `-0` as equal and considers `NaN` equal to `NaN`.

### Common Interview Questions

- Why not use `filter` with `indexOf`? That would be O(n²) because `indexOf` scans the array for each element.
- How would you handle objects if you wanted structural equality? You would need a custom key‑extraction function or a `Map` keyed by a serialized representation.

### Interview Takeaways

- For ordered deduplication, a `Set` is the ideal tool: O(1) average insertion/lookup and order preservation.

### Related

- [[Set Lookup]]
- [[Intersection]]
- [[Difference]]
- [[From Pairs]]
