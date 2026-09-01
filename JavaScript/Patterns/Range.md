---
aliases:
  - Range
---

## Core Idea
Generate a numeric sequence from a start point up to, but not including, an end point, optionally advancing by a step value. Supports negative ranges, zero step repetition, and single-argument defaulting.

## Recognition
Use when a problem needs:
- Integer or floating‑point ranges
- Negative, positive, or mixed sign ranges
- Constant or variable step increments
- Inclusive start, exclusive end without mutation of inputs

## Template
```js
export default function range(start = 0, end, step = 1) {
  if (end === undefined) {
    end = start;
    start = 0;

    if (end < 0) {
      step = -1;
    }
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

## Variations
- `rangeClosed(start, end, step)` includes the end value
- `rangeInfinite(step)` returns an infinite generator
- `rangeFrom(end, step)` starts from `0` by default

## Complexity
- Time: **O(n)** where *n* is the count of produced numbers
- Space: **O(n)** for the resulting array

## Common Mistakes
- Forgetting to handle the single‑argument normalization
- Ignoring negative step direction – a positive step will never reach a negative end
- Using `>=` or `>` with negative step incorrectly
- Assuming `Array(end - start)` works with negative ranges – in that case use `Math.abs(end - start)` *only* for zero step

## Interview Tips
- Emphasize argument defaulting: no need to supply `undefined` on the third param
- Explain the distinct handling for zero step: it produces a repeated start value
- Contrast with integer step: `step > 0` vs `step < 0`
- Mention Lodash's `_.range` and its zero step behavior

## Problems Using This Pattern
- [[Range]]

## Related Patterns
- [[Set Lookup]]
- [[Array Traversal]]

## Related Concepts
- [[Rest Parameters]]
- [[Array Methods]]
