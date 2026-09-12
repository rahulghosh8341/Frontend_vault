---
title: Implement groupBy(array, iteratee)
aliases:
  - Group By
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Array Iteration]]"
  - "[[Iteratee Functions]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Creates an object whose keys are produced by calling an iteratee on each element, with each key storing an array of elements that produced that result.

## Problem

Implement `groupBy(array, iteratee)` so it creates an object whose keys are the results of calling `iteratee` on each element of `array`. Each key stores an array of the original elements that produced that result. Do not modify the original array.

```js
groupBy([6.1, 4.2, 6.3], Math.floor);
// => { '4': [4.2], '6': [6.1, 6.3] }

groupBy([{ n: 3 }, { n: 5 }, { n: 3 }], (o) => o.n);
// => { '3': [{ n: 3 }, { n: 3 }], '5': [{ n: 5 }] }

groupBy([], (o) => o); 
// => {}

groupBy([{ n: 1 }, { n: 2 }], (o) => o.m); 
// => { undefined: [{ n: 1 }, { n: 2 }] }
```

## Companies

- None

## Pattern

- [[Array Traversal]]

## 🤔 Thought Process

* The output must be an **object** where each key is the value returned by `iteratee`.
* For every item:

  1. Calculate the key.
  2. If that key doesn't exist, create an empty array.
  3. Push the original item into that array.
* Since the required output is an object, a plain object is simpler than using `Map` and converting it back.

## 💻 Final Solution

```js
export default function groupBy(array, iteratee) {
  const result = {};

  for (const item of array) {
    const key = iteratee(item);

    if (!result[key]) {
      result[key] = [];
    }

    result[key].push(item);
  }

  return result;
}
```

## 🤔 Why This Works

```js
if (!Object.hasOwn(result, key)) {
  result[key] = [];
}

result[key].push(item);
```

This handles both:

* **First occurrence** → creates the array.
* **Later occurrences** → pushes into the existing array.

Example:

```js
groupBy([6.1, 4.2, 6.3], Math.floor)
```

becomes:

```js
{
  4: [4.2],
  6: [6.1, 6.3]
}
```

## 🐞 Bugs I Made

* Using `Map` wasn't wrong, but it was unnecessary because the required return type is an **object**.
* `Object.fromEntries(result)` adds an unnecessary conversion step.
* Important distinction:

  * `countBy` → `key → count`
  * `groupBy` → `key → array of original elements`

## Production Considerations

- Modern JavaScript (ES2024+) provides native `Object.groupBy(items, callbackFn)` and `Map.groupBy(items, callbackFn)`.
- In Lodash / production utilities, `iteratee` can also accept property shorthand strings/paths in addition to functions.

## ⭐ Revision Notes

### 🔑 Key Facts

* `groupBy` groups **original elements**, not the iteratee results.
* `Object.hasOwn(obj, key)` checks whether the key actually exists on the object.
* `Map` is useful when you need arbitrary key types, but here the expected result is an object.
* Time: **O(n)**
* Extra space: **O(n)** for the grouped result.

### 🧠 Mental Model

```text
item
 ↓
iteratee(item)
 ↓
key
 ↓
result[key]
 ↓
add item
```

So:

```js
result[key] = [item1, item2, item3]
```

rather than storing the calculated key itself.

### Common Interview Questions

- What is the difference between `countBy` and `groupBy`? → `countBy` returns occurrence count per key (`{ key: count }`), whereas `groupBy` collects original elements into arrays (`{ key: [elem1, elem2] }`).
- What is the native ES2024 alternative? → `Object.groupBy(array, iteratee)` or `Map.groupBy(array, iteratee)`.
- How can you write `groupBy` using `Array.prototype.reduce`? → `array.reduce((acc, item) => { const k = iteratee(item); (acc[k] = acc[k] || []).push(item); return acc; }, {})`.

### Interview Takeaways

**`groupBy` = bucket items by a derived key.**

The standard pattern is:

```js
const result = {};

for (const item of array) {
  const key = iteratee(item);

  if (!Object.hasOwn(result, key)) {
    result[key] = [];
  }

  result[key].push(item);
}
```

For this GFE problem, **your `Map` approach works, but the direct object approach is cleaner and more appropriate**.

### Related

- [[Array Traversal]]
- [[Count By]]
- [[Array.prototype.reduce]]
