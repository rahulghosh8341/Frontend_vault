---
aliases:
  - Min Max By
---
## Core Idea

Find the extremal element (minimum or maximum) in a collection by evaluating an iteratee function on each element, preserving the first-encountered element on ties and skipping null/undefined iteratee results.

## Recognition

Use when a problem involves:
- Finding max/min of objects by a specific property
- Finding max/min with custom transformation rules (`iteratee`)
- Ignoring missing/null values during reduction
- First-match tie-breaking

## Template

```js
function maxBy(array, iteratee) {
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

## Variations

- `minBy`: check `value < minValue` instead of `>`
- Reducing with initial value vs lazy initialization (`hasCandidate` flag)
- Multi-property tie breaking

## Complexity

- Time: **O(n)** — single pass through the array
- Space: **O(1)** — stores only pointers to best element and best value

## Common Mistakes

- Initializing max value to `0`, which breaks when all values are negative.
- Initializing max value to `-Infinity` or `undefined` without a `hasCandidate` flag when custom values (like objects) are compared.
- Using `>=` instead of `>`, which returns the **last** matching element on ties instead of the **first**.
- Returning the computed iteratee result instead of the **original array element**.
- Not skipping `null` or `undefined` results from iteratees.

## Interview Tips

- Explain why `hasCandidate` flag is needed (handles negative numbers, zero, and empty arrays safely).
- Clarify tie-breaking: problem specifies returning the *first* one, hence strict `>` (not `>=`).
- Mention Lodash compatibility (`_.maxBy`).

## Problems Using This Pattern

- [[Max By]]
- [[Min By]]

## Related Patterns

- [[Array Traversal]]
- [[Reduction]]

## Related Concepts

- [[Iteratee]]
- [[Nullish Values]]
- [[Tie Breaking]]
