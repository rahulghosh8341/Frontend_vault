---
title: Memoize II
aliases:
  - Memoize II
difficulty: Medium
time: 30 min
languages:
  - JavaScript
companies:
  - "[[Airbnb]]"
  - "[[LinkedIn]]"
pattern:
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
  - "[[Function.prototype.apply]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-12
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 30 min
> Implement `memoize(func)` that caches results for multiple string/number arguments.

## Problem

Return memoized version of `func` that accepts multiple arguments (strings or numbers). Cache results keyed by full argument list. Preserve `this` binding.

```js
const memoized = memoize(expensiveMul);
memoized(3, 7); // computes, caches
memoized(3, 7); // cached
memoized(5, 8); // computes, caches
```

## Companies

- [[Airbnb]]
- [[LinkedIn]]

## Pattern

- [[Closure]]

## 🤔 Thought Process

The goal of memoization is:

> **Same arguments → return cached result instead of calling `func` again.**

For every call:

1. Collect all arguments.
2. Create a key representing the **entire ordered argument list**.
3. Check whether that key exists in the `Map`.
4. If it exists → return cached result.
5. Otherwise → call `func`, store the result, and return it.

```js
const query = JSON.stringify(value);
```

Example:

```text
(3, 7) → "[3,7]"
(5, 8) → "[5,8]"
(3, 7) → "[3,7]" → cache hit
```

## 💻 Final Solution

```js
export default function memoize(func) {
  let result = new Map();

  return function(...value){
    let computedRes;
    let query = JSON.stringify(value);
    if(result.has(query)){
      return result.get(query)
    }
    computedRes = func.apply(this,value)
    result.set(query,computedRes);
    return computedRes;
  }
}
```

## 🤔 Why This Works

The cache represents:

```text
arguments → result

[3,7] → 21
[5,8] → 40
```

Using the serialized arguments as the Map key lets us reliably check whether we've already computed that **exact call**.

The important part:

```js
if (result.has(query)) {
  return result.get(query);
}

const computedRes = func.apply(this, value);

result.set(query, computedRes);
```

`func.apply(this, value)` is important because the question requires `func` to be called with the **memoized function's `this` binding**.

## 🐞 Bugs I Made

### Used different keys for `has/get` and `set`

Initially:

```js
result.has(query)
result.get(query)

result.set(value, computedRes) // ❌
```

`query` and `value` are different:

```text
value = [3, 7]
query = "[3,7]"
```

So use `query` consistently:

```js
result.set(query, computedRes); // ✅
```

## Production Considerations

- `JSON.stringify` limitations: argument order in objects, `undefined`, `NaN`, circular refs. Safe here since args are only strings/numbers.
- Alternative: nested `Map` (trie-like) per argument avoids serialization cost.
- No cache eviction. Production use needs LRU or TTL.

## ⭐ Revision Notes

### Key Facts

- Memoization = **cache function results**.
- Use a `Map` for the cache.
- The cache key must represent the **full argument list**.
- Argument order matters: `[3, 7]` ≠ `[7, 3]`
- `JSON.stringify(value)` creates a usable key for this problem because arguments are only strings/numbers.
- `func.apply(this, value)` preserves the required `this` binding.
- Cache only after computing a missing result.

### Common Interview Questions

- Why `JSON.stringify` not `args.join`? `join` loses type info: `(1, "23")` and `(12, "3")` would collide.
- Why `apply` not `call`? Multiple args from array. `apply` spreads array as arguments.
- Limitation of `JSON.stringify`? Fails on `undefined`, functions, circular refs. Fine for strings/numbers only.

### Interview Takeaways

- New concept vs Memoize I: cache key from **multiple arguments**
- Consistent key usage across `has`/`get`/`set` is critical
- Connection chain: Higher-Order Functions > Closure > Map > `this` > `apply` > Memoize II

### Related

- [[Closure]]
- [[Memoize]]
- [[Once]]
