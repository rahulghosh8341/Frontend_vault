---
title: Implement a function rangeRight([start=0], end, [step=1]) that creates an array of numbers progressing from start up to, but not including, end with a specified step, similar to range, but in descending order
aliases:
  - Range Right
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[Range]]"
concepts:
  - "[[Rest Parameters]]"
  - "[[Array Methods]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Build numeric sequence from start downwards to end (exclusive), matching Lodash's `_.rangeRight` semantics.

## Problem

Implement a function `rangeRight(start = 0, end, step = 1)` that behaves like `range` but produces a descending sequence. The generated numbers progress from `start` toward `end` (exclusive), stepping by `step`.

```js
rangeRight(4);            // => [3, 2, 1, 0]
rangeRight(-4);           // => [-3, -2, -1, 0]
rangeRight(1, 5);        // => [4, 3, 2, 1]
rangeRight(0, 20, 5);     // => [15, 10, 5, 0]
rangeRight(0, -4, -1);    // => [-3, -2, -1, 0]
rangeRight(1, 4, 0);      // => [1, 1, 1]
rangeRight(0);            // => []
```

**Constraints:** Return empty array when `start === end`. Do not mutate input parameters.

## Companies

- None

## Pattern

- [[Range]]

## 🤔 Thought Process

1. Normalise arguments like `range`: single argument sets `end` and resets `start` to 0, defaulting `step` to `-1` if `end < 0`.
2. If `start === end`, return [].
3. Handle `step === 0`: repeat `start` `end - start` times, assuming `start < end` because sequence must move toward `end`.
4. Construct result array by iterating from `start` toward `end` with `step`, using a loop that continues while the current value is greater than `end` when the step is negative, or less than `end` when the step is positive but inverted for descending order.

## 💻 Final Solution

```js
export default function rangeRight(start = 0, end, step = 1) {
  if (end === undefined) {
    end = start;
    start = 0;
    if (end < 0) step = -1;
  }

  if (start === end) return [];
  
  if (step === 0) {
    if (start >= end) return [];
    return Array(end - start).fill(start);
  }

  const result = [];
  for (let v = start; step > 0 ? v < end : v > end; v += step) result.push(v);
  result.reverse();
  
  return result;
}
```

## 🤔 Why This Works

- Argument normalization mirrors `range`, so the rest of the logic is identical except the final array is reversed when the step is positive.
- Reversing the array at the end keeps the sequence in descending order while re‑using the existing forward‑loop logic.
- Zero‑step handling stays the same: we produce a repeated `start` value for the length implied by `end - start`.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Lodash provides `_.rangeRight` with the same behaviour.
- Time: **O(n)**; Space: **O(n)**.
- Edge: `rangeRight(-2, -5, 0)` returns `[]` because `start >= end`.

## ⭐ Revision Notes

### Key Facts

- The core difference from `range` is the final `reverse()` when the step is positive; without it the loop would produce an ascending array.
- This approach keeps the logic simple and avoids writing two separate loops.

### Common Interview Questions

- "How does the algorithm differ when the desired sequence is descending?"
- "What happens if you call `rangeRight(5,5)`?"

### Interview Takeaways

- Demonstrate familiarity with Lodash `rangeRight` and clarify the edge‑case handling for zero step.

### Related

- [[Range]]
