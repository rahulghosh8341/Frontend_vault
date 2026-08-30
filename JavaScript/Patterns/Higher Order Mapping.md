---
aliases:
  - Higher Order Mapping
---

## Core Idea

Apply a transformation function to each element of an array, producing a new array of the same length. Uses `Array.prototype.map` under the hood.

## Recognition

- Problem asks to transform every element independently
- Output array has same length as input
- No cross-element dependencies
- Often called "map" pattern

## Template

```javascript
Array.prototype.transform = function(fn) {
  return this.map(fn);
};

// Usage
arr.transform(x => x * x);
```

## Variations

| Variation | Description |
|-----------|-------------|
| In-place | Mutate original (use `forEach`) |
| Async | `Promise.all(arr.map(asyncFn))` |
| Filter + map | `arr.filter(...).map(...)` |

## Complexity

- Time: O(n) — visits each element once
- Space: O(n) — new array allocated

## Common Mistakes

- Using arrow function for prototype method (loses `this`)
- Forgetting to return from callback
- Mutating original array

## Interview Tips

- Explain `this` binding in prototype methods
- Mention `map` vs `forEach` difference (return value)
- Know when to polyfill vs use native

## Problems Using This Pattern

- [[Array.prototype.square]]
- [[Array.prototype.filter]]
- [[Flatten]]
- [[From Pairs]]

## Related Patterns

- [[Array Traversal]]
- [[Filter Pattern]]

## Related Concepts

- [[Prototypes]]
- [[Higher Order Functions]]