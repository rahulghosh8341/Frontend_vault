---
title: Implement a function findIndex(array, predicate, [fromIndex=0]) that takes an array of values, a function predicate, and an optional fromIndex number argument, and returns the index of the first element in the array that satisfies the provided testing function predicate.
aliases:
  - Find Index
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Index Normalization]]"
concepts:
  - "[[Array.prototype.findIndex]]"
  - "[[Predicate]]"
solved: true
solvedDate: 2026-08-31
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Return index of first element where predicate true, -1 if none

## Problem

`findIndex(array, predicate, [fromIndex=0])` scans `array` from `fromIndex` (inclusive) forward, calling `predicate(value, index, array)` on each element. Returns the first index where predicate returns truthy, or `-1` if none found. Handles negative and out-of-bounds `fromIndex`:

- Negative: count from end (`-1` = last element)
- If `fromIndex < -array.length` → start at `0`
- If `fromIndex >= array.length` → no search, return `-1`

```javascript
const arr = [1, 2, 3, 4, 5];

// first > 3
findIndex(arr, (num) => num > 3); // => 3

// start at index 4
findIndex(arr, (num) => num > 3, 4); // => 4

// no such element
findIndex(arr, (num) => num > 10, 3); // => -1

// negative fromIndex
findIndex(arr, (num) => num > 2, -3); // => 2
findIndex(arr, (num) => num % 2 === 0, -3); // => 3

// very negative -> start at 0
findIndex(arr, (num) => num > 2, -10); // => 2

// out-of-bounds high -> -1
findIndex(arr, (num) => num > 2, 10); // => -1
```

## Companies

None

## Pattern

- [[Index Normalization]]

## 🤔 Thought Process

Normalize `fromIndex` using the same translate-then-clamp rule as [[Fill]]:
- if negative: `start = Math.max(fromIndex + array.length, 0)`
- else: `start = Math.min(fromIndex, array.length)`
Then linear scan from `start` to `array.length-1`, return first `i` where `predicate(array[i], i, array)` true. If loop ends, return `-1`.

## 💻 Final Solution

```javascript
export default function findIndex(array, predicate, fromIndex = 0) {
  let start;

  if (fromIndex < 0) {
    start = Math.max(fromIndex + array.length, 0);
  } else {
    start = Math.min(fromIndex, array.length);
  }

  for (let i = start; i < array.length; i++) {
    if (predicate(array[i], i, array)) {
      return i;
    }
  }
  return -1;
}
```

## 🤔 Why This Works

- Normalization matches native `Array.prototype.findIndex` (and `fill`, `slice`, `splice`).
- Negative index translated: `-3` on length 5 → `5 + (-3) = 2`.
- Very negative: `-10 + 5 = -5` → `max(-5, 0) = 0`.
- Out-of-bounds high: `10` → `min(10, 5) = 5` → loop condition `i < 5` fails immediately → `-1`.
- Predicate receives all three args per spec.

## 🐞 Bugs I Made

None.

## Production Considerations

Native `Array.prototype.findIndex(predicate, fromIndex)` already implements this exactly. Lodash `_.findIndex` adds placeholder iteratee shorthands and treats `null`/`undefined` arrays as empty. For manual implementation, the two-step normalization (translate then clamp) is critical — reversing the order breaks negative out-of-bounds cases.

## ⭐ Revision Notes

### Key Facts

- Returns index, not element (contrast with `find`)
- Stops at first truthy predicate
- `fromIndex` default `0`, negative counts from end
- Out-of-bounds handling: clamp to `[0, array.length]`

### Common Interview Questions

- **What if `fromIndex` is negative and larger magnitude than length?** e.g., `findIndex([1,2,3], x=>x>0, -5)` → start at `0`.
- **What if predicate never true?** Loop finishes, return `-1`.
- **Why not use `array.slice(fromIndex).findIndex(...)`?** Creates temporary slice; manual scan avoids extra allocation.

### Interview Takeaways

- Normalize indices the same way as `fill`, `slice`, `splice` — translate then clamp.
- Always pass full predicate signature `(value, index, array)`.
- Early return on match keeps average case better than worst-case O(n).

### Related

- [[Index Normalization]]
- [[Find Last Index]]
- [[Array.prototype.findIndex]]