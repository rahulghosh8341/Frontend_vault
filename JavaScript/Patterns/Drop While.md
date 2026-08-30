---
aliases:
  - Drop While
---

## Core Idea

Drop elements from one end of an array while a predicate returns truthy; stop at the first element where the predicate is falsy. The result is the remaining slice — everything before (drop-left) or after (drop-right) the boundary. Contrast with `filter`, which drops every matching element.

## Recognition

- "Remove elements from the start/end until the condition stops"
- Prefix/suffix trimming by a condition (not by fixed count — that's Drop/DropRight)
- Predicate invoked per element, often with `(value, index, array)`

## Template

```javascript
// Drop from left until predicate falsy
function dropWhile(array, predicate) {
  let index = 0;
  while (index < array.length && predicate(array[index], index, array)) {
    index++;
  }
  return array.slice(index);
}

// Drop from right until predicate falsy
function dropRightWhile(array, predicate) {
  let index = array.length - 1;
  while (index >= 0 && predicate(array[index], index, array)) {
    index--;
  }
  return array.slice(0, index + 1);
}
```

## Variations

- **Drop (fixed count)**: drop first/last N elements regardless of predicate
- **Drop While with Set lookups**: drop while element in a forbidden set
- **Boundary-index variant**: find boundary index first (`findIndex`/`findLastIndex`), then slice

## Complexity

- O(n) worst case, single pass
- O(1) extra space (slice returns new array, that's O(n) output)

## Common Mistakes

- Off-by-one on `slice(0, index + 1)` for right drops
- Using `filter` instead of `while` — filter removes all matches, drop-while removes only the contiguous run
- Forgetting the `index >= 0` / `index < length` guard for all-true predicates
- Not passing all predicate args `(value, index, array)` when spec requires them

## Interview Tips

- Clarify drop-while (contiguous prefix/suffix) vs filter (all matches) early
- Call out `slice` boundary math explicitly — interviewer checks off-by-one
- Mention `while` stops at first falsy → early exit, no full scan

## Problems Using This Pattern

- [[Drop Right While]]
- [[Drop While]]

## Related Patterns

- [[Array Traversal]]
- [[Set Lookup]]

## Related Concepts

- [[Array.prototype.slice]]
- [[Predicate]]
