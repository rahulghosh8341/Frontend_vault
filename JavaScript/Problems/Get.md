---
title: "Implement the `_.get` method"
aliases:
  - Get
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - Meta
pattern:
  - "[[Object Path Traversal]]"
concepts:
  - "[[Object Property Access]]"
  - "[[Path Normalization]]"
  - "[[Falsy Values]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Safely access deeply nested object properties via a dot-separated path or array of keys, returning a default when the path can't be fully resolved.

## Problem

Implement `get(object, path, [defaultValue])` that returns the value at `path` in `object`.

`path` can be a dot-separated string (`profile.name.firstName`) or an array (`['profile', 'name', 'firstName']`).

If the resolved value is `undefined`, return `defaultValue`. If an intermediate value is non-traversable (primitives like `null`), return `defaultValue` if provided, else `undefined`.

```js
get(john, 'profile.name.firstName'); // => 'John'
get(jane, 'profile.name.firstName'); // => undefined
get({ a: { b: true } }, 'a.b.c');   // => undefined
get({ a: 0 }, 'a', 'fallback');     // => 0
```

## Companies

- [[ByteDance]] [[Amazon]] [[TikTok]]

## Pattern

- [[Object Path Traversal]]

## 🤔 Thought Process

1. **Normalize path**: string → `split('.')`, array → copy with spread.
2. Set `current = objectParam`.
3. Walk each segment. Before accessing `current[segment]`, verify `current` is traversable (`typeof === 'object' && !== null`).
4. After loop, if result is `undefined` → return `defaultValue`. Else return result.

Key insight: preserve falsy values (`0`, `false`, `''`). Don't use truthiness checks — only check for `undefined` at the end.

## 💻 Final Solution

```js
/**
 * @param {Record<string, unknown> | Array<unknown>} objectParam
 * @param {string | Array<string | number>} pathParam
 * @param {unknown} [defaultValue]
 * @return {unknown}
 */
export default function get(objectParam, pathParam, defaultValue) {
  let path = [];
  if (typeof pathParam === 'string') {
    path = pathParam.split('.');
  } else {
    path = [...pathParam];
  }
  let current = objectParam;
  for (let segment of path) {
    if (typeof current !== 'object' || current === null) {
      return defaultValue;
    }
    current = current[segment];
  }
  return current === undefined ? defaultValue : current;
}
```

## 🤔 Why This Works

- **Path normalization** converts both input formats into one uniform array, letting the rest of the code use a single loop.
- **Traversability check** (`typeof current !== 'object' || current === null`) stops traversal when hitting a primitive — `typeof null === 'object'` is a JS quirk, so `|| current === null` is essential.
- **`undefined` check at end** preserves falsy values like `0` and `false` — only `undefined` triggers the default.
- **`for...of`** iterates values, not indices (unlike `for...in`).

## 🐞 Bugs I Made

1. **Used `for...in` instead of `for...of`** — got array indices (0, 1, 2) instead of values ('a', 'b', 'c').
2. **`typeof current === undefined`** — `typeof` returns a string; should be `_=== 'undefined'` or just `current === undefined`.
3. **Used `&&` instead of `||`** in traversability check — `typeof !== 'object' && current === null` never catches `typeof null === 'object'` cases.
4. **Didn't stop on primitive intermediate** — traversing through `true` to `.b` instead of returning default early.

## Production Considerations

- Lodash `_.get` supports bracket syntax (`a[0].b.c`) — this solution handles dot-separated only as specified.
- Native optional chaining (`user?.profile?.name?.firstName`) covers most use cases for static paths.
- `_.get` is useful for dynamic/computed paths from config or user input.
- Consider performance for deeply nested objects with many path segments — each access is O(1) but adds up.

## ⭐ Revision Notes

### Key Facts

- Normalize path first, then single traversal algorithm
- `typeof null === 'object'` — must explicitly check `current === null`
- Never use truthiness for traversability — `0`, `false`, `''` are valid resolved values
- `for...of` gives values, `for...in` gives indices
- Only return `defaultValue` when resolved value is `undefined`, not when it's falsy

### Common Interview Questions

- Why not use truthiness check? → Falsy values like `0` and `false` are valid return values
- How does `typeof null` work? → Returns `'object'` (historic JS bug)
- Could you implement this recursively? → Yes: base case empty path, recursive case resolve first segment and recurse with rest
- How to handle bracket syntax like `a[0].b`? → Additional parsing step, split on `[` and `]`

### Interview Takeaways

- **Path normalization first** — one algorithm for both input formats
- **Check traversability before access** — defensive property access pattern
- **`undefined` vs falsy** — the core distinction in this problem
- Mention optional chaining as modern alternative for static paths

### Related

- [[Object Path Traversal]]
- [[Object Property Access]]
- [[Path Normalization]]
