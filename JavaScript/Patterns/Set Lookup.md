---
aliases:
  - Set Lookup
---

## Core Idea

Use a `Set` for fast membership testing (`has()`) instead of linear scans (`includes()`, nested loops). Reduces complexity from O(n²) to O(n).

## Recognition

- "Does X exist in this list/collection?"
- Remove duplicates, filter by another array, intersection/union/difference
- Need to check membership repeatedly within a loop

## Template

```javascript
// Difference of two arrays
function difference(array, values) {
  const set = new Set(values);
  return array.filter((item) => !set.has(item));
}

// Unique values
function unique(array) {
  return [...new Set(array)];
}

// Intersection
function intersection(a, b) {
  const set = new Set(b);
  return a.filter((item) => set.has(item));
}
```

## Variations

- **Sparse arrays**: skip empty slots (`!(i in array)` check) since holes read as `undefined`
- **Object identity**: Set uses SameValueZero — objects compared by reference, not structure
- **Chaining**: filter + Set for union/difference/intersection compositions
- **Multi-set**: count frequencies with `Map` when uniqueness insufficient (e.g., Intersection By)

## Complexity

- Build Set: O(n) time, O(n) space
- Each `has()`: O(1)
- Full filter pass: O(n)
- Total: O(n + m), beats nested-loop O(n·m)

## Common Mistakes

- Using `includes()` in loop → accidental O(n²)
- Forgetting `NaN`: `set.has(NaN)` works (SameValueZero), `array.includes(NaN)` also works — but `_===` does not
- Treating `undefined` holes as real values in sparse arrays
- Expecting structural equality for objects — Set is reference-based

## Interview Tips

- Name the complexity win: "Set gives O(1) lookup vs O(n) includes"
- Mention `NaN` handling — shows SameValueZero knowledge
- Offer the `new Set(array)` one-liner for unique before reaching for loops

## Problems Using This Pattern

- [[Difference]]
- [[Unique Array]]
- [[Intersection]]

## Related Patterns

- [[Array Traversal]]
- [[Fixed Size Grouping]]

## Related Concepts

- [[Set]]
- [[NaN Equality]]
- [[Sparse Arrays]]
