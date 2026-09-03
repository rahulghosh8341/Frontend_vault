---
title: Clamp
solved: true
solvedDate: 2026-08-06
type: coding
---

# Clamp

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 5 min

#javascript #easy #math #conditionals #comparison

## Summary

Implement a `clamp(value, lower, upper)` function that restricts a number to an inclusive range.

- Return `lower` if the value is below the range.
- Return `upper` if the value exceeds the range.
- Otherwise return the original value.

---

## Companies

>[ Not specified] 
---

## Pattern

⭐⭐⭐ [[Range Checking]]

---

## Function Signature

```ts
clamp(value: number, lower: number, upper: number): number
```

---

## Constraints

- Inclusive bounds
- Return a number
- Do not modify inputs
- Constant time solution

---

## Concepts

- [[Conditional Statements]]
- [[Comparison Operators]]
- [[Math Utilities]]

---

## Related Topics

- [[Math.min]]
- [[Math.max]]
- [[Input Validation]]

---

## Production Notes

JavaScript already provides a concise implementation:

```js
Math.max(lower, Math.min(value, upper))
```

Custom implementations are common interview questions to test logical reasoning.

---

## Interview Follow-ups

- What if `lower > upper`?
- How would you clamp floating-point values?
- Can this be written without `if` statements?
- Why is the range inclusive?
- Which implementation is more readable?

---

## Similar Problems

- [[Normalize]]
- [[Range Checking]]
- [[Binary Search]]

---

## My Submission

```js
export default function clamp(value, lower, upper) {
  if (value < lower) return lower;
  if (value > upper) return upper;
  return value;
}
```

### Feedback

✅ Correct.
Time: **O(1)**

Space: **O(1)**

Can also be simplified to:

```js
return Math.max(lower, Math.min(value, upper));
```

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


