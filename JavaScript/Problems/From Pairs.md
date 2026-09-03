---
title: Implement a function `fromPairs(pairs)` that transforms a list of key-value pairs into an object
aliases:
  - From Pairs
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[Array Traversal]]"
solved: true
solvedDate: 2026-08-19
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Transforms an array of key-value pairs into an object, overwriting duplicate keys with later values.

## Problem

Implement a function `fromPairs(pairs)` that transforms a list of key-value pairs into an object. Process pairs in order; if a key appears more than once, the later pair overwrites the earlier value.

```js
const pairs = [
  ['a', 1],
  ['b', 2],
  ['c', 3],
];

fromPairs(pairs); // => { a: 1, b: 2, c: 3 }
```

---

## Pattern

- [[Array Traversal]]

---

## 🤔 Thought Process

1. Create an empty object `result` to store the key-value pairs.
2. Traverse every pair in the `pairs` array sequentially.
3. Each pair is a 2-element array: `[key, value]`.
4. Access the key using `pairs[i][0]` and the value using `pairs[i][1]` (or destructuring `[key, value]`).
5. Assign `result[key] = value`. Property assignment in JavaScript automatically overwrites existing keys if duplicate keys appear later.
6. Return the constructed object.

---

## 💻 Final Solution

```js
export default function fromPairs(pairs) {
  let result = {};

  for (let i = 0; i < pairs.length; i++) {
    result[pairs[i][0]] = pairs[i][1];
  }

  return result;
}
```

---

## 🤔 Why This Works

- Direct object property assignment (`obj[key] = value`) sets a new key or overwrites an existing key if it already exists in the object.
- By iterating sequentially from index `0` to `pairs.length - 1`, any later key-value pair naturally overwrites previous values for the same key.
- Time Complexity: **O(n)** time where `n` is the number of pairs.
- Space Complexity: **O(n)** space to construct the output object.

---

## 🐞 Bugs I Made

- **Off-by-one boundary error**: Initially wrote `i < pairs.length - 1` in the `for` loop condition, which caused the loop to skip the last pair.
- **Syntax error in loop header**: The loop header increment was initially written with a comma instead of a semicolon (e.g., `for (let i = 0; i < pairs.length, i++)`).

---

## Production Considerations

In modern JavaScript (ES2019+), native `Object.fromEntries()` accomplishes this exact transformation natively:

```js
export default function fromPairs(pairs) {
  return Object.fromEntries(pairs);
}
```

- Prefer `Object.fromEntries()` in production for clean, idiomatic code.
- Custom iteration is helpful for understanding how object key-value transformation works under the hood or supporting legacy environments.

---

## ⭐ Revision Notes

### Key Facts

- `Object.fromEntries(pairs)` is the native ES2019 inverse of `Object.entries(obj)`.
- Property assignment `obj[key] = value` overwrites previous entries for duplicate keys automatically.
- Non-string keys (e.g., numbers, symbols) used as object property keys are converted to strings in standard JS objects unless using a `Map`.

### Common Interview Questions

- What native ES2019 method replaces `fromPairs`? → `Object.fromEntries()`
- How does key collision work in plain objects vs `Map`? → Plain object keys are converted to strings/symbols and overwritten; `Map` retains key types and insertion order.
- What is the inverse of `Object.fromEntries()`? → `Object.entries(obj)`

### Interview Takeaways

- Watch out for loop boundary conditions (`i < pairs.length` vs `i <= pairs.length` or `i < pairs.length - 1`).
- Syntax check: ensure loop headers use semicolons `;`, not commas `,`.

### Related

- [[Array Traversal]]
