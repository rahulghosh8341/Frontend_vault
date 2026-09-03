---
title: Implement compact(array) so it creates a new array with all falsy values removed. This includes false, null, 0, '', undefined, NaN, and empty slots in sparse arrays. Do not modify the original array.
aliases:
  - Compact
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Falsy Values]]"
  - "[[Array.prototype.filter]]"
solved: true
solvedDate: 2026-08-30
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Create new array with all falsy values removed

## Problem

Remove all falsy values (`false`, `null`, `0`, `''`, `undefined`, `NaN`, empty slots) from array, return new filtered array. Do not modify original.

```javascript
compact([0, 1, false, 2, '', 3, null]); // => [1, 2, 3]
compact(['hello', 123, [], {}]);        // => ['hello', 123, [], {}]
compact([1, , null, 2, , 3]);          // => [1, 2, 3]
```

## Companies

None

## Pattern

- [[Array Traversal]]

## 🤔 Thought Process

`Boolean()` function converts any value to its boolean truthiness. Used as `filter()` callback, it keeps only truthy values, automatically handling all falsy cases including `NaN` and empty slots.

## 💻 Final Solution

```javascript
export default function compact(array) {
  return array.filter(Boolean)
}
```

## 🤔 Why This Works

`Boolean()` acts as identity function for truthy values, converts falsy values to `false`. `filter()` keeps elements where callback returns truthy value. Empty slots in sparse arrays are skipped by `filter()`.

## 🐞 Bugs I Made

None.

## Production Considerations

Lodash `_.compact` uses same approach. `filter(Boolean)` is idiomatic and performant. For manual implementation: `array.filter(item => item)` works identically.

## ⭐ Revision Notes

### Key Facts

- `Boolean()` is truthiness test, not strict equality
- `filter()` skips empty array holes automatically
- Returns new array, original untouched

### Common Interview Questions

- **Why `Boolean` instead of `item => !!item`?** `Boolean` is more idiomatic, same effect
- **Does this remove `0`?** Yes, `0` is falsy
- **What about `{}` and `[]`?** Both truthy, kept

### Interview Takeaways

- Know JS falsy values list cold
- `Boolean` function as callback pattern
- Immutability: `filter` returns new array

### Related

- [[Array Traversal]]
- [[Falsy Values]]
- [[Array.prototype.filter]]
