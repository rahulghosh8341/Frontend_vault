# Chunk

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min

#javascript #easy #arrays #iteration

---

## Summary

Split an array into subarrays of a fixed size while preserving order.

- Return a new array.
- Last chunk may contain fewer elements.
- Do not mutate the original array.

---

## Companies

> Not specified

---

## Pattern

⭐⭐⭐ [[Fixed Size Grouping]]

⭐⭐ [[Array Traversal]]

---

## Function Signature

```ts
chunk<T>(array: T[], size?: number): T[][]
```

---

## Constraints

- `size >= 1`
- `0 <= array.length <= 100`
- Return `[]` for an empty array.
- Preserve element order.
- Do not mutate the input array.

---

## Prerequisites

- [[Array]]
- [[Iteration]]
- [[Array.push]]

---

## Production Notes

- Lodash provides `_.chunk()`.
- A single-pass iterative solution is the most efficient approach.
- `slice()` can simplify the implementation but creates additional intermediate arrays.

---

## Interview Follow-ups

- What happens if `size` is larger than the array length?
- How would you implement this using `slice()`?
- Can you solve it using `reduce()`?
- What should happen when `size` is zero or negative?
- Can the solution be implemented without an extra temporary array?

---

## Complexity

| Metric | Value |
|---------|------:|
| Time | O(n) |
| Space | O(n) |

---

## My Submission

```js
export default function chunk(array, size = 1) {
  if (!Array.isArray(array) || array.length === 0 || size < 1) return [];

  let chunk = [];
  let result = [];

  for (let i = 0; i < array.length; i++) {
    chunk.push(array[i]);

    if (chunk.length === size || i === array.length - 1) {
      result.push(chunk);
      chunk = [];
    }
  }

  return result;
}
```

---

## Review

✅ Correct

### Possible Improvements

- Rename `chuck` to `chunk` for clarity.
- If the interviewer guarantees valid input, the `Array.isArray()` check can be omitted.
- `slice()` offers a concise alternative implementation, though it creates additional arrays.

---

## 🧠 Thought Process



---

## 💻 Final Solution



---

## 🤔 Why This Works



---

## 🐞 Bugs I Made



---

## ⭐ Revision Notes

