---
aliases:
  - Index Normalization
---

## Core Idea

Turn a possibly-negative, possibly-out-of-range index into a real array index: translate negatives from the end, then clamp to `[0, length]`.

```
index < 0  → max(length + index, 0)
index >= 0 → min(index, length)
```

## Recognition

- Optional `start` / `end` arguments on an array operation
- Negative indices must count from the end
- Out-of-bounds indices must be clamped, not throw
- Polyfills: `fill`, `slice`, `splice`, `at`, `indexOf(..., fromIndex)`

## Template

```javascript
function normalize(index, length) {
  return index < 0
    ? Math.max(length + index, 0)
    : Math.min(index, length);
}

// Usage in a range-based operation
function fill(array, value, start = 0, end = array.length) {
  start = normalize(start, array.length);
  end = normalize(end, array.length);
  for (let i = start; i < end; i++) array[i] = value;
  return array;
}
```

## Variations

- **Defaults**: `start = 0`, `end = array.length` via parameter defaults
- **Empty range guard**: if normalized `start >= end`, loop body never runs — no extra check needed
- **`toInteger` first**: Lodash coerces fractional/`NaN` indices before normalizing
- **`Array.prototype.at`**: single index, returns `undefined` when out of range instead of clamping

## Complexity

- Time: O(1) per index
- Space: O(1)

## Common Mistakes

- Clamping before translating negatives — `-10` clamps to `0`, then translate gives wrong result. Order: translate, then clamp
- Using `length - index` instead of `length + index` for negatives
- Forgetting `Math.max(..., 0)` — `-10 + 5 = -5` still negative
- Treating `start > end` as an error; correct behavior is no-op

## Interview Tips

- State the two-step rule out loud: translate, then clamp
- Walk through one negative out-of-bounds case (`-10` on length 5) to prove the order matters
- Mention native `Array.prototype.fill` follows same normalization

## Problems Using This Pattern

- [[Fill]]
- [[Clamp]]
- [[Array.prototype.at]]

## Related Patterns

- [[Range Checking]]
- [[Array Traversal]]

## Related Concepts

- [[Array.prototype.fill]]
- [[Negative Indices]]
