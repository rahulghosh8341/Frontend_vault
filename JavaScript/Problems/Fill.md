---
title: Implement a function fill(array, value, [start=0], [end=array.length]) that fills an array with values from start up to, but not including, end. This method mutates array.
aliases:
  - Fill
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Index Normalization]]"
  - "[[Range Checking]]"
concepts:
  - "[[Array.prototype.fill]]"
  - "[[Negative Indices]]"
  - "[[Mutation]]"
solved: true
solvedDate: 2026-08-30
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Fill array with value from start to end (exclusive), mutates in place

## Problem

`fill(array, value, [start=0], [end=array.length])` — replace elements in `[start, end)` with `value`. Mutates and returns the same array. Negative indices count from end; out-of-bounds indices clamp.

```javascript
fill([1, 2, 3], 'a');                 // ['a', 'a', 'a']
fill([4, 6, 8, 10], '*', 1, 3);       // [4, '*', '*', 10]

// out-of-bounds
fill([4, 6, 8, 10, 12], '*', 1, 8);   // [4, '*', '*', '*', '*']
fill([4, 6, 8, 10, 12], '*', 8, 10);  // [4, 6, 8, 10, 12]

// negative in-bounds
fill([4, 6, 8, 10, 12], '*', -3, -1); // [4, 6, '*', '*', 12]

// negative out-of-bounds
fill([4, 6, 8, 10, 12], '*', -10, 2); // ['*', '*', 8, 10, 12]
fill([4, 6, 8, 10, 12], '*', -10, -8);// [4, 6, 8, 10, 12]
```

## Companies

- None

## Pattern

- [[Index Normalization]]
- [[Range Checking]]

## 🤔 Thought Process

- Checks whether the input is an array.
- Sets default values for `start` and `end`.
- Converts negative indexes to positions counted from the end.
- Limits `start` and `end` to valid array boundaries.
- Loops from `start` to `end - 1`.
- Replaces each selected element with `value`.
- Mutates and returns the original array.

## 💻 Final Solution

```javascript
export default function fill(array, value, start = 0, end = array.length) {
  if (!Array.isArray(array)) return array;

  start = start < 0 ? Math.max(array.length + start, 0) : Math.min(start, array.length);
  end   = end   < 0 ? Math.max(array.length + end,   0) : Math.min(end,   array.length);

  for (let i = start; i < end; i++) {
    array[i] = value;
  }
  return array;
}
```

## 🤔 Why This Works

- **Translate then clamp**: negative index becomes `length + index`, then `Math.max(..., 0)` floors it. Positive index gets `Math.min(..., length)` ceiling.
- `-10` on length 5 → `5 + (-10) = -5` → `max(-5, 0) = 0`. Correct.
- `end = 8` on length 5 → `min(8, 5) = 5`. Correct.
- Empty range (`start >= end`) needs no special case — `for` loop condition `i < end` fails immediately, array returned untouched.
- `array[i] = value` writes in place, so mutation requirement satisfied; returning `array` (not a copy) matches native `Array.prototype.fill`.

## 🐞 Bugs I Made

None.

## Production Considerations

Native `Array.prototype.fill(value, start, end)` already implements this exact spec, including negative-index normalization — `array.fill('*', 1, 3)`. Native `fill` also fills holes in sparse arrays (unlike `forEach`/`map`). Lodash `_.fill` additionally supports `_.fill(array, _, start, end)` placeholder semantics and coerces non-integer indices via `toInteger`.

## ⭐ Revision Notes

### Key Facts

- Range is `[start, end)` — end exclusive
- Order matters: translate negatives first, then clamp
- Mutates original array, returns same reference
- `start >= end` after normalization → no-op, not an error
- Native `Array.prototype.fill` follows identical rules

### Common Interview Questions

- **Why `Math.max` for negatives but `Math.min` for positives?** Negatives floor at 0, positives ceiling at length
- **What happens with `fill(arr, 'x', 3, 1)`?** Nothing — empty range, loop never runs
- **Does native fill handle sparse arrays?** Yes, it writes to holes; `map`/`forEach` skip them
- **Why return `array` instead of a new array?** Spec says mutate in place; matches native behavior

### Interview Takeaways

- Clamping order is the trap — say "translate, then clamp" explicitly
- Empty-range no-op falls out of the loop condition for free, no guard needed
- Know that native `Array.prototype.fill` exists and matches this spec

### Related

- [[Index Normalization]]
- [[Range Checking]]
- [[Array.prototype.fill]]
- [[Clamp]]
