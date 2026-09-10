---
title: Implement classNames to conditionally join CSS class names
aliases:
  - Classnames
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Higher Order Mapping]]"
concepts:
  - "[[Conditional Logic]]"
  - "[[Array Recursion]]"
solved: true
solvedDate: 2026-09-08
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Conditionally join CSS class names from mixed inputs (strings, objects, arrays).

## Problem

Implement `classNames(...args)` that accepts mixed inputs and returns a space‑separated string of class names. The function should:

- Accept strings, objects, arrays, and falsy values.
- Recursively flatten arrays.
- Treat object keys as class names when their value is truthy.
- Preserve encounter order of arguments.
- Deduplicate class names (first occurrence wins).
- Return string without leading/trailing whitespace.

**Examples**

```js
classNames('foo', 'bar'); // 'foo bar'
classNames('foo', { bar: true }); // 'foo bar'
classNames({ 'foo-bar': true }); // 'foo-bar'
classNames({ 'foo-bar': false }); // ''
classNames({ foo: true }, { bar: true }); // 'foo bar'
classNames({ foo: true, bar: true }); // 'foo bar'
classNames({ foo: true, bar: false, qux: true }); // 'foo qux'
```

Arrays are recursively flattened:

```js
classNames('a', ['b', { c: true, d: false }]); // 'a b c'
```

Mixed values:

```js
classNames(
  'foo',
  {
    bar: true,
    duck: false,
  },
  'baz',
  { quux: true },
); // 'foo bar baz quux'
```

Falsy handling:

```js
classNames(null, false, 'bar', undefined, { baz: null }, ''); // 'bar'
```

## Pattern

- [[Higher Order Mapping]]

## 🤔 Thought Process

* Normalize each argument: strings/numbers → keep; objects → keys; arrays → recurse.
* Use a `Set` to track seen class names for deduplication.
* Recursively process arrays to flatten them.
* Preserve order by iterating arguments in order and processing each completely before moving to the next.
* Trim final string to remove leading/trailing whitespace.

## 💻 Final Solution

```js
export default function classNames(...args) {
  const seen = new Set();
  const result = [];

  function process(value) {
    // Handle falsy values
    if (!value) return;

    // String or number → add directly
    if (typeof value === 'string' || typeof value === 'number') {
      if (!seen.has(value)) {
        seen.add(value);
        result.push(value);
      }
      return;
    }

    // Array → recurse into each element
    if (Array.isArray(value)) {
      for (const item of value) {
        process(item);
      }
      return;
    }
    
    // Object → iterate keys, add truthy ones
    if (typeof value === 'object') {
      for (const key of Object.keys(value)) {
        if (Object.hasOwn(value, key) && value[key]) {
          if (!seen.has(key)) {
            seen.add(key);
          }
          result.push(key);
        }
      }
      return;
    }
  }

  for (const arg of args) {
    process(arg);
  }

  return result.join(' ').trim();
}
```

## 🤔 Why This Works

* `process()` handles all input types uniformly.
* `Set` ensures each class name appears only once.
* Recursive array handling preserves order and flattens nested structures.
* `trim()` removes any leading/trailing whitespace.
* Order is preserved because we iterate arguments in order and process each argument fully before moving to the next.

## 🐞 Bugs I Made

* Using `Array.isArray()` instead of `instanceof Array` — safer across frames.
* `Object.keys(value)` only includes own enumerable properties, matching requirements.
* `Object.hasOwn()` prevents inherited keys from being included.
* `seen` ensures deduplication while preserving first‑occurrence order.
* `trim()` handles edge cases where whitespace might be introduced.

## Production Considerations

- For large lists, the recursive `process` may hit call‑stack limits; an iterative approach could be used.
- The function is pure and has no side effects.
- In React, prefer `clsx` or `classnames` libraries for production.

## ⭐ Revision Notes

### 🔑 Key Facts

* `Set` maintains insertion order (ES2015+), so deduplication preserves first occurrence.
* `Object.hasOwn()` checks own properties only.
* `Array.isArray()` works across iframes.
* `trim()` removes leading/trailing whitespace.
* The algorithm is O(n) where n is total items processed.

### 🧠 Mental Model

Think of it as a **filter‑map‑flatten** pipeline:

```text
args → normalize each arg → flatten arrays → filter truthy keys → deduplicate → join
```

### Common Interview Questions

- How to handle nested arrays deeper than 2 levels? The recursive `process` handles any depth automatically.
- Why use `Object.hasOwn` instead of `in`? `in` includes inherited properties; we only want own enumerable keys.
- What about non‑string keys? `Object.keys` returns strings; they become class names as-is.

### Interview Takeaways

* Master the three input types: string/number, object, array.
* Use `Set` for deduplication while preserving order.
* Recursion is natural for nested array handling.

### Related

- [[Higher Order Mapping]]
- [[Array Traversal]]
- [[Conditional Logic]]

## Related Concepts

- [[Type Checking]]
- [[Falsy Values]]
- [[Array.prototype.reduce]]