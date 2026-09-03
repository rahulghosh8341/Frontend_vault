---
title: Implement minBy(array, iteratee) that finds the element inside array with the minimum value returned by iteratee
aliases:
  - Min By
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[Min Max By]]"
concepts:
  - "[[Iteratee]]"
  - "[[Reduction]]"
solved: true
solvedDate: 2026-09-01
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Find the element with the minimum iteratee result, skipping null/undefined, returning the first on ties.

## Problem

Implement a function `minBy(array, iteratee)` that finds the element inside `array` with the minimum value returned by `iteratee`.

```js
minBy([2, 3, 1, 4], (num) => num); // => 1
minBy([{ n: 1 }, { n: 2 }], (o) => o.n); // => { n: 1 }
minBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => undefined
```

**Constraints:** ignore `null`/`undefined` iteratee results, return the first element on ties, return `undefined` if no valid elements exist.

## Companies

- None

## Pattern

- [[Min Max By]]

## 🤔 Thought Process

1. Traverse array.
2. Calculate `current = iteratee(element)` for each.
3. Skip if `null` or `undefined`.
4. Use `hasCandidate` flag to set initial candidate on first valid element.
5. Replace minimum only if `current < minValue` (strict `<` preserves first element on ties).
6. Return `minElement`.

## 💻 Final Solution

```js
export default function minBy(array, iteratee) {
  let hasCandidate = false;
  let minValue;
  let minElement;

  for (const element of array) {
    let current = iteratee(element);

    if (current === null || current === undefined) {
      continue;
    }

    if (!hasCandidate || current < minValue) {
      minValue = current;
      hasCandidate = true;
      minElement = element;
    }
  }

  return minElement;
}
```

## 🤔 Why This Works

Dual tracking (`minValue` + `minElement`) retains the reference to the original array element while comparing transformed primitives. `hasCandidate` cleanly handles negative values and `0`. Strict `<` ensures tie-breaking keeps the earliest element.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Standard Lodash function: `_.minBy`.
- `O(n)` time, `O(1)` space.

## ⭐ Revision Notes

### Key Facts

- Mirror image of `maxBy` — only operator flips (`<` instead of `>`).
- Flag-based candidate initialization avoids `Infinity` sentinel pitfalls.

### Common Interview Questions

- Difference between `minBy` and `maxBy`? Only the comparison operator flips.

### Interview Takeaways

- Both `minBy` and `maxBy` share the exact `[[Min Max By]]` pattern.

### Related

- [[Min Max By]]
- [[Max By]]
