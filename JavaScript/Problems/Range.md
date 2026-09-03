---
title: Implement range([start=0], end, [step=1]) that creates an array of numbers progressing from start up to, but not including, end
aliases:
  - Range
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[JavaScript/Patterns/Range|Range]]"
concepts:
  - "[[Rest Parameters]]"
  - "[[Array Methods]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Create numeric sequence from start to end (exclusive) with optional step, supporting negative ranges, zero step repetition, and default arguments.

## Problem

Implement a function `range(start = 0, end, step = 1)` that returns an array of numbers progressing from `start` to `end` (exclusive). Handles:
- Single-argument call → `start` becomes 0, `end` the argument.
- Negative ranges → step defaults to `-1` when `end < 0` and `step` omitted.
- Zero step → repeat `start` `end - start` times.
- Return empty array when `start === end`.

```js
range(4);            // => [0,1,2,3]
range(-4);           // => [0,-1,-2,-3]
range(1,5);         // => [1,2,3,4]
range(0,20,5);       // => [0,5,10,15]
range(0,-4,-1);      // => [0,-1,-2,-3]
range(1,4,0);       // => [1,1,1]
range(0);            // => []
```

**Constraints:** Return empty array if start equals end. Do not mutate input parameters.

## Companies

- None

## Pattern

- [[JavaScript/Patterns/Range|Range]]

## 🤔 Thought Process

1. Normalise arguments when only one value is supplied: set `end = start`, `start = 0`, and `step = -1` if `end < 0`.
2. Early‑return if `start === end`.
3. Handle `step === 0`: return `Array(end - start).fill(start)` if `start < end`, otherwise return [].
4. Build result array using a loop: positive step with `< end`, negative step with `> end`.

## 💻 Final Solution

```js
export default function range(start = 0, end, step = 1) {
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
  if (step > 0) {
    for (let v = start; v < end; v += step) result.push(v);
  } else {
    for (let v = start; v > end; v += step) result.push(v);
  }
  return result;
}
```

## 🤔 Why This Works

- Arg normalisation guarantees `start` always represents the first number in the sequence.
- Positive/negative step handling ensures correct loop bounds without `>=` overflow.
- Zero‑step case gives reproducible fixed‑length array by using `Array(fill)` instead of a loop.
- No mutation: we never alter `start`, `end`, or `step` beyond local copies.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Lodash implements `_.range` with identical behaviour.
- Time: **O(n)**; Space: **O(n)**.
- Edge‑cases: `range(-2, -5, 0)` returns `[]` because `start >= end`.

## ⭐ Revision Notes

### Key Facts

- The single‑argument case is the most subtle: it flips start/end and sets a negative step for negative ranges.
- Zero‑step logic must guard against empty ranges.

### Common Interview Questions

- "How does JavaScript handle reference vs value in default params?"
- "What happens if you call `range(5,5)`?"

### Interview Takeaways

- Argument defaults & conditional normalisation are a common interview theme.
- Demonstrate care for edge cases: zero, negative, and overshoot.

### Related

- [[Range]]
