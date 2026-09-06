---
aliases:
  - Type Checking
---

## Core Idea

Determine the runtime type / category of a value so the right operation can be applied to it. JavaScript's `typeof`, `instanceof`, `Array.isArray`, and the object's own properties (`length`, `size`, `Object.keys`) are the building blocks for branching on type.

## Recognition

Use this pattern when the problem requires behavior to vary based on the value's shape:
- Different collection types need different "size" / "empty" checks (Array, String, Map, Set, plain Object).
- A function must accept multiple types and dispatch differently (e.g., `isEmpty`, `Size`).
- A guard is needed before performing type-specific operations.

## Template

```js
function handle(value) {
  // 1. Null / undefined first (typeof null === "object")
  if (value === null || value === undefined) return /* ... */;

  // 2. Primitives vs objects
  if (typeof value !== 'object') return /* ... */;

  // 3. Specific object subtypes (order matters)
  if (Array.isArray(value)) return /* ... */;
  if (value instanceof Map || value instanceof Set) return /* ... */;

  // 4. Generic plain object fallback
  return /* ... */;
}
```

## Variations

- **Size / isEmpty checks**: `length` for arrays/strings, `size` for Map/Set, `Object.keys().length` for plain objects.
- **Iteration dispatch**: choose `for...of` for iterables, `Object.entries` for plain objects.
- **Equality dispatch**: `_===` for primitives, reference equality for objects, custom comparator via `intersectionWith`.

## Complexity

Time: **O(1)** for the type check itself.
For collection-size variants: **O(1)** for arrays/strings/Map/Set; **O(n)** for plain objects via `Object.keys()`.

## Common Mistakes

- Forgetting `typeof null === "object"` — must guard `null` before the object branch.
- Using `instanceof Array` instead of `Array.isArray` — fails across frames (iframes).
- Using `value.length` on a Map or Set — they don't have `length`, only `size`.
- Writing `Object.keys(null)` — throws. Guard first.
- Ternary `condition ? true : false` — the condition is already boolean.

## Interview Tips

- State the "check order" rule up front: null/undefined → typeof → Array.isArray → instanceof → generic object.
- When asked about `isEmpty` / `Size`, draw the dispatch table:
  - Array/String → `length`
  - Map/Set → `size`
  - Object → `Object.keys().length`
  - Primitive → empty / size 0
- Mention `typeof` quirks: `typeof null === "object"`, `typeof [] === "object"`, `typeof function() {} === "function"`.

## Problems Using This Pattern

- [[Is Empty]]
- [[Size]]
- [[Intersection]]
- [[Intersection With]]

## Related Patterns

- [[Array Traversal]]
- [[Set Lookup]]
- [[Range Checking]]

## Related Concepts

- [[Type Coercion]]
- [[Equality]]
- [[Collections]]
