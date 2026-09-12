---
title: Memoize
aliases:
  - Memoize
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Airbnb]]"
  - "[[LinkedIn]]"
  - "[[Atlassian]]"
  - "[[Uber]]"
pattern:
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-12
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Implement `memoize(func)` that caches results for single string/number arguments.

## Problem

Return memoized version of `func`. Cache results keyed by argument. Subsequent calls with same argument return cached result without re-invoking `func`. Preserve `this` binding.

```js
const memoized = memoize(expensiveFunction);
memoized(5);  // computes, caches
memoized(5);  // returns cached
memoized(10); // computes, caches
```

## Companies

- [[Airbnb]]
- [[LinkedIn]]
- [[Atlassian]]
- [[Uber]]

## Pattern

- [[Closure]]

## 🤔 Thought Process

- Need cache: `Map` over plain object because `Map.has()` distinguishes missing key from `undefined` value.
- Check `result.has(value)` before computing. Handles cached `undefined`/`null`/`0` correctly.
- `func.call(this, value)` preserves `this` binding from call site.
- Store computed result in Map, return it.

## 💻 Final Solution

```js
export default function memoize(func) {
  let result = new Map();

  return function(value){
    let computedRes;
    if(result.has(value)){
      return result.get(value)
    }
    computedRes = func.call(this,value)
    result.set(value,computedRes);
    return computedRes;
  }
}
```

## 🤔 Why This Works

- `Map` handles both string and number keys without type coercion. Plain object would coerce `5` and `"5"` to same key.
- `Map.has()` checks key existence, not truthiness. Cached `undefined`, `null`, `false`, `0` all work.
- `func.call(this, value)` forwards `this` from call site. Regular function (not arrow) needed for dynamic `this`.
- Closure over `result` Map persists cache across calls.

## 🐞 Bugs I Made

None.

## Production Considerations

- `Map` keeps strong references to values. Large cached objects never garbage collected. `WeakMap` not usable here since keys are primitives.
- Single argument only. Multi-arg memoization needs serialized cache key or nested Maps (see Memoize II).
- No cache eviction. Production memoize often uses LRU cache with size limit.

## ⭐ Revision Notes

### Key Facts

- `Map` over `{}`: no key coercion, `has()` checks existence not truthiness
- `func.call(this, arg)` for single arg, `func.apply(this, args)` for multiple
- Memoization trades memory for speed
- Only pure functions safe to memoize (same input = same output)

### Common Interview Questions

- Why Map not plain object? Object coerces keys to strings. `obj[1]` and `obj["1"]` collide. Map preserves type.
- What if `func` returns `undefined`? `Map.has()` still returns `true` for that key. Plain object `if (cache[key])` would miss it.
- Why `call` not `apply`? Single argument. Either works, `call` more direct for single arg.

### Interview Takeaways

- Map vs Object for cache is common follow-up
- `has()` vs truthiness check is subtle but important
- Foundation for Memoize II (multi-arg)

### Related

- [[Closure]]
- [[Make Counter II]]
- [[Once]]
- [[Memoize II]]
