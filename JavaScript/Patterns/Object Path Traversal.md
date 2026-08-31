---
aliases:
  - Object Path Traversal
---

## Core Idea

Safely traverse nested objects/arrays using a normalized sequence of property keys, returning a default value when the path cannot be fully resolved.

## Recognition

- Input: object + dot-separated string path or array of keys
- Need to handle missing intermediate properties
- Must preserve falsy values (`0`, `false`, `''`)
- Default value returned when path fails or resolves to `undefined`

## Template

```js
function get(obj, path, defaultValue) {
  // 1. Normalize path to array
  const segments = Array.isArray(path) ? [...path] : path.split('.');

  // 2. Walk
  let current = obj;
  for (const segment of segments) {
    if (current == null || typeof current !== 'object') {
      return defaultValue;
    }
    current = current[segment];
  }

  // 3. Resolve
  return current === undefined ? defaultValue : current;
}
```

## Variations

- **Bracket syntax**: Parse `a[0].b.c` → `['a', '0', 'b', 'c']`
- **Recursive approach**: Base case empty path, recurse with remaining segments
- **Set variant**: `set(object, path, value)` — walk to second-to-last segment, assign
- **Has variant**: `has(object, path)` — boolean check, return `false` on any traversal failure

## Complexity

- **Time**: O(d) where d = depth of path
- **Space**: O(d) for path array, O(1) iterative

## Common Mistakes

- `for...in` instead of `for...of` — gives indices, not values
- `typeof null === 'object'` — must explicitly check for null
- Using truthiness check (`!current`) — loses `0`, `false`, `''`
- `typeof current === undefined` — typeof returns string, need `_=== 'undefined'` or direct comparison
- Not stopping when intermediate value is primitive — accessing `.b` on `true`

## Interview Tips

- Mention optional chaining (`?.`) as modern alternative for static paths
- Emphasize normalize-first approach — one algorithm for both input formats
- Discuss `undefined` vs falsy distinction — core concept tested
- Mention recursive alternative if asked for variation

## Problems Using This Pattern

- [[Get]]
- [[Deep Set]] (set variant)

## Related Patterns

- [[DFS Recursion]] (recursive variant)
- [[Index Normalization]]
