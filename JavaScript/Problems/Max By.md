---
title: Implement maxBy(array, iteratee) that finds the element inside array with the maximum value returned by iteratee
aliases:
  - Max By
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
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Find the element with the maximum iteratee result, skipping null/undefined, and returning the first on ties.

## Problem

Implement a function `maxBy(array, iteratee)` that finds the element inside `array` with the maximum value returned by `iteratee`. The iteratee is invoked with one argument: `(value)`.

```js
maxBy([{ n: 1 }, { n: 2 }], (o) => o.n); // => { n: 2 }
maxBy([1, 2], (o) => -o); // => 1
maxBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => undefined
```

**Constraints:** ignore elements where iteratee produces `null` or `undefined`, return the first element on ties, return `undefined` if no valid elements exist.

## Companies

- None

## Pattern

- [[Min Max By]]

## 🤔 Thought Process

1. Traverse the array.
2. Run `iteratee(element)` for every element.
3. If the iteratee result is `null` or `undefined`, **skip it**.
4. The first valid result becomes the initial maximum using a `hasCandidate` flag.
5. For subsequent valid results, replace the maximum only when `value > maxValue`.
6. Return the **original element**, not the iteratee result.

## 💻 Final Solution

```js
export default function maxBy(array, iteratee) {
  let maxElement;
  let maxValue;
  let hasCandidate = false;

  for (const element of array) {
    const value = iteratee(element);

    if (value === null || value === undefined) {
      continue;
    }

    if (!hasCandidate || value > maxValue) {
      maxValue = value;
      maxElement = element;
      hasCandidate = true;
    }
  }

  return maxElement;
}
```

## 🤔 Why This Works

**Separation of element and value.** Storing `maxElement` and `maxValue` separately ensures we return the original array object `{ n: 2 }` instead of the primitive `2`.

**First valid candidate initialization.** Using `hasCandidate = false` instead of seeding with `0` or `-Infinity` handles arrays of all-negative numbers (e.g. `[1, 2]`, `o => -o`) correctly without initial-value bias.

**Strict greater-than (`>`).** Prevents overwriting on ties, ensuring the *first* matching element is preserved.

**Nullish check.** `value === null || value === undefined` skips missing properties gracefully.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Lodash provides `_.maxBy(array, [iteratee=_.identity])`.
- Complexity: `O(n)` time, `O(1)` space.
- Defensive checks: ensure `iteratee` is a function if public-facing.

## ⭐ Revision Notes

### Key Facts

- Use a boolean flag (`hasCandidate`) for lazy initialization when initial default values cannot be safely guessed.
- Strict `>` preserves first occurrence on ties; `>=` would preserve the last.
- Skip `null`/`undefined` explicitly because comparison operators coerce them in unexpected ways (`null > 0` is false, but `null >= 0` is also false — yet `null == 0` is weird).

### Common Interview Questions

- What if all items return `undefined`? Returns `undefined` (since `maxElement` stays unassigned).
- How to implement `minBy`? Just change `value > maxValue` to `value < minValue`.

### Interview Takeaways

- Track both the matched item and its comparison metric when returning non-primitive results.

### Related

- [[Min Max By]]
- [[Min By]]
- [[Array Traversal]]
