---
title: Implement a function findLastIndex(array, predicate, [fromIndex=array.length-1]) that takes an array of values, a function predicate, and an optional fromIndex number argument. findLastIndex iterates over elements of the array from right to left.
aliases:
  - Find Last Index
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Index Normalization]]"
concepts:
  - "[[Array.prototype.findLastIndex]]"
  - "[[Predicate]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Return index of last element where predicate true, searching right-to-left

## Problem

`findLastIndex(array, predicate, [fromIndex=array.length-1])` scans `array` backwards from `fromIndex` (inclusive), calling `predicate(value, index, array)` on each element. Returns first index satisfying predicate during backward iteration, or `-1`.

Edge case handling differs from forward `findIndex`:
- Negative: count from end (`-3` on length 5 → index 2)
- Negative out-of-bounds (`fromIndex < -array.length`) → start at index `0` (still search)
- Positive out-of-bounds (`fromIndex >= array.length`) → start at `array.length - 1` (still search)

Note asymmetry: forward version returns `-1` for positive out-of-bounds; backward version clamps to last index.

```javascript
const arr = [5, 4, 3, 2, 1]; // indices 0..4

findLastIndex(arr, (num) => num > 3);       // => 1 (from end)
findLastIndex(arr, (num) => num > 1, 3);    // => 3
findLastIndex(arr, (num) => num < 1, 3);    // => -1

findLastIndex(arr, (num) => num > 2, -3);   // => 2
findLastIndex(arr, (num) => num % 2 === 0, -3); // => 1
findLastIndex(arr, (num) => num > 0, 10);   // => 4
findLastIndex(arr, (num) => num > 0, -10);  // => 0
```

## Companies

None

## Pattern

- [[Index Normalization]]

## 🤔 Thought Process

Same translate-then-clamp normalization as [[Find Index]], but clamping differs per direction:
- Negative: `Math.max(fromIndex + array.length, 0)` — clamps to 0 on very negative
- Positive: `Math.min(fromIndex, array.length - 1)` — clamps to last index (not length, since start is inclusive here)
Then backward `for` loop from `start` down to `0`, return first match, `-1` after loop.

## 💻 Final Solution

```javascript
/**
 * @template T
 * @param {Array<T>} array
 * @param {(value: T, index: number, array: Array<T>) => boolean} predicate
 * @param {number} [fromIndex=array.length - 1]
 * @returns {number}
 */
export default function findLastIndex(
  array,
  predicate,
  fromIndex = array.length - 1,
) {
  let start;

  start = (fromIndex < 0) ? Math.max(fromIndex + array.length, 0) : Math.min(fromIndex, array.length - 1);

  for (let i = start; i >= 0; i--) {
    if (predicate(array[i], i, array)) {
      return i;
    }
  }
  return -1;
}
```

## 🤔 Why This Works

- Negative `-3` on length 5 → `5 + (-3) = 2`. Correct.
- Very negative `-10 + 5 = -5` → `max(-5, 0) = 0` — search still runs from index 0, matching spec.
- Positive out-of-bounds `10` → `min(10, 4) = 4` — starts from last index, matching spec.
- Backward loop `i >= 0` terminates after index 0; returns `-1` if no match.
- Predicate receives all three args per spec.

## 🐞 Bugs I Made

None.

## Production Considerations

Native `Array.prototype.findLastIndex(predicate, fromIndex)` (ES2023) implements this spec, including the clamping asymmetry — positive out-of-bounds clamps to `length - 1` unlike forward `findIndex` which returns `-1`. Lodash `_.findLastIndex` matches this behavior. Watch the clamp ceiling: `array.length - 1`, not `array.length` — using `length` makes the first checked index a hole/`undefined`.

## ⭐ Revision Notes

### Key Facts

- Right-to-left scan; default `fromIndex = array.length - 1`
- Clamp ceiling is `array.length - 1`, floor is `0` — both bounds inclusive search
- Asymmetric vs forward version: out-of-bounds still searches here

### Common Interview Questions

- **Why `array.length - 1` not `array.length` for positive clamp?** Start index is where search begins; `length` would be past the end
- **Difference from forward `findIndex` edge cases?** Forward: positive OOB → `-1`. Backward: positive OOB → start from last index
- **What if array empty?** `fromIndex` default `-1` → `max(-1 + 0, 0) = 0`... actually `min(-1, -1) = -1`, loop `i >= 0` never runs → `-1`

### Interview Takeaways

- Mirror of `findIndex` — reverse loop direction, adjust clamp ceiling by one
- Both bounds matter here: `max(..., 0)` and `min(..., length - 1)`
- Same predicate signature discipline

### Related

- [[Index Normalization]]
- [[Find Index]]
- [[Array.prototype.findLastIndex]]
